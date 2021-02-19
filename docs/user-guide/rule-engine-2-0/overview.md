---
layout: docwithnav
title: Обзор движка правил
description: Документация к IoT платформе Ростелеком
---
* TOC
{:toc}

Система движка правил настраивается для обработки сложных событий. С помощью узла правил вы сможете фильтровать, пополнять и изменять входящие сообщения от IoT-устройств и связанных объектов. Также вы можете инициировать разные действия, например, получение уведомлений или установку связи со внешней системой.
  
## Ключевые концепции

#### Сообщения движка правил

Это упорядоченная, неизменяемая структура данных, в которой можно просмотреть разные системные сообщения. 

  * Поступающая с устройств [телеметрия](/docs/user-guide/telemetry/), [обновление атрибутов](/docs/user-guide/attributes/) или [RPC-вызовы](/docs/user-guide/rpc/);
  * События жизненного цикла сущностей: создан, обновлен, назначен, не назначен, обновление атрибутов;
  * События статусов устройств: подключен, отключен, активен, неактивен и т.д.;
  * Другие системные события.
  
Rule Engine Message содержит следующую информацию:

  * ID сообщения: генерируется на базе времени, универсальный уникальный идентификатор;
  * Источник, откуда поступило сообщение: идентификатор Устройства, Объекта или другой  [сущности](/docs/user-guide/entities-and-relations/);
  * Тип сообщения: "Post telemetry" или "Inactivity Event" и т.д.;
  * Полезная нагрузка сообщения: JSON-тело с актуальной нагрузкой сообщения;
  * Метаданные: список пар «ключ-значение» с дополнительными данными о сообщении. 

#### Узел правил

Это базовый компонент движка правил, который обрабатывает поступающие сообщения и генерирует одно и более исходящих сообщений. Rule Node -  основная логическая единица движка правил. Он может фильтровать, дополнять, менять входящие сообщения, выполнять действия или устанавливать связь со внешними системами.

#### Отношение узла правил

Узлы правил могут быть связаны с другими узлами правил. У каждой связи есть тип, лэйбл, с помощью которого определяется логическое значение связи. Когда узел генерирует исходящие сообщения, система всегда установит тип связи, чтобы направить сообщение к соответствующим узлам.
 
Отношения узла правил имеют статус либо «Success», либо "Failure". Узлы, которые представляют собой логические операции могут использовать значения «True» или «False». Некоторые специфичные rule nodes могут использовать разные типы связей, например: «Post Telemetry», «Attributes Updated» («Обновление атрибутов»), «Entity Created» («Сущность создана») и т.д.

#### Цепочка правил

Это логическая группа, состоящая из rule nodes и их связей. Например, на приведенной далее схеме rule chain выполняет следующие функции:

  * сохраняет все сообщения с телеметрией в базу данных;
  * поднимает "High Temperature Alarm" ("Сигнализация о высокой температуре"), если в сообщении передается отметка выше 50 градусов;
  * поднимает "Low Temperature Alarm" ("Сигнализация о низкой температуре"), если в сообщении передается отметка ниже -40 градусов;
  * логирует сбои в проверках температуры для выявления логических или синтаксических ошибок в скрипте. 

![image](/images/user-guide/rule-engine-2-0/rule-node-relations.png)

Тенант-администратор может установить одну **Root Rule Chain** ("Корневую цепочку правил") и, опционально, множество других цепочек правил.  
Root rule chain обрабатывает все входящие сообщения и может отправить их в другие цепочки правил для дополнительной обработки.
Другие rule chains также могут отправлять сообщения разным rule chains.

Например, нижеприведенная цепочка правил выполняет следующие функции:

  * поднимает "High Temperature Alarm", если в сообщении передается отметка выше 50 градусов;
  * удаляет событие "High Temperature Alarm", если отметка в сообщении опустится ниже 50 градусов;
  * отправит события о "Created" ("Созданных") и "Cleared" ("Удаленных") сигналах тревоги во внешнюю rule chain, которая обработает уведомления для показа соответствующим пользователям.
 
![image](/images/user-guide/rule-engine-2-0/rule-chain-references.png)

#### Результат обработки сообщений

Существует 3 возможных результата обработки сообщений: Success ("Успешно"), Failure ("Ошибка") и Timeout ("Превышено время ожидания"). 
Попытка обработки сообщения помечается статусом "Failure", если один из rule nodes возвращает для нее статус "Failure", и нет других rule nodes, которые могут её обработать. 
В случае, когда общее время обработки превышает установленный лимит, присваивается статус "Timeout".

