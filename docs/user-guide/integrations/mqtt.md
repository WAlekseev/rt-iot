---
layout: docwithnav
title: MQTT интеграция
description: MQTT интеграция
---

* TOC
{:toc}

Интеграция MQTT позволяет подключаться к внешним брокерам MQTT, подписываться на потоки данных от этих брокеров и конвертировать любой тип полезной нагрузки с ваших устройств в формат сообщений платформы. Его типичное использование - всякий раз, когда ваши устройства уже подключены к внешнему брокеру MQTT или к любой другой платформе IoT или поставщику подключения с серверной частью на основе MQTT.

Пожалуйста, просмотрите схему интеграции, чтобы узнать больше.

 ![image](/images/user-guide/integrations/mqtt-integration.png)

Интеграция MQTT платформы действует как клиент MQTT. Он подписывается на темы и преобразует данные в обновления телеметрии и атрибутов. В случае сообщения нисходящего канала интеграция MQTT преобразует его в формат, подходящий для устройства, и отправляет внешнему брокеру MQTT.
Обратите внимание: брокер MQTT должен быть либо совмещен с экземпляром платформы, либо развернут в облаке и иметь действительное DNS-имя или статический IP-адрес. Платформа, не может подключиться к брокеру MQTT, развернутому в локальной сети.

Это видео представляет собой пошаговое руководство по настройке интеграции MQTT.

<br/>
<div id="video">  
 <div id="video_wrapper">
     <iframe src="https://www.youtube.com/embed/BeN5bsDQbdo" frameborder="0" allowfullscreen></iframe>
 </div>
</div> 

## Конфигурация MQTT интеграции 

Также вы можете следовать этому руководству, которое раскрывает о интеграции MQTT для обеспечения подключения устройств к Платформе и возможности отправки команд RPC на устройства.

### Предпосылки

В этом уроке мы будем использовать:

  - Брокер MQTT, доступный для экземпляра ThingsBoard PE - broker.hivemq.com (порт 1883);
  - MQTT-клиенты mosquitto_pub и mosquitto_sub для отправки и получения сообщений;
  - расширенный [симулятор устройства] (/docs/user-guide/integration/resources/mqtt-client.py) для примера моделирования RPC.

Предположим, у нас есть датчик, который отправляет текущие показания температуры.
Наше сенсорное устройство ** SN-001 ** публикует показания своей температуры в **tb/mqtt-integration-tutorial/sensors/SN-001/temperature**, и оно подписано на ** **tb/mqtt-integration-tutorial/sensors/SN-001/rx** для приема вызовов RPC.

Отправим сообщение с показаниями температуры в простом формате.: **`{"value":25.1}`**
 
### Uplink and Downlink Converters

Перед настройкой интеграции MQTT вам необходимо создать конвертеры восходящего и нисходящего каналов.

**Uplink Converter** - это скрипт для анализа и преобразования данных, полученных с помощью интеграции MQTT.
**Downlink Converter** анализирует и преобразует данные, отправленные из платформы в формат, который используется существующими устройствами.

#### Uplink Конвертер

Чтобы создать конвертер восходящего канала, перейдите в раздел ** Конвертеры данных** и нажмите **Добавить новый конвертер данных -> Создать новый конвертер**.
Назовите его **"MQTT Uplink Converter"** и выберите тип **Uplink**. На данный момент используйте режим отладки.

** ПРИМЕЧАНИЕ **. Хотя режим отладки очень полезен для разработки и устранения неполадок, оставление его включенным в производственном режиме может значительно увеличить дисковое пространство, используемое базой данных, поскольку все данные отладки хранятся там. После завершения отладки настоятельно рекомендуется выключить режим отладки.

Теперь скопируйте и вставьте следующий скрипт в раздел функции декодера:

