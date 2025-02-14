---
layout: docwithnav
title: Узлы аналитики
description: Узлы аналитики
---

Узлы аналитики используется для анализа потоковых или сохраняемых данных.

* TOC
{:toc}

# Aggregate Latest Node


![image](/images/user-guide/rule-engine-2-0/pe/nodes/analytics-aggregate-latest.png)

Периодически выполняет агрегацию атрибутов дочерних сущностей или последних временных рядов для указанного набора родительских сущностей.

Выполняет агрегацию атрибутов или последних временных рядов, извлеченных из дочерних сущностей с настраиваемым периодом.

Затем результат агрегации устанавливается в указанный целевой атрибут временных рядов родительской сущности.

Для каждой родительской сущности и агрегатного атрибута генерируется сообщение типа POST_TELEMETRY_REQUEST.

Настройки:

![image](/images/user-guide/rule-engine-2-0/pe/nodes/analytics-aggregate-latest-config.png)

- **Execution period value/time unit** - значение/единица времени периода выполнения - задает период выполнения задачи агрегирования.
- **Entities** - сущности - задает набор родительских сущностей, для которых должна выполняться агрегация. Может быть: 
    - **Single entity** - одна конкретная сущность. 
    - **Group of entities** - определенная группа сущностей. 
    - **Relations query** - набор сущностей, найденных запросом отношений, в который входят сущности, начиная с корневой.
- **Child entities** - Дочерние сущности - задает запрос отношений, используемый для поиска дочерних сущностей, начиная с родительской сущности. Следует указывать только в том случае, если для родительских сущностей выбран запрос на одну сущность или запрос на отношения. Для группы сущностей дочерние сущности выбираются из самой группы сущностей.
- **Aggregate latest mappings** - агрегация последнего отображения - таблица настроек отображения, определяющая правила агрегации дочерних атрибутов.

Mapping Configuration:

![image](/images/user-guide/rule-engine-2-0/pe/nodes/analytics-aggregate-latest-mapping-config.png)

- **Latest telemetry** - последняя телеметрия - указывает, следует ли агрегировать значение последней дочерней телеметрии или атрибута.
- **Source telemetry/attribute** - исходная телеметрия/атрибут – последняя дочерняя телеметрия или имя ключа атрибута.  
- **Attribute scope** - Область действия атрибута - указывает область действия дочернего атрибута в случае, если атрибут используется в качестве источника (последняя телеметрия не проверена).
- **Default value** - значение по умолчанию - числовое значение, используемое по умолчанию в тех случаях, когда исходный атрибут не определен или отсутствует для дочерней сущности.
- **Aggregation function** - функция агрегации - математическая функция, используемая для агрегации дочерних значений. Может быть:
    - **Minimum** - определяет минимальное значение атрибута среди всех дочерних сущностей. 
    - **Maximum** - - определяет максимальное значение атрибута среди всех дочерних сущностей.
    - **Sum** - вычисляет сумму значений атрибутов дочерних объектов.
    - **Average** - вычисляет среднее значение для значений атрибутов дочерних сущностей.
    - **Count** - выполняет подсчет дочерних сущностей. В этом случае исходная телеметрия/атрибут не используется и может быть пустой. 
    - **Count unique** - - выполняет подсчет уникальных значений атрибутов дочерних сущностей.
- **Target telemetry** - имя ключа целевой телеметрии родительской сущности, используемого для хранения результата агрегации.. 
- **Filter entities** - следует ли выполнять фильтрацию дочерних сущностей до агрегирования их значений атрибутов. 
- **Entity filter** - фильтр, используемый для фильтрации дочерних сущностей. Он состоит из двух частей:
    1. Список атрибутов / ключей последних временных рядов сущностей, которые должны быть извлечены и использованы в функции фильтра JavaScript.
    1. Функция фильтрации JavaScript должна возвращать результат фильтрации в виде логического значения.
    Она получает карту атрибутов, содержащую извлеченные атрибуты/значения последних временных рядов в качестве входного аргумента. Извлеченные значения атрибутов будут добавлены в атрибуты карты с помощью атрибутов/ключи последних временных рядов префиксом, обозначающим область действия:
        - общие атрибуты -> <code>shared_</code>
        - клиентские атрибуты -> <code>cs_</code>
        - серверные атрибуты -> <code>ss_</code>
        - телеметрия -> без префикса 

Для каждой родительской сущности узел будет генерировать и пересылать по цепочке **Success** новые сообщения типа POST_TELEMETRY_REQUEST, саму родительскую сущность в качестве отправителя и тело в формате json, содержащее целевую телеметрию с агрегированным значением. В случае, когда агрегация некоторых дочерних атрибутов завершится неудачей, узел сгенерирует сообщение об ошибке с указанием причины сбоя, и в сообщении передаст родительскую сущность в качестве отправителя. Сообщение об ошибке направляется по цепочке **Failure**.

<br/>

# Aggregate Stream Node

![image](/images/user-guide/rule-engine-2-0/pe/nodes/analytics-aggregate-stream.png)

Вычисляет MIN/MAX/SUM/AVG/COUNT/UNIQUE на основе входящего потока данных. Группирует входящий поток данных в интервалы на основе идентификатора отправителя сообщения (например, конкретного устройства, актива, клиента), функции агрегирования (например, “Average”, “Sum”, “Min”, “Max”), значения интервала агрегирования (например, 1 минута, 6 часов).