Ниже представлена диаграмма с возможными сценариями:

![image](/images/user-guide/rule-engine-2-0/not-a-failure.png)

Если «Transformation» («Преобразование») скрипта завершается ошибкой, сообщению не присваивается статус «Failed», так как (???).
Если «Transformation» скрипт завершается успешно, он будет отправлен в «External System» («Внешнюю систему») с помощью REST API. Если внешняя система перегружена, запрос может «повисеть» некоторое время. Предположим, что общее время ожидания обработки пакета сообщений составляет 20 секунд. Опустим время выполнения transformation-скрипта, поскольку его значение < 1ms. Если внешняя система вернет ответ в течение 20 секунд, сообщение успешно обработается. То же самое и для вызова «Save to DB» («Сохранить в БД»): если он выполнится успешно, сообщение успешно обработается. Но если внешняя система не вернет ответ в течение 20 секунд, то попытке обработки сообщения присвоится статус «time-out». То же и в случае неудачного выполнения вызова «Save to DB»: сообщению присвоится статус failed.

#### Очередь движка правил

Движок правил при запуске подписывается на очереди и запрашивает новые сообщения. «Главная» тема используется в качестве основной точки входа для новых входящих сообщений. Вы можете настроить множество очередей с помощью thingsboard.yml или переменных окружения. Затем вы можете переместить сообщение в другую тему с помощью “Checkpoint” node. Он автоматически подтвердит, что сообщение перемещено.

Очередь состоит из следующих параметров:

 * name - используется для статистики и логирования;
 * topic - для выполнения очереди (генерирование и получение сообщений).
 * poll-interval - период между запросами сообщений, если не были получены новые сообщения, изменяется в миллисекундах;
 * partitions - число разделов, связанной с этой очередью. Используется для масштабирования количества сообщений, которые могут быть обработаны параллельно;
 * pack-processing-timeout - интервал для обработки пакета сообщений, которые вернулись от потребителей, измеряется в миллисекундах;
 * submit-strategy - определяет логику и порядок отправки сообщений в rule engine. Подробнее в отдельном параграфе ниже.
 * processing-strategy - определяет логику подтверждения сообщений. Подробнее в отдельном параграфе ниже.
  
##### Стратегия очереди отправки

Сервис Rule Engine постоянно запрашивает сообщения, относящиеся к определенной теме. И когда Клиент отправляет список сообщений, создается объект TbMsgPackProcessingContext. Queue submit strategy контролирует процесс передачи сообщений из TbMsgPackProcessingContext в rule chains. Есть 5 доступных стратегий:

 * BURST - все сообщения передаются в цепочка правил в порядке их поступления.  
 * BATCH - сообщения группируются в пакеты с помощью параметра конфигурации "queue.rule-engine.queues[queue index].batch-size". Новый пакет не будет передан до тех пор, пока не будет получено подтверждение получения предыдущего.
 * SEQUENTIAL_BY_ORIGINATOR - сообщения передаются последовательно от разных сущностей (источников, откуда поступают сообщения). Новое сообщение для устройства А не передается до тех пор, пока не будет получено предыдущее. 
 * SEQUENTIAL_BY_TENANT - сообщения передаются последовательно от разных тенантов (владельцев источников, откуда поступают сообщения). Новое сообщение для тенанта А не будет передаваться до тех пор, пока не будет получено предыдущее). 
 * SEQUENTIAL  - сообщения отправляются последовательно.  Новое сообщение не будет передано до тех пор, пока не будет получено предыдущее. В связи с этим процесс обработки длится достаточно долго.
 
В [руководстве](/docs/user-guide/rule-engine-2-5/tutorials/queues-for-synchronization/) можно ознакомиться с примерами использования submit strategy.

##### Стратегия обработки очереди
  
