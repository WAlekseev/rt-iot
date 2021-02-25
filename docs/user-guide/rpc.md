---
layout: docwithnav

title: Возможности RPC
description: Возможности RPC

---

* TOC
{:toc}

С помощью платформы вы можете отправлять RPC с серверного приложения на устройства и наоборот. Это функция позволяет отправлять команды на устройства и получать информацию о результате их выполнения. Также вы можете отправлять запрос с устройства, производить необходимые вычисления или выполнять ину обработку на стороне сервера, а затем отправлять ответ обратно на устройство. 
В данном руководстве рассмотрены следующие темы:

 - Типы RPC-вызовов
 - Основные варианты использования RPC
 - Клиентский и серверный RPC API
 - RPC-виджеты

## Типы RPC вызовов

RPC платформы делятся на два типа: RPC с устройств (далее – клиентский RPC) и RPC с сервера (далее – серверный RPC)
 
Серверный RPC может работать в формате one-way и в формате two-way.

 - One-way серверный RPC-запрос отправляется только в одном направлении: на устройство, без дальнейшего подтверждения о получении. И, соответственно, устройство не отправляет ответ на данный запрос. RPC-вызов может не выполниться только при отсутствии связи с устройством в течение некоторого времени (интервал времени может быть задан).

   
   {:refdef: style="text-align: center;"}
   ![image](/images/user-guide/one-way-rpc.png)
   {: refdef}
   
 - Two-way RPC-запрос отправляется на устройство, которое затем отправляет ответ в течение определенного времени. Серверный запрос блокируется до тех пор, пока не получит ответ от устройства.

   {:refdef: style="text-align: center;"}
   ![image](/images/user-guide/two-way-rpc.png)
   {: refdef}


## API RPC устройств

Платформа имеет API для передачи и получения RPC-команд из приложений, выполняющихся на устройстве. Этот API специфичен для каждого сетевого протокола. Вы можете ознакомиться с API и примерами использования на следующих страницах:

 - [справка о MQTT RPC API](/docs/reference/mqtt-api/#rpc-api)
 - [справка о CoAP RPC API](/docs/reference/coap-api/#rpc-api)
 - [справка о HTTP RPC API](/docs/reference/http-api/#rpc-api) 

## Server-side RPC API

Платформа имеет **System RPC Service**, с помощью которой можно отправлять RPC-запросы из серверных приложений на устройства. Для этого нужно выполнить HTTP POST-запрос вида:

```shell
http(s)://host:port/api/plugins/rpc/{callType}/{deviceId}
```

где 

 - **callType** может быть **oneway** или **twoway**
 - **deviceId** [id устройства](/docs/user-guide/ui/devices/#get-device-id)

В теле запроса должен передаваться JSON-объект с двумя элементами: 
 
 - **method** - название метода, json строка
 - **params** - параметр метода, json объект

Например:

{% capture tabspec %}mqtt-rpc-from-client
A,set-gpio-request.sh,shell,resources/set-gpio-request.sh,/docs/user-guide/resources/set-gpio-request.sh
B,set-gpio-request.json,json,resources/set-gpio-request.json,/docs/user-guide/resources/set-gpio-request.json{% endcapture %}  
{% include tabs.html %}

Обратим внимание на то, что для передачи данного запроса, вам нужно подменить **$JWT_TOKEN** на валидный JWT токен. Этот токен должен принадлежать:

 - пользователю с ролью **TENANT_ADMIN**
 - пользователю с ролью **CUSTOMER_USER** которому принадлежит устройство с **$DEVICE_ID**

## RPC узла правил

Можно интегрировать RPC-события в процесс обработки данных. Существует два узла правил для работы с RPC-запросами.


-  [RPC ответ](/docs/user-guide/rule-engine-2-0/action-nodes/#rpc-call-reply-node) 
-  [RPC запрос](/docs/user-guide/rule-engine-2-0/action-nodes/#rpc-call-request-node) 

## RPC widgets

Обратитесь к [библиотеке виджетов](/docs/user-guide/ui/widget-library/#gpio-widgets) for more details.