```javascript
/** Decoder **/

// decode payload to string
var payloadStr = decodeToString(payload);
var data = JSON.parse(payloadStr);
var topicPattern = 'tb/mqtt-integration-tutorial/sensors/(.+)/temperature';

var deviceName =  metadata.topic.match(topicPattern)[1];
// decode payload to JSON
var deviceType = 'sensor';

// Result object with device attributes/telemetry data
var result = {
   deviceName: deviceName,
   deviceType: deviceType,
   attributes: {
       integrationName: metadata['integrationName'],
   },
   telemetry: {
       temperature: data.value,
   }
};

/** Helper functions **/

function decodeToString(payload) {
   return String.fromCharCode.apply(String, payload);
}

function decodeToJson(payload) {
   // convert payload to string.
   var str = decodeToString(payload);

   // parse string to JSON
   var data = JSON.parse(str);
   return data;
}

return result;

``` 

Назначение функции декодера - анализировать входящие данные и метаданные в формате, который может использовать платформа.
**deviceName** и **deviceType** являются обязательными, а **атрибуты** и **телеметрия** необязательны.
**Атрибуты** и **телеметрия** представляют собой плоские объекты "ключ-значение". Вложенные объекты не поддерживаются.

#### Downlink Конвертер

Конвертер нисходящего канала преобразует исходящее сообщение RPC, а затем интеграция отправляет его внешнему брокеру MQTT.

**ПРИМЕЧАНИЕ** Даже если вы не будете отправлять RPC по нисходящему каналу, вам все равно необходимо создать фиктивный конвертер нисходящего канала.

Создайте еще один конвертер с именем **"MQTT Downlink Converter"** и введите **Downlink**. Оставьте сценарий по умолчанию и нажмите **Добавить**.     

### Настройки интеграции MQTT

Перейдите в раздел **Интеграции** и нажмите кнопку **Добавить новую интеграцию**. Назовите ее **MQTT Integration**, выберите тип **MQTT**, включите режим отладки и из раскрывающихся меню добавьте недавно созданные преобразователи восходящего и нисходящего каналов.

Укажите хост: **broker.hivemq.com**. Порт: **1883**. Лучше убрать галочку с параметра **Чистая сессия**. Многие брокеры не поддерживают прикрепленные сеансы, поэтому автоматически закроют соединение, если вы попытаетесь подключиться с включенной этой опцией.

Добавьте фильтр тем **tb/mqtt-integration-tutorial/sizes/+/temperature**. Вы также можете выбрать уровень MQTT QoS. По умолчанию мы используем MQTT QoS level 0.

Давайте оставим шаблон темы Downlink по умолчанию, что означает, что Integration возьмет metadata.topic и будет использовать его в качестве темы downlink.

Нажмите **Добавить**, чтобы сохранить интеграцию.

### Отправить Uplink сообщение

Теперь давайте смоделируем устройство, отправляющее показания температуры в интеграцию:

```shell
mosquitto_pub -h broker.hivemq.com -p 1883 -t "tb/mqtt-integration-tutorial/sensors/SN-001/temperature" -m '{"value":25.1}'
```

Как только вы перейдете к **Группы устройств -> Все**, вы должны найти устройство **SN-001**, предоставленное интеграцией.
Щелкните устройство, перейдите на вкладку **Последняя телеметрия**, чтобы увидеть там ключ «температура» и его значение (25.1).

Вернитесь к своей интеграции и перейдите на вкладку «События». Там вы увидите сообщение, используемое интеграцией.
На вкладке «События» конвертера MQTT Uplink будут столбцы **In**, **Out** и **Metadata**.
**In** и **Metadata** являются входными данными для преобразователя данных, а **Out** - результатом.

Резюме: Конвертер данных восходящей линии связи определяет инициализацию устройства и интерпретацию входных данных.
В нашем примере мы захватываем имя устройства из темы (SN-001), устанавливаем тип устройства по умолчанию (датчик) и заполняем его значением телеметрии.
В более сложных случаях вы можете написать сценарий, который будет брать эти данные из любой части данных или метаданных.