Processing Strategy контролирует процесс повторной обработки сообщений, первая обработка которых завершилась ошибкой или таймаутом. Есть 5 доступных стратегий:

 * SKIP_ALL_FAILURES - позволяет игнорировать все ошибки и таймауты, что приведет к потере всех сообщений. Например, если БД перестала работать, сообщения не будут сохраняться, но будут помечаться как "подтвержденные" и затем удаляться из очереди. Эта стратегия нацелена скорее на обратную совместимость с предыдущими релизами и средой разработки и демонстрационной средой
 * RETRY_ALL - повторить попытку получения всех сообщений из пакета обработки. Если обработка 1 из 100 сообщений завершится ошибкой, стратегия снова обработает 100 сообщений и повторно отправит их в Rule Engine. 
 * RETRY_FAILED - повторить обработку всех сообщений, которая при первой попытке завершилась ошибкой. Если обработка 1 из 100 сообщений завершится ошибкой, стратегия снова обработает 1 сообщение и повторно отправит пакет в Rule Engine. Если обработка завершится таймаутом, то повторная обработка выполняться не будет.
 * RETRY_TIMED_OUT - повторить обработку сообщений, которая при первой попытке завершилась таймаутом. Если 1 из 100 сообщений вернет таймаут, стратегия снова обработает только 1 сообщение и повторно отправит пакет в Rule Engine. Если обработка завершится таймаутом, то повторная обработка выполняться не будет.
 * RETRY_FAILED_AND_TIMED_OUT - rповторить попытку обработки всех сообщений, которая ранее завершилась ошибкой или таймаутом.
 
Все "RETRY*" стратегии поддерживают важные параметры конфигурации:
 
 * retries - Число повторных попыток, 0 не ограничено;
 * failure-percentage - Пропустить повторные попытки, если процент завершившихся ошибкой или таймаутом сообщений меньше определенного процента сообщений;
 * pause-between-retries - Время ожидания ответа от Клиента до выполнения повторных попыток, измеряется в секундах;
 
 В следующем [руководстве](/docs/user-guide/rule-engine-2-5/tutorials/queues-for-message-reprocessing/) можно ознакомиться с примерами использования стратегии обработки.

##### Очереди по умолчанию

