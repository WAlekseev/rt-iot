---
layout: docwithnav
title: Узлы фильтрации
description: Узлы фильтрации

---
Данный функционал используется для фильтрации и маршрутизации сообщений.

* TOC
{:toc}

##### Проверка связей узла фильтрации

<table  style="width:12%">
   <thead>
     <tr>
     <td style="text-align: center"><strong><em>С версии IoT Ростелеком 2.3</em></strong></td>
     </tr>
   </thead>
</table> 

![image](/images/user-guide/rule-engine-2-0/nodes/filter-check-relation.png)

Проверяет связь между выбранной сущности и отправителем сообщения по типу и направлению.

![image](/images/user-guide/rule-engine-2-0/nodes/filter-check-relation-config.png)

Если связь существует, сообщение отправляется по направлению **True**, в противном случае – по направлению **False**.

**Примечание**: Узел правил может проверять наличие связи с конкретным объектом или с любым объектом по направлению и типу связи. Для этого надо отключить флажок в настройках узла правил:

![image](/images/user-guide/rule-engine-2-0/nodes/check-relation-checkbox.png)

Если флажок не выставлен и существует какая-либо связь, сообщение отправится по направлению **True**, в противном случае – по направлению **False**.

##### Проверка существующих полей в узле

![image](/images/user-guide/rule-engine-2-0/nodes/check-existance-fields.png)

Узел правил проверяет наличие выбранных ключей в данных и метаданных входящих сообщений.

![image](/images/user-guide/rule-engine-2-0/nodes/check-existance-fields-config.png)

<![endif]-->

Если установлен флажок **Проверить, что все выбранные ключи присутствуют** и что они существуют в данных и метаданных сообщения, Сообщение отправится по направлению **True**, в противном случае – по направлению **False**.

Если этот флажок не установлен и хотя бы один из ключей в данных или метаданных сообщения существует - сообщение отправится по направлению **True**, в противном случае – по направлению **False**.

##### Узел переключения типов сообщений



![image](/images/user-guide/rule-engine-2-0/nodes/filter-message-type.png)

