---
layout: docwithnav
title: OPC-UA интеграция
description: OPC-UA интеграция
---

* TOC
{:toc}


## Обзор

Интеграция OPC UA позволяет передавать данные с сервера OPC UA на платформу и преобразовывать полезную нагрузку устройства в предусмотренный платформой формат

![image](/images/user-guide/integrations/opc-ua-integration.png)

### Видео-урок

Вы можете посмотреть данное видео, в котором содержится пошаговая инструкция по запуску OPC-UA интеграции

<br/>
<div id="video">  
    <div id="video_wrapper">
        <iframe src="https://www.youtube.com/embed/KK0gXGXFQ0E" frameborder="0" allowfullscreen></iframe>
    </div>
</div> 

### Настройка интеграции OPC-UA

В данном руководстве вы найдете информацию для настройки интеграции между платформой и OPC-UA, чтобы получать данные кондиционеров с [OPC UA C++ Demo Server](https://www.unified-automation.com/downloads/opc-ua-servers/file/download/details/opc-ua-c-demo-server-v161-windows.html) и давать возможность пользователям включать и выключать кондиционер с помощью функции интеграции с устройством.


#### Условия

- Скачайте и установите [OPC UA C++ Demo Server](https://www.unified-automation.com/downloads/opc-ua-servers/file/download/details/opc-ua-c-demo-server-v161-windows.html).
- После установки запустите **диалоговое окно от имени Администратора**.
- Подтвердите корректность **URL-адреса конечной точки** и запомните значения хоста и порта конечной точки. Эти значения понадобятся во время запуска интеграции OPC-UA.

![image](/images/user-guide/integrations/opc-ua/opc-ua-server-config.png)

- Запустите **UaCPPServer**.  Откроется диалоговое окно консоли с URL-адресами конечных точек сервера.

#### Настройка платформы

##### Конвертер данных от устройства

Сначала понадобится создать конвертер данных от устройства, который будет использоваться для получения сообщения с сервера OPC UA. Конвертер будет преобразовывать входящую полезную нагрузку сообщений в требуемый формат сообщения.
В сообщении должны содержаться поля **deviceName** и **deviceType**. Они используются для отправки данных на конкретные устройства.  Если устройство не будет найдено, то создастся новое.
Как будет выглядеть полезная нагрузка из интеграции OPC UA:



Payload:
{% highlight json %}
{
    "temperature": "72.15819999999641"
}
{% endhighlight %}

Metadata:
{% highlight json %}
{
    "opcUaNode_namespaceIndex": "3",
    "opcUaNode_name": "AirConditioner_1",
    "integrationName": "OPC-UA Airconditioners",
    "opcUaNode_identifier": "AirConditioner_1",
    "opcUaNode_fqn": "Objects.BuildingAutomation.AirConditioner_1"
}
{% endhighlight %}

Мы возьмем значение метаданных **opcUaNode_name**, сопоставим его с **deviceName** и установим **airconditioner** в качестве **deviceType**.

Но вы можете использовать другое сопоставление в зависимости от ваших целей.
Также мы получим значения полей **temperature**, **humidity** и **powerConsumption** и будем использовать их в качестве телеметрии устройства.

Далее переходим в **Конвертеры данных** и создаем новый конвертер **от устройства** с помощью следующей функции:

{% highlight javascript %}

var data = decodeToJson(payload);
var deviceName = metadata['opcUaNode_name'];
var deviceType = 'airconditioner';

var result = {
   deviceName: deviceName,
   deviceType: deviceType,
   telemetry: {
   },
   attributes: {
   }
};

if (data.temperature) {
    result.telemetry.temperature = Number(Number(data.temperature).toFixed(2));
}

if (data.humidity) {
    result.telemetry.humidity = Number(Number(data.humidity).toFixed(2));
}

if (data.powerConsumption) {
    result.telemetry.powerConsumption = Number(Number(data.powerConsumption).toFixed(2));
}

if (data.state !== undefined) {
    result.attributes.state = data.state === '1' ? true : false;
}

function decodeToString(payload) {
   return String.fromCharCode.apply(String, payload);
}

function decodeToJson(payload) {
   var str = decodeToString(payload);
   var data = JSON.parse(str);
   return data;
}

return result;
{% endhighlight %}

![image](/images/user-guide/integrations/opc-ua/uplink-converter.png)

##### Конвертер данных к устройству

Для отправки сообщений на устройства с OPC UA узла, надо настроить конвертер данных к устройству.

Вызов данных из данного конвертера должны иметь следующую структуру:

{% highlight json %}
[{
    "contentType": "JSON",
    "data": "{\"writeValues\":[],\"callMethods\":[{\"objectId\":\"ns=3;s=AirConditioner_1\",\"methodId\":\"ns=3;s=AirConditioner_1.Stop\",\"args\":[]}]}",
    "metadata": {}
}]
{% endhighlight %}

- **contentType** - определяет кодировку данных {TEXT \| JSON \| BINARY}. Для интеграции OPC UA по умолчанию используется формат JSON.
- **data** - актуальные данные, которые будут обработаны интеграцией OPC UA и отправлены в целевые узлы OPC UA:
    - **writeValues** - массив, в котором передаются методы записи значений:
        - **nodeId** - целевой узел в [OPC UA NodeId format](http://documentation.unified-automation.com/uasdkhp/1.0.0/html/_l2_ua_node_ids.html#UaNodeIdsConcept) (`ns=<namespaceIndex>;<identifiertype>=<identifier>`)
        - **value** - значение для записи
    - **callMethods** - массив методов для вызовов:
        - **objectId** - целевой объект в [OPC UA NodeId format](http://documentation.unified-automation.com/uasdkhp/1.0.0/html/_l2_ua_node_ids.html#UaNodeIdsConcept) 
        - **methodId** - целевой метод в [OPC UA NodeId format](http://documentation.unified-automation.com/uasdkhp/1.0.0/html/_l2_ua_node_ids.html#UaNodeIdsConcept) 
        - **args** - массив входных значений метода
- **metadata** - не используется в интеграции OPC UA, поле может быть пустым.

Перейдите в **Конвертеры данных** и создайте новый конвертер данных **к устройству** с помощью следующей функции:



{% highlight javascript %}
var data = {
    writeValues: [],
    callMethods: []
};

if (msgType === 'RPC_CALL_FROM_SERVER_TO_DEVICE') {
    if (msg.method === 'setState') {
        var targetMethod = msg.params === 'true' ? 'Start' : 'Stop';
        var callMethod = {
              objectId: 'ns=3;s=' + metadata['deviceName'],
              methodId: 'ns=3;s=' +metadata['deviceName']+'.'+targetMethod,
              args: []
        };
        data.callMethods.push(callMethod);
    }
}

var result = {
    contentType: "JSON",
    data: JSON.stringify(data),
    metadata: {}
};

return result;
{% endhighlight %}

Этот конвертер будет обрабатывать RPC-команды к устройству с помощью метода **setState** и логического значения **params** для вызова метода "Старт" или "Стоп" для кондиционера.

Узел назначения определяется с помощью поля **deviceName**, передающегося в метаданных входящего сообщения.

![image](/images/user-guide/integrations/opc-ua/downlink-converter.png)

##### OPC-UA Integration

Интеграция OPC-UA

Далее создадим Интеграцию OPC UA с сервером OPC UA.
Откройте меню **Интеграции** и добавьте новую интеграцию с типом **OPC-UA**

- Название: OPC-UA кондиционеры
- Тип: OPC-UA
- Конвертер данных (Uplink): Кондиционер Uplink
- Конвертер данных (Downlink): Кондиционер Downlink
- Название приложения: \<empty\> (название клиентского приложения)
- Uri приложения: \<empty\> (uri клиентского приложения)
- Хост: **Хост конечной точки** (подробнее [Предусловия](/docs/user-guide/integrations/opc-ua/#prerequisites))
- Порт: **Порт конечной точки** (подробнее [Предусловия](/docs/user-guide/integrations/opc-ua/#prerequisites))
- Период сканирования в секундах: 10 (как часто повторно сканировать узлы OPC UA)
- Время ожидания в миллисекундах: 5000 (время ожидания в миллисекундах  до выставления статуса "сбой" для запроса на сервер OPC UA)
- Безопасность: None (может быть *Basic128Rsa15 / Basic256 / Basic256Sha256 / None*)
- Идентичность: Anonymous (может быть *Anonymous / Username*)
- Сопоставление:
     - Шаблон узла устройства: `Objects\.BuildingAutomation\.AirConditioner_\d+$` (регулярне выражение, которое используется для сопоставления отсканированных OPC UA Node FQNs/IDs с именем устройства.)
     - MappingType: полное имя (может быть *Fully Qualified Name* / *ID*)
     - Теги подписки (список тегов узла (**Путь**) для того, чтобы подписаться на ключи сопоставления  (**Ключ**), используется в выходных сообщениях):
        - state - Состояние
        - temperature - Температура
        - humidity - Влажность
        - powerConsumption - Энергопотребление

![image](/images/user-guide/integrations/opc-ua/opc-ua-integration-mapping.png)

##### Цепочка правил для кондиционера

Чтобы продемонстрировать возможности интеграции OPC-UA и движка правил, мы создадим раздельные цепочки правил для обработки сообщений от устройства и к устройству, которые связаны с интеграцией OPC-UA.

Создадим цепочку правил **Кондиционеры**.

 - Скачайте файл [**airconditioners.json**](/docs/user-guide/resources/airconditioners.json).
 - Чтобы импортировать этот файл, кликните кнопку `+` справа на странице **Цепочка правил** и выберете опцию **Импортировать цепочку правил**.
 - Дважды нажмите на узел интеграции к устройству **Кондиционеры** и выберете **OPC-UA Кондиционеры** в поле **Интеграции**.
 - Подтвердите и сохраните изменения.


![image](/images/user-guide/integrations/opc-ua/airconditioners-rule-chain.png)
![image](/images/user-guide/integrations/opc-ua/airconditioners-integration-downlink.png)

 - Откройте редактирование **Root Rule Chain**.
 - Добавьте узел **rule chain**.
 - Выберете цепочку правил **Кондиционеры** и соедините с узлом переключения типа сообщения с помощью следующих лэйблов:
**Attributes Updated** / **Post telemetry** / **RPC Request to Device**.

![image](/images/user-guide/integrations/opc-ua/root-rule-chain.png)

##### Дашборд для кондиционеров

Для визуализации данных, поступающих с кондиционеров и тестирования RPC-команд мы создадим дашборд **Кондиционеры**.

 - Скачайте файл [**airconditioners_dashboard.json**](/docs/user-guide/resources/airconditioners_dashboard.json).
 - Чтобы импортировать файл JSON, на странице **Дашборды** выберите опцию **Импортировать дашборд**.

#### Валидация

Чтобы проверить нашу интеграцию,

 - Перейдите на страницу **Группы устройств**. Там вы увидите группу **Кондиционеры**.
 - Когда вы откроете эту группу, вы увидите в списке 10 устройств с типом кондиционер.

![image](/images/user-guide/integrations/opc-ua/airconditioners-group.png)

 - Откройте Детали группы объектов, вкладку **Последняя телеметрия**.
 - Вы увидите данные, обновляющиеся в режиме реального времени.

![image](/images/user-guide/integrations/opc-ua/airconditioner-latest-telemetry.png)

 - Перейдите в раздел **Дашборды** и откройте дашборд **Кондиционеры**.
 - Вы увидите телеметрические данные до последней минуты со всех кондиционеров.

![image](/images/user-guide/integrations/opc-ua/airconditioners-dashboard.png)

 - Откройте детали Кондиционеров, кликнув на кнопку детали в виджете сущностей.

![image](/images/user-guide/integrations/opc-ua/airconditioners-dashboard-click-details.png)

 - Статус кондиционера будет высвечиваться зеленым цветом.
 - Попробуйте выключить кондиционер, нажав **Вкл/Выкл круговой переключатель**.
 - Статус перейдет в серый цвет, температура начнет подниматься, показатели влажности начнут расти, а расход энергии прекратится.

![image](/images/user-guide/integrations/opc-ua/airconditioners-dashboard-details.png)

## See also

- [Integration Overview](/docs/user-guide/integrations/) 
- [Uplink Converters](/docs/user-guide/integrations/#uplink-data-converter) 
- [DownLink Converters](/docs/user-guide/integrations/#downlink-data-converter) 
- [Rule Engine](/docs/user-guide/rule-engine-2-0/re-getting-started/)