### Отправка одностороннего RPC на устройство

В этом разделе описывается, как отправить односторонний запрос RPC на устройство с помощью виджетов управления.

Для вашего удобства вы можете просмотреть это видео, чтобы настроить RPC на устройство и получить смоделированный ответ через интеграцию MQTT.

<br/>
<div id="video">  
 <div id="video_wrapper">
     <iframe src="https://www.youtube.com/embed/A_jCXQj_LXs" frameborder="0" allowfullscreen></iframe>
 </div>
</div> 

### Simulating of Two-Way RPC 

Теперь попробуйте смоделировать отправку RPC-запроса на устройство с получением ответа.

Сначала вы должны изменить конвертеры для отправки сообщений нисходящего канала на **tb/mqtt-integration-tutorial/sensors/+/rx/twoway** и получить ответы устройства на 
**tb/mqtt-integration-tutorial/sensors/+/rx/response**

Измените код конвертера нисходящего канала для отправки сообщений **tb/mqtt-integration-tutorial/sensors/+/rx/twoway**:

```js
/** Encoder **/

var value = parseInt(msg.params.replace(/"/g,""));
var data = {value: value}
// Result object with encoded downlink payload
var result = {

    // downlink data content type: JSON, TEXT or BINARY (base64 format)
    contentType: "JSON",

    // downlink data
    data: JSON.stringify(data),

    // Optional metadata object presented in key/value format
    metadata: {
        topic: 'tb/mqtt-integration-tutorial/sensors/'+metadata['deviceName']+'/rx/twoway'
    }

};

return result;
```

Затем подготовьте конвертер восходящего канала для получения ответных сообщений. Перейдите в **«Конвертер MQTT Uplink Converter»** и вставьте следующий код:
```js
/** Decoder **/

// decode payload to string
var payloadStr = decodeToString(payload);
var data = JSON.parse(payloadStr);
var topicPattern = 'tb/mqtt-integration-tutorial/sensors/(.+?)/.*';

var deviceName =  metadata.topic.match(topicPattern)[1];
// decode payload to JSON
var deviceType = 'sensor';

// Result object with device attributes/telemetry data
var telemetry;
if (metadata.topic.endsWith('/temperature')) {
    // Transform the incoming data as before
    telemetry = getTemperatureTelemetry(data);
} else if (metadata.topic.endsWith('/rx/response')) {
    // Get the input value as is
    telemetry = data;
}

var result = {
   deviceName: deviceName,
   deviceType: deviceType,
   attributes: {
       integrationName: metadata['integrationName'],
   },
   telemetry: telemetry
};

/** Helper functions **/

function getTemperatureTelemetry(data) {
    return {temperature: data.value}
}

function decodeToString(payload) {
   return String.fromCharCode.apply(String, payload);
}

function decodeToJson(payload) {
   // covert payload to string.
   var str = decodeToString(payload);

   // parse string to JSON
   var data = JSON.parse(str);
   return data;
}

return result;
```
Приведенный выше сценарий немного отличается от того, что у нас было изначально. Он различает запросы пост-телеметрии и ответы на вызовы RPC, тем самым публикуя различные выходные данные в движок правил.

Теперь запустите эмулятор устройства. Обратите внимание, что mosquitto_pub и mosquitto_sub недостаточно, поэтому запустите [продвинутый симулятор](/docs/user-guide/integrations/resources/mqtt-client.py):

```js
python mqtt-client.py
```

Попробуйте повернуть колесико на приборной панели. В окне терминала у вас должен быть вывод, похожий на:

```log
Incoming message
Topic: tb/mqtt-integration-tutorial/sensors/SN-001/rx/twoway
Message: {"value":36}
This is a Two-way RPC call. Going to reply now!
Sending a response message: {"rpcReceived":"OK"}
```

Перейдите в **Группы устройств** и найдите **rpcReceived** значение телеметрии **ОК** на вкладке телеметрии вашего устройства SN-001.
 