В настройках узла администратор определяет набор разрешенных типов входящих сообщений.
В системе заданы [предопределенные типы сообщений](/docs/user-guide/rule-engine-2-0/overview/#predefined-message-types), такие как **Post-атрибуты**, **Post-телеметрия**, **RPC-запрос** и т. д. Также администратор может определить любые типы кастомных сообщений в настройках узла.

![image](/images/user-guide/rule-engine-2-0/nodes/filter-message-type-config.png)

Если тип входящего сообщения ожидаемый - сообщение отправится по направлению **True**, в противном случае – по направлению **False**.

##### Message Type Switch Node

![image](/images/user-guide/rule-engine-2-0/nodes/filter-message-type-switch.png)

Направляет входящие сообщения по их типу. Если входящее сообщение имеет известный [тип](/docs/user-guide/rule-engine-2-0/overview/#predefined-message-types), то оно отправляется в соответствующую цепочку, в противном случае – в другую.

Если вы используете кастомные типы сообщений, вы можете направить их через **другую** цепочку, входящую в **узел переключения типов сообщений**, в **узел переключения** или **узел фильтрации по типу сообщений**, который настроен по требуемой логике маршрутизации.

##### Узел фильтрации по типу отправителя


![image](/images/user-guide/rule-engine-2-0/nodes/filter-originator-type.png)

В настройках узла администратор определяет набор разрешенных типов [сущностей](/docs/user-guide/entities-and-relations/) отправителя для входящего сообщения.

![image](/images/user-guide/rule-engine-2-0/nodes/filter-originator-type-config.png)

Если тип отправителя входит в ожидаемые - сообщение отправляется по направлению **True**, в противном случае – по направлению **False**.

##### Узлы переключения типов отправителей

![image](/images/user-guide/rule-engine-2-0/nodes/filter-originator-type-switch.png)

Направляет входящие сообщения по типу сущности [Отправитель](/docs/user-guide/entities-and-relations/).

##### Узел фильтрации скриптов

![image](/images/user-guide/rule-engine-2-0/nodes/filter-script.png)

Анализирует входящее сообщение по настроенным JavaScript-условиям.

Функция JavaScript получает 3 входных параметра:

- <code>msg</code> - полезная нагрузка сообщения.
- <code>metadata</code> - метаданные сообщения.
- <code>msgType</code> - тип сообщения.

Скрипт должен вернуть булевое значение. Если возвращается **True** - сообщение отправится по направлению **True**, в противном случае – по направлению **False**.

![image](/images/user-guide/rule-engine-2-0/nodes/filter-script-config.png)
 
Полезная нагрузка сообщения может быть доступна через переменную <code>msg</code>. Например <code>msg.temperature < 10;</code><br/> 
Метаданные могут быть доступны через переменную <code>metadata</code>. Например, <code>metadata.customerName === 'John';</code><br/> 
Тип сообщения может быть доступен через переменную <code>msgType</code>. Например, <code>msgType === 'POST_TELEMETRY_REQUEST'</code><br/> 

Полный пример скрипта:

{% highlight javascript %}
if(msgType === 'POST_TELEMETRY_REQUEST') {
    if(metadata.deviceType === 'vehicle') {
        return msg.humidity > 50;
    } else if(metadata.deviceType === 'controller') {
        return msg.temperature > 20 && msg.humidity > 60;
    }
}

return false;
{% endhighlight %}

Условие JavaScript можно проверить с помощью функции [Test JavaScript](/docs/user-guide/rule-engine-2-0/overview/#test-javascript-functions).

Вы можете увидеть примеры, где используется этот узел, в следующих туториалах:

- [Создание и удаление сигналов тревоги](/docs/user-guide/rule-engine-2-0/tutorials/create-clear-alarms/)
- [Ответы на вызовы RPC](/docs/user-guide/rule-engine-2-0/tutorials/rpc-reply-tutorial/#add-filter-script-node)

##### Узел переключения

![image](/images/user-guide/rule-engine-2-0/nodes/filter-switch.png)

Маршрутизирует входящие сообщения по одному или нескольким направлениям.
Узел выполняет настроенную JavaScript-функцию, которая получает 3 входных параметра:

- <code>msg</code> - полезная нагрузка сообщения.
- <code>metadata</code> - метадата сообщения.
- <code>msgType</code> - тип сообщения
 
Скрипт должен **_возвращать массив, состоящий из имен отношений_**, в которые должно будет направлено сообщение. Если возвращаемый массив пустой – сообщение не отправится ни в один узел и будет сброшено.

![image](/images/user-guide/rule-engine-2-0/nodes/filter-switch-config.png)

Полезная нагрузка сообщения может быть доступна через переменную <code>msg</code>. Например <code>msg.temperature < 10;</code><br/> 
Метаданные могут быть доступны через переменную <code>metadata</code>. Например, <code>metadata.customerName === 'John';</code><br/> 
Тип сообщения может быть доступен через переменную <code>msgType</code>. Например, <code>msgType === 'POST_TELEMETRY_REQUEST'</code><br/> 

Полный пример скрипта:

{% highlight javascript %}
if (msgType === 'POST_TELEMETRY_REQUEST') {
    if (msg.temperature < 18) {
        return ['Low Temperature Telemetry'];
    } else {
        return ['Normal Temperature Telemetry'];
    }
} else if (msgType === 'POST_ATTRIBUTES_REQUEST') {
    if (msg.currentState === 'IDLE') {
        return ['Idle State', 'Update State Attribute'];
    } else if (msg.currentState === 'RUNNING') {
        return ['Running State', 'Update State Attribute'];
    } else {
        return ['Unknown State'];
    }
}
return [];
{% endhighlight %}

Функцию переключения JavaScript можно проверить с помощью [Test JavaScript](/docs/user-guide/rule-engine-2-0/overview/#test-javascript-functions).

Для указания имени кастомного отношения следует выбрать тип **Кастомный**. Это позволит ввести пользовательское имя отношения. Имена таких отношений можно заполнять в любом регистре.

![image](/images/user-guide/rule-engine-2-0/nodes/filter-switch-custom-relation.png)

##### Узел фильтрации геозоны по GPS

![image](/images/user-guide/rule-engine-2-0/nodes/filter-gps-geofencing.png)

Фильтрует входящие сообщения по параметрам, которые генерируются на основе показателей GPS. Извлекает широту и долготу из данных или метаданных и проверяет, находятся ли они внутри настроенного периметра (гео-ограждение).

![image](/images/user-guide/rule-engine-2-0/nodes/filter-gps-geofencing-default-config.png)

Узел правил по умолчанию извлекает информацию о периметре из метаданных сообщения. Если флажок **Извлекать информацию о периметре из метаданных сообщения** снят, необходимо сделать дополнительные настройки.

<br>

###### **Получение информации о периметре из метаданных сообщения**

Существует два варианта определения площади в зависимости от типа периметра:

- Полигон 

Метаданные входящего сообщения должны содержать ключ с именем **периметр** и иметь следующую структуру данных:
     
{% highlight java %}[[lat1,lon1],[lat2,lon2], ... ,[latN,lonN]]{% endhighlight %}
 
- Круг
                 
{% highlight java %}"centerLatitude": "value1", "centerLongitude": "value2", "range": "value3"

Все значения для этих ключей имеют double-precision floating-point тип данных.

Ключ "rangeUnit" требует специального значения из списка: METER, KILOMETER, FOOT, MILE, NAUTICAL_MILE (обязательно большими буквами).
{% endhighlight %}

###### Получить информацию о периметре из конфигурации узла
 
Существует два варианта определения площади в зависимости от типа периметра:
 
- Полигон
             
![image](/images/user-guide/rule-engine-2-0/nodes/filter-gps-geofencing-polygon-config.png)           

- Круг
                  
![image](/images/user-guide/rule-engine-2-0/nodes/filter-gps-geofencing-circle-config.png)          
    
Если широта и долгота настроены внутри периметра сообщения, то сообщение будет отправлено по направлению **True**, в противном – по направлению **False**.
      
Направление **Failure** будет использоваться, когда:

- во входящем сообщении в данных или метаданных не передается настроенный ключ «Широта» или «Долгота».
- отсутствует значение «Периметра»;
 