По умолчанию настроены три очереди: Main, HighPriority и SequentialByOriginator. Они различаются стратегиями отправки и обработки. Как правило, движок правил обрабатывает сообщения из Main-темы и может дополнительно помещать их в другие темы, используя «Checkpoint» rule node. 
Main-тема по умолчанию игнорирует сообщения, обработка которых завершилась ошибкой. Это сделано для обратной совместимости с предыдущими релизами. Однако вы можете изменить настройки, что подразумевает возможные риски.  
Стоит иметь ввиду, что, если обработка одного сообщения не выполнится из-за какой-либо ошибки в вашем rule node-скрипте, это может помешать обработке следующих сообщений.
С помощью специального [дашборда](/docs/user-guide/rule-engine-2-0/overview/#rule-engine-statistics) можно следить за обработкой данных и ошибками в движке правил. 

HighPriority-тема может быть использована для отправки сигналов тревоги и других критичных этапов обработки. Сообщения в теме HighPriority постоянно перерабатываются в случае сбоя до тех пор, пока обработка не завершится успешно. Это может быть полезно в тех случаях, если ваш SMTP-сервер или внешняя система вышли из строя. The Rule Engine будет повторять попытку отправки сообщения, пока оно не будет обработано. 

С помощью темы SequentialByOriginator вы убедитесь в том, что сообщения обработались в правильном порядке. Сообщения от одной и той же сущности будут обработаны в том порядке, в котором они попадают в очередь. Rule Engine не отправит новое сообщение в rule chain до тех пор, пока предыдущее сообщение от той же сущности не будет получено.

## Предопределенные типы сообщений

Список предопределенных Типов Сообщений представлен в таблице:

<table>
  <thead>
      <tr>
          <td><b>Message Type</b></td><td><b>Display Name</b></td><td><b>Description</b></td><td><b>Message metadata</b></td><td><b>Message payload</b></td>
      </tr>
  </thead>
  <tbody>
      <tr>
          <td>POST_ATTRIBUTES_REQUEST</td>
          <td><b>Post attributes</b></td>
          <td>Request from device to publish <a href="/docs/user-guide/attributes/#attribute-types">client side</a> attributes (see <a href="/docs/reference/mqtt-api/#publish-attribute-update-to-the-server">attributes api</a> for reference)</td>
          <td><b>deviceName</b> - originator device name,<br><b>deviceType</b> - originator device type</td>
          <td>key/value json: <br> <code style="font-size: 12px;">{ <br> &nbsp;&nbsp;"currentState": "IDLE" <br> }</code></td>
      </tr>
      <tr>
          <td>POST_TELEMETRY_REQUEST</td>
          <td><b>Post telemetry</b></td>
          <td>Request from device to publish telemetry (see <a href="/docs/reference/mqtt-api/#telemetry-upload-api">telemetry upload api</a> for reference)</td>
          <td><b>deviceName</b> - originator device name,<br><b>deviceType</b> - originator device type,<br><b>ts</b> - timestamp (milliseconds)</td>
          <td>key/value json: <br> <code style="font-size: 12px;">{ <br> &nbsp;&nbsp;"temperature": 22.7 <br> }</code></td>
      </tr>
      <tr>
          <td>TO_SERVER_RPC_REQUEST</td>
          <td><b>RPC Request from Device</b></td>
          <td>RPC request from device (see <a href="/docs/reference/mqtt-api/#client-side-rpc">client side rpc</a> for reference)</td>
          <td><b>deviceName</b> - originator device name,<br><b>deviceType</b> - originator device type,<br><b>requestId</b> - RPC request Id provided by client</td>
          <td>json containing <b>method</b> and <b>params</b>: <br> <code style="font-size: 12px;">{ <br> &nbsp;&nbsp;"method": "getTime", <br>&nbsp;&nbsp;"params": { "param1": "val1" } <br> }</code></td>
      </tr>
      <tr>
          <td>RPC_CALL_FROM_SERVER_TO_DEVICE</td>
          <td><b>RPC Request to Device</b></td>
          <td>RPC request from server to device (see <a href="/docs/user-guide/rpc/#server-side-rpc-api">server side rpc api</a> for reference)</td>
          <td><b>requestUUID</b> - internal request id used by sustem to identify reply target,<br><b>expirationTime</b> - time when request will be expired,<br><b>oneway</b> - specifies request type: <i>true</i> - without response, <i>false</i> - with response</td>
          <td>json containing <b>method</b> and <b>params</b>: <br> <code style="font-size: 12px;">{ <br> &nbsp;&nbsp;"method": "getGpioStatus", <br>&nbsp;&nbsp;"params": { "param1": "val1" } <br> }</code></td>
      </tr>
      <tr>
          <td>ACTIVITY_EVENT</td>
          <td><b>Activity Event</b></td>
          <td>Event indicating that device becomes active</td>
          <td><b>deviceName</b> - originator device name,<br><b>deviceType</b> - originator device type</td>
          <td>json containing device activity information: <br> <code style="font-size: 12px;">{<br> &nbsp;&nbsp;"active": true,<br> &nbsp;&nbsp;"lastConnectTime": 1526979083267,<br> &nbsp;&nbsp;"lastActivityTime": 1526979083270,<br> &nbsp;&nbsp;"lastDisconnectTime": 1526978493963,<br> &nbsp;&nbsp;"lastInactivityAlarmTime": 1526978512339,<br> &nbsp;&nbsp;"inactivityTimeout": 10000<br>}</code></td>
      </tr>
      <tr>
          <td>INACTIVITY_EVENT</td>
          <td><b>Inactivity Event</b></td>
          <td>Event indicating that device becomes inactive</td>
          <td><b>deviceName</b> - originator device name,<br><b>deviceType</b> - originator device type</td>
          <td>json containing device activity information, see <b>Activity Event</b> payload</td>
      </tr>
      <tr>
          <td>CONNECT_EVENT</td>
          <td><b>Connect Event</b></td>
          <td>Event produced when device is connected</td>
          <td><b>deviceName</b> - originator device name,<br><b>deviceType</b> - originator device type</td>
          <td>json containing device activity information, see <b>Activity Event</b> payload</td>
      </tr>
      <tr>
          <td>DISCONNECT_EVENT</td>
          <td><b>Disconnect Event</b></td>
          <td>Event produced when device is disconnected</td>
          <td><b>deviceName</b> - originator device name,<br><b>deviceType</b> - originator device type</td>
          <td>json containing device activity information, see <b>Activity Event</b> payload</td>
      </tr>
      <tr>
          <td>ENTITY_CREATED</td>
          <td><b>Entity Created</b></td>
          <td>Event produced when new entity was created in system</td>
          <td><b>userName</b> - name of the user who created the entity,<br><b>userId</b> - the user Id</td>
          <td>json containing created entity details: <br> <code style="font-size: 12px;">{<br> &nbsp;&nbsp;"id": {<br> &nbsp;&nbsp;&nbsp;&nbsp;"entityType": "DEVICE",<br> &nbsp;&nbsp;&nbsp;&nbsp;"id": "efc4b9e0-5d0f-11e8-8559-37a7f8cdca74"<br> &nbsp;&nbsp;},<br> &nbsp;&nbsp;"createdTime": 1526918366334,<br> &nbsp;&nbsp;...<br> &nbsp;&nbsp;"name": "my-device",<br> &nbsp;&nbsp;"type": "temp-sensor"<br>}</code></td>
      </tr>
      <tr>
          <td>ENTITY_UPDATED</td>
          <td><b>Entity Updated</b></td>
          <td>Event produced when existing entity was updated</td>
          <td><b>userName</b> - name of the user who updated the entity,<br><b>userId</b> - the user Id</td>
          <td>json containing updated entity details, see <b>Entity Created</b> payload</td>
      </tr>
      <tr>
          <td>ENTITY_DELETED</td>
          <td><b>Entity Deleted</b></td>
          <td>Event produced when existing entity was deleted</td>
          <td><b>userName</b> - name of the user who deleted the entity,<br><b>userId</b> - the user Id</td>
          <td>json containing deleted entity details, see <b>Entity Created</b> payload</td>
      </tr>
      <tr>
          <td>ENTITY_ASSIGNED</td>
          <td><b>Entity Assigned</b></td>
          <td>Event produced when existing entity was assigned to customer</td>
          <td><b>userName</b> - name of the user who performed assignment operation,<br><b>userId</b> - the user Id,<br><b>assignedCustomerName</b> - assigned customer name,<br><b>assignedCustomerId</b> - Id of assigned customer</td>
          <td>json containing assigned entity details, see <b>Entity Created</b> payload</td>
      </tr>
      <tr>
          <td>ENTITY_UNASSIGNED</td>
          <td><b>Entity Unassigned</b></td>
          <td>Event produced when existing entity was unassigned from customer</td>
          <td><b>userName</b> - name of the user who performed unassignment operation,<br><b>userId</b> - the user Id,<br><b>unassignedCustomerName</b> - unassigned customer name,<br><b>unassignedCustomerId</b> - Id of unassigned customer</td>
          <td>json containing unassigned entity details, see <b>Entity Created</b> payload</td>
      </tr>
      <tr>
          <td>ADDED_TO_ENTITY_GROUP</td>
          <td><b>Added to Group</b></td>
          <td>Event produced when entity was added to <a href="/docs/user-guide/groups/">Entity Group</a>. This Message Type is specific to <a href="/products/thingsboard-pe/">ThingsBoard PE</a>.</td>
          <td><b>userName</b> - name of the user who performed assignment operation,<br><b>userId</b> - the user Id,<br><b>addedToEntityGroupName</b> - entity group name,<br><b>addedToEntityGroupId</b> - Id of entity group</td>
          <td>empty json payload</td>
      </tr>
      <tr>
          <td>REMOVED_FROM_ENTITY_GROUP</td>
          <td><b>Removed from Group</b></td>
          <td>Event produced when entity was removed from <a href="/docs/user-guide/groups/">Entity Group</a>. This Message Type is specific to <a href="/products/thingsboard-pe/">ThingsBoard PE</a>.</td>
          <td><b>userName</b> - name of the user who performed unassignment operation,<br><b>userId</b> - the user Id,<br><b>removedFromEntityGroupName</b> - entity group name,<br><b>removedFromEntityGroupId</b> - Id of entity group</td>
          <td>empty json payload</td>
      </tr>
      <tr>
          <td>ATTRIBUTES_UPDATED</td>
          <td><b>Attributes Updated</b></td>
          <td>Event produced when entity attributes update was performed</td>
          <td><b>userName</b> - name of the user who performed attributes update,<br><b>userId</b> - the user Id,<br><b>scope</b> - updated attributes scope (can be either <b>SERVER_SCOPE</b> or <b>SHARED_SCOPE</b>)</td>
          <td>key/value json with updated attributes: <br> <code style="font-size: 12px;">{ <br> &nbsp;&nbsp;"softwareVersion": "1.2.3" <br> }</code></td>
      </tr>
      <tr>
          <td>ATTRIBUTES_DELETED</td>
          <td><b>Attributes Deleted</b></td>
          <td>Event produced when some of entity attributes were deleted</td>
          <td><b>userName</b> - name of the user who deleted attributes,<br><b>userId</b> - the user Id,<br><b>scope</b> - deleted attributes scope (can be either <b>SERVER_SCOPE</b> or <b>SHARED_SCOPE</b>)</td>
          <td>json with <b>attributes</b> field containing list of deleted attributes keys: <br> <code style="font-size: 12px;">{ <br> &nbsp;&nbsp;"attributes": ["modelNumber", "serial"] <br> }</code></td>
      </tr>
      <tr>
        <td>ALARM</td>
        <td><b>Alarm event</b></td>
        <td>Event produced when an alarm was created, updated or deleted</td>
        <td> 
            All fields from original Message Metadata
            <br><b>isNewAlarm</b> - true if a new alram was just created
            <br><b>isExistingAlarm</b> - true if an alarm is existing already
            <br><b>isClearedAlarm</b> - true if an alarm was cleared
        </td>
        <td>json containing created alarm details: <br><code style="font-size: 12px;">
        {
          <br> &nbsp;&nbsp;"tenantId": {
            <br> &nbsp;&nbsp;&nbsp;&nbsp; ...
          <br> &nbsp;&nbsp;},
          <br> &nbsp;&nbsp;"type": "High Temperature Alarm",
          <br> &nbsp;&nbsp;"originator": {
            <br> &nbsp;&nbsp;&nbsp;&nbsp; ...
          <br> &nbsp;&nbsp;},
          <br> &nbsp;&nbsp;"severity": "CRITICAL",
          <br> &nbsp;&nbsp;"status": "CLEARED_UNACK",
          <br> &nbsp;&nbsp;"startTs": 1526985698000,
          <br> &nbsp;&nbsp;"endTs": 1526985698000,
          <br> &nbsp;&nbsp;"ackTs": 0,
          <br> &nbsp;&nbsp;"clearTs": 1526985712000,
          <br> &nbsp;&nbsp;"details": {
            <br> &nbsp;&nbsp;&nbsp;&nbsp;"temperature": 70,
            <br> &nbsp;&nbsp;&nbsp;&nbsp;"ts": 1526985696000
          <br> &nbsp;&nbsp;},
          <br> &nbsp;&nbsp;"propagate": true,
          <br> &nbsp;&nbsp;"id": "33cd8999-5dac-11e8-bbab-ad47060c9431",
          <br> &nbsp;&nbsp;"createdTime": 1526985698000,
          <br> &nbsp;&nbsp;"name": "High Temperature Alarm"
        <br>}
        </code>
        </td>
      </tr>
      <tr>
          <td>REST_API_REQUEST</td>
          <td><b>REST API Request to Rule Engine</b></td>
          <td>Event produced when user executes REST API call</td>
          <td><b>requestUUID</b> - the unique request id,<br><b>expirationTime</b> - the expiration time of the request</td>
          <td>json with request payload</td>
      </tr>
   </tbody>
</table>
 
## Типы узлов правил

Все доступные узлы правил сгруппированы в соответствии с их типами:

  * [**Узлы фильтрации**](/docs/user-guide/rule-engine-2-0/filter-nodes/) используются для фильтрации и маршрутизации сообщений;
  * [**Узлы насыщения**](/docs/user-guide/rule-engine-2-0/enrichment-nodes/) используются для обновления мета-данных входящих сообщений;
  * [**Узлы преобразования**](/docs/user-guide/rule-engine-2-0/transformation-nodes/) используются для изменения полей входящих сообщений (например, Originator, Type, Payload, Metodata);
  * [**Узлы действий**](/docs/user-guide/rule-engine-2-0/action-nodes/) выполняют разные действия в зависимости от получаемых сообщений;
  * [**Сторонние узлы**](/docs/user-guide/rule-engine-2-0/external-nodes/) используются для взаимодействия со внешними системами.

## Настройки узлов

У любого Rule Node могут иметь определенные параметры конфигурации, которые зависят от Rule Node Implementation (реализация узла правил). 
Например, «Filter – script» rule node настраивается через кастомную JS-функцию, которая обрабатывает поступающие данные. «External – send email» node конфигурация позволяет задавать настройки параметров соединения с почтовым сервером
  
Окно конфигурации Rule Node можно открыть двойным щелчком по node в редакторе цепочки правил:  
  
![image](/images/user-guide/rule-engine-2-0/rule-node-configuration.png)

### Тестовые JavaScript-функции

Некоторые rule nodes имеют определенные UI-функции, с помощью которых пользователи могут тестировать JS-функции.  
Кликнув на **Test Filter Function**, вы запустите редактор JS. В нем вы можете подменить входные параметры и проверить результаты функции.
    
![image](/images/user-guide/rule-engine-2-0/rule-node-test-function.png)

Вы можете установить:

- **Тип сообщения** слева вверху.
- **Полезную нагрузку сообщения** слева на странице редактирования сообщения.
- **Metadata** справа в разделе Метаданные.
- Текущий **JS script** разделе Фильтрация.

Нажав **Test**, вы увидите результаты теста в разделе **Результат** справа.

## Статистика узла правил

С помощью дефолтного дашборда вы можете отслеживать статистику по Rule Engine. Он автоматически подгружается для каждого тенанта. Сбор статистики включен по умолчанию и контролируется с помощью настроек конфигурации.

На скриншоте показано, как визуализируются данные из статистики по ошибкам обработки и причинам этих ошибок:

![image](/images/user-guide/rule-engine-2-0/rule-engine-stats-dashboard.png)

## Отладка

С помощью ThingsBoard вы можете просматривать входящие и исходящие сообщения для каждого Rule Node. Чтобы включить режим отладки, пользователь должен убедиться, что в главном окне конфигурации выставлен значок «Режим отладки».

Включив режим отладки, пользователь может просматривать информацию о входящих и исходящих сообщениях до тех пор (????). На скриншоте показан пример того, как выглядит режим отладки сообщений:
  
![image](/images/user-guide/rule-engine-2-0/rule-node-debug.png)  

## Импорт/экспорт

Вы можете экспортировать и импортировать свою цепочку правил в формате JSON, а также импортировать её в уже заведенную карточку для цепочки правил или создать новую и импортировать в неё.

Чтобы экспортировать цепочку правил, надо перейти на страницу **Цепочки правил** и нажать на кнопку Экспорт, расположенную на нужной вам карточке цепочки правил.
 
![image](/images/user-guide/rule-engine-2-0/rule-chain-export.png)

Чтобы импортировать цепочку правил, надо перейти на страницу **Цепочки правил** и нажать на кнопку "+" в правом нижнем углу экрана, а затем нажать на кнопку импорта.

## Архитектура

На следующей [странице](/docs/user-guide/rule-engine-2-0/architecture/) вы сможете узнать подробности о внутренних компонентах механизма правил. 

## Пользовательские вызовы REST API в движок правил


В системе платформы есть API для отправки кастомных REST API вызовов в движок правил, обработки полезной нагрузки запроса и возврата результата обработки в теле ответа. Эти функции могут быть полезны в следующих случаях:
 
 - чтобы расширить существующий REST API с помощью кастоманых API-вызовов;
 - чтобы добавлять в REST API-вызовы атрибуты устройств/объектов и отправлять их во внешние системы для сложной обработки;
 - чтобы предоставить ваш кастоманый API для ваших кастомных виджетов.
 
Чтобы выполнить REST API-вызов, вы можете использовать rule-engine-controller [REST APIs](/docs/reference/rest-api/): 
 
![image](/images/user-guide/rule-engine-2-0/rest-api.png) 

Примечание: id сущности, указанный в вызове, будет использоваться в качестве источника, откуда отправляется сообщение движка правил. Если вы не зададите параметр id для сущности, то в качестве отправителя выступит id пользователя, под которым вы используете платформу.

## Руководства

С помощью следующих инструкций вы сможете создать цепочи правил на конкретном примере:

  * [**Преобразование входящих сообщений с устройств**](/docs/user-guide/rule-engine-2-0/tutorials/transform-incoming-telemetry/) 
  * [**Преобразование входящих сообщений на основе предыдущих сообщений с устройств**](/docs/user-guide/rule-engine-2-0/tutorials/transform-telemetry-using-previous-record/) 
  * [**Создание и удаление сигналов тревоги при поступлении сообщений с устройств**](/docs/user-guide/rule-engine-2-0/tutorials/create-clear-alarms/)
  * [**Отправка электронных писем клиенту по сигналу тревоги с устройства**](/docs/user-guide/rule-engine-2-0/tutorials/send-email/) 
  * [**Отправка сообщений между связанными устройствами**](/docs/user-guide/rule-engine-2-0/tutorials/rpc-reply-tutorial/)
  
Больше туториалов доступно по ссылке [ссылке](https://thingsboard.io/docs/guides/).
 
