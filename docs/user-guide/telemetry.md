---
layout: docwithnav
title: Работа с данными телеметрии
description: Работа с данными телеметрии

---

* TOC
{:toc}

Платформа имеет широкий функционал для работы с телеметрическими данными:

 - **собирать** данные с устройств с помощью MQTT, CoAP или HTTP протоколов.
 - **сохранять** временные ряды данных в Cassandra (эффективная, масштабируемая и отказоустойчивая NoSQL база данных).
 - **запрашивать** временные ряды данных за произвольный промежуток времени, или свежие значения.
 - **подписываться** на обновления данных с помощью websockets (для визуализации данных или аналитики в режиме реального времени) (for visualization or real-time analytics).
 - **визуализировать** временные ряды данных с помощью кастомизируемых виджетов и дашбордов.
 - **фильтровать и анализировать** данные с помощью движка правил (/docs/user-guide/rule-engine/).
 - **создавать сигналы тревоги** на основе собранных данных.
 - **передавать** данные во внутреннее хранилище с помощью узлов правил (например, Kafka или RabbitMQ Rule Nodes).

В данном руководстве содержится обзор вышеперечисленных функций, а также полезные ссылки для получения более подробной информации.  

![image](/images/user-guide/telemetry.svg)

## API загрузки телеметрических данных с устройств