Интервалы периодически сохраняются по политике сохранения интервалов и значения проверок интервалов. Интервалы кэшируются в памяти по значению интервала TTL. Состояние интервалов сохраняется в формате объектов временных рядов по политике сохранения состояний и значений сохранения состояний. В случае отсутствия данных для определенных сущностей можно сгенерировать значения по умолчанию для этих сущностей. Для поиска этих сущностей можно установить флажок «Автоматически создавать интервалы» и настроить сущности интервалов.


Генерирует сообщения типа "POST_TELEMETRY_REQUEST’ с результатами агрегации для определенного интервала.


С помощью ниже указанных настроек можно рассчитать среднюю почасовую температуру и сохранить ее в течение одной минуты после окончания часового интервала. В случае, если для определенного интервала телеметрия поступит с задержкой, узел правила будет искать ее во внутреннем кэше или среди значений телеметрии.

![image](/images/user-guide/rule-engine-2-0/pe/nodes/analytics-aggregate-stream-config.png)
    
Результаты агрегирования будут сохраняться в базе данных раз в минуту. Кроме того, вы можете сохранить интервал для каждого нового сообщения, чтобы избежать потери данных в случае отключения сервера. В случае, если приборы для какого-либо здания не сообщают показания температуры, можно сгенерировать значение по умолчанию (равное нулю) для такого здания на каждом интервале, выбрав пункт “Создавать интервалы автоматически” и указав группу сущностей “Здания”.   

![image](/images/user-guide/rule-engine-2-0/pe/nodes/analytics-aggregate-stream-config-2.png)

# Alarms Count Node

![image](/images/user-guide/rule-engine-2-0/pe/nodes/analytics-alarms-count.png)

Узел периодически делает подсчет сигналов тревоги для выбранного набора объектов.

Выполняет подсчет сигналов для выбранных объектов, включая их дочерние объекты, если они заданы с настраиваемым периодом.

Предоставляет результаты подсчета сигналов, затем добавляет данные в указанный атрибут временных рядов объекта.

Генерирует сообщение типа POST_TELEMETRY_REQUEST для каждого объекта и результата подсчета сигналов.

Настройки:

![image](/images/user-guide/rule-engine-2-0/pe/nodes/analytics-alarms-count-config.png)

- **Execution period value/time unit** - значение/ единица времени периода выполнения - задает период выполнения задачи подсчета сигналов тревоги.
- **Entities** - Сущности - задает набор сущностей, для которых должен выполняться подсчет сигналов. Сущности могут быть: 
    - **Single entity** - одна конкретная сущность. 
    - **Group of entities** - определенная группа сущностей. 
    - **Relations query** - sнабор сущностей, найденных запросом отношений, начиная с корневой сущности.
- **Count alarms for child entities** - подсчет сигналов для дочерних сущностей - следует ли выполнять подсчет тревог для дочерних сущностей каждого найденного объекта.    
- **Child entities** - Дочерние сущности - задает запрос отношений, используемый для поиска дочерних сущностей, начиная с родительской сущности. Их следует указывать только в том случае, если установлен флажок «Считать сигналы тревоги для дочерних сущностей» и выбраны типа Одиночная сущность или Запрос отношений. В случае группы сущностей дочерние сущности выбираются из самой группы сущностей.
- **Alarms count mappings** - сопоставление количества сигналов тревоги - таблица настроек отображения, указывающая правила, используемые для подсчета сигналов тревоги.

Mapping Configuration:

![image](/images/user-guide/rule-engine-2-0/pe/nodes/analytics-alarms-count-mapping-config.png)

- **Target telemetry** - целевая телеметрия - имя ключа целевой телеметрии объекта. Ключ используется для хранения результатов подсчета сигналов тревоги.
- **Status filter** - фильтр статусов - список разрешенных статусов, по которым можно выполнять фильтрацию сигналов. Если фильтр не установлен, то будут выбраны сигналы тревоги с со всеми статусами. 
- **Severity filter** - фильтр серьезности - список разрешенных значений серьезности, по которым можно выполнять фильтрацию сигналов. Если фильтр не установлен, то будут выбраны сигналы тревоги со всеми степенями серьезности. 
- **Type filter** - фильтр по типу - список разрешенных типов, по которым можно выполнять фильтрацию сигналов. Если не указано, то будут выбраны сигналы тревоги со всеми типами.
- **Specify interval** - указать интервал - если установлен флажок, то будут выбраны только те сигналы тревоги, которые были созданы в течение указанного интервала. Если флажок не установлен, в выборку попадут все существующие сигналы тревоги.

Для каждого выбранного узла сущности будут генерироваться и пересылаться по цепочке **Success** новые сообщения типа POST_TELEMETRY_REQUEST, а также тело в формате json, в котором содержится целевая телеметрия со значением количества тревог. В случае, если произойдет сбой при подсчете количества сигналов тревоги для сущности, узел сгенерирует сообщение об ошибке с указанием причины сбоя и передаст сущность в качестве отправителя сообщения. Сообщение об отказе пересылается по цепочке **Failure**

<br/>