Платформа имеет API для загрузки временных рядов данных в формате «ключ-значение». Гибкость и простота данного формата позволяет с легкостью бесшовно интегрировать систему с любыми IoT-устройствами. API для загрузки телеметрии специфичен для всех поддерживаемых сетевых протоколов. Вы можете ознакомиться с API и примерами использования на следующих страницах:

 - [справка о MQTT API](/docs/reference/mqtt-api/#telemetry-upload-api)
 - [справка о CoAP API](/docs/reference/coap-api/#telemetry-upload-api)
 - [справка о HTTP API](/docs/reference/http-api/#telemetry-upload-api)
  
## Сервис телеметрии

Данный сервис отвечает за сохранение временных рядов данных во внутреннем хранилище. Также он предоставляет серверный API, с помощью которого можно запрашивать обновления данных, а также подписаться на них.

### Внутреннее хранилище данных

Платформа использует NoSQL базу данных Cassandra или SQL БД для хранения всех данных.

Устройство, которое передает данные на сервер, сразу получает подтверждение сохранения отправленных данных в БД. 
С помощью современных MQTT-клиентов можно временно локально хранить недоставленные данные. Таким образом, даже если один из узлов платформы выйдет из строя, устройство не потеряет данные и сможет передать их на другие серверы.

Серверные приложения также могут публиковать телеметрию для разных сущностей и типов сущностей.

Также вы можете делать запросы в БД напрямую: Платформа предоставляет набор RESTful и Websocket API, которые упрощают этот процесс и соблюдают политику безопасности:

 - Тенант-администратор может получать данные всех сущностей, которые относятся к соответствующему тенанту
 - Пользователь с правами клиента может получать данные только тех сущностей, которые назначены ему
  
#### API поиска данных

Сервис телеметрии имеет REST API для получения данных отдельной сущности:

![image](/images/user-guide/telemetry-service/rest-api.png)

**Примечание:** вышеперечисленные API доступны в Swagger UI. Ознакомиться с базовой документацией по REST API можно по [ссылке](/docs/reference/rest-api/) 

##### API ключей временных рядов данных

Вы можете получать список ключей данных для разных типов сущностей и id сущностей с помощью GET-запросов вида 

```shell
http(s)://host:port/api/plugins/telemetry/{entityType}/{entityId}/keys/timeseries
```

{% capture tabspec %}get-telemetry-keys
A,get-telemetry-keys.sh,shell,resources/get-telemetry-keys.sh,/docs/user-guide/resources/get-telemetry-keys.sh
B,get-telemetry-keys-result.json,json,resources/get-telemetry-keys-result.json,/docs/user-guide/resources/get-telemetry-keys-result.json{% endcapture %}
{% include tabs.html %}

Поддерживаются следующие типы сущностей: TENANT, CUSTOMER, USER, DASHBOARD, ASSET, DEVICE, ALARM, ENTITY_VIEW

##### API значений временных рядов данных

Вы можете получать список последних значений для разных типов сущностей и id сущностей с помощью GET-запросов вида 
 
```shell
http(s)://host:port/api/plugins/telemetry/{entityType}/{entityId}/values/timeseries?keys=key1,key2,key3
```

{% capture tabspec %}get-latest-telemetry-values
A,get-latest-telemetry-values.sh,shell,resources/get-latest-telemetry-values.sh,/docs/user-guide/resources/get-latest-telemetry-values.sh
B,get-latest-telemetry-values-result.json,json,resources/get-latest-telemetry-values-result.json,/docs/user-guide/resources/get-latest-telemetry-values-result.json{% endcapture %}
{% include tabs.html %}

Поддерживаются следующие типы сущностей: TENANT, CUSTOMER, USER, DASHBOARD, ASSET, DEVICE, ALARM, ENTITY_VIEW

Также вы можете просматривать историю значений конкретного типа сущности или id сущности с помощью GET-запроса вида: 
 
```shell
http(s)://host:port/api/plugins/telemetry/{entityType}/{entityId}/values/timeseries?keys=key1,key2,key3&startTs=1479735870785&endTs=1479735871858&interval=60000&limit=100&agg=AVG
```

Используемые параметры:

 - **keys** - список телеметрических ключей, перечисленных через запятую.
 - **startTs** - – unix timestamp, определяющая начало интервала в миллисекундах.
 - **endTs** - unix timestamp, определяющая конец интервала в миллисекундах.
 - **interval** - интервал агрегации, измеряется в миллисекундах.
 - **agg** - функция агрегации: MIN, MAX, AVG, SUM, COUNT, NONE.
 - **limit** - максимальное число точек данных для возврата или интервалов для обработки.

С помощью параметров startTs, endTs и intervals платформа идентифицирует разделы агрегации или подзапросы и выполняет асинхронные запросы в БД, которые используют встроенные функции агрегации.

{% capture tabspec %}get-telemetry-values
A,get-telemetry-values.sh,shell,resources/get-telemetry-values.sh,/docs/user-guide/resources/get-telemetry-values.sh
B,get-telemetry-values-result.json,json,resources/get-telemetry-values-result.json,/docs/user-guide/resources/get-telemetry-values-result.json{% endcapture %}
{% include tabs.html %}

Поддерживаются следующие типы сущностей: TENANT, CUSTOMER, USER, DASHBOARD, ASSET, DEVICE, ALARM, ENTITY_VIEW

#### Websocket API

Websocket активно используется в Web UI платформы. Websocket  API имеет тот же функционал, что и REST API. С помощью него можно подписаться на обновления данных с устройств. Вы можете подключиться через websocket к сервису телеметрии по ссылке вида 

```shell
ws(s)://host:port/api/ws/plugins/telemetry?token=$JWT_TOKEN
```

Вы можете 

[передавать команды на подписку](https://github.com/thingsboard/thingsboard/blob/master/application/src/main/java/org/thingsboard/server/service/telemetry/cmd/TelemetryPluginCmdsWrapper.java) 
и
[получать обновления](https://github.com/thingsboard/thingsboard/blob/master/application/src/main/java/org/thingsboard/server/service/telemetry/sub/SubscriptionUpdate.java):

where 

 - **cmdId** - уникальный id команды (в рамках соответствующего websocket-соединения)
 - **entityType** - уникальный тип сущности. Поддерживаемые типы: TENANT, CUSTOMER, USER, DASHBOARD, ASSET, DEVICE, ALARM
 - **entityId** - уникальный id сущности
 - **keys** - список ключей данных, перечисленных через запятую
 - **timeWindow** - интервал для подписки на временные ряды данных, в миллисекундах. Данные будут приходить в рамках выбранного интервала  **[now()-timeWindow, now()]**
 - **startTs** - время начала интервала выборки для запроса истории данных, в миллисекундах.
 - **endTs** - время конца интервала выборки для запроса истории, в миллисекундах.
 
## Визуализация данных

Платформа предоставляет возможность конфигурировать и кастомизировать дашборды для визуализации данных. Подробнее можно ознакомиться на странице    
<p><a href="/docs/user-guide/visualization" class="button"> руководство по визуализации.</a></p>

## Движок правил

Платформа предоставляет возможность конфигурировать правила обработки данных. 
Каждое правило состоит из:

- фильтров, которые помогают фильтровать поступающие данные
- процессоров, с помощью которых можно создавать сигналы тревоги или дополнять поступающие данные серверными значениями
- действий для создания определенной логики работы с отфильтрованными данными.  – инструкции по движку правил.

Более подробную информацию можно найти на странице    
<p><a href="/docs/user-guide/rule-engine" class="button">Руководство по движку правил</a></p>
