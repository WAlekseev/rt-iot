---
layout: docwithnav
title: Отправка уведомлений и сигналов тревоги через Telegram-бот
---

* TOC
{:toc}

## Обзор

Telegram предоставляет возможность создавать Telegram-ботов, которые рассматриваются как сторонние приложения.
В данном руководстве содержится информация о том, как создать такого бота и настроить движок правил для отправки уведомлений в приложение Telegram с помощью расширения Rest API вызовов.

## Пример использования

Данный урок основан на уроке [создание и удаление сигналов тревоги](/docs/user-guide/rule-engine-2-0/tutorials/create-clear-alarms/#use-case). 
Мы повторно используем цепочки правил из вышеупомянутого руководства и добавим еще несколько узлов правил для интеграции с Telegram

Предположим, что ваше устройство использует датчик DHT22 для сбора и передачи показаний температуры на IoT платформу Ростелеком.
Датчик хорошо считывает показания температуры в диапазоне от -40 до 80°C. Нам нужно генерировать сигналы тревоги, если показания находятся за пределами установленного диапазона, и отправлять уведомления в приложение Telegram, когда создается сигнал тревоги.

В данном руководстве мы настроим движок правил для:

- Отправки уведомлений пользователю, если был создан сигнал тревоги.

- Добавления текущего типа сигнала тревоги и его отправителя в тело сообщения с помощью узла преобразования скрипта.

## Предусловия 

Предположим, что вы ознакомились со следующими руководствами и статьями:

  * [Начало работы](/docs/getting-started-guides/helloworld/).
  * [Обзор движка правил](/docs/user-guide/rule-engine-2-0/overview/).
  * [Создание и удаление сигналов тревоги](/docs/user-guide/rule-engine-2-0/tutorials/create-clear-alarms/).

## Флоу сообщений  

В этом разделе содержится объяснение назначения каждого узла в данном руководстве:

- Узел A: [**Узел Transformation script**](/docs/user-guide/rule-engine-2-0/transformation-nodes/#script-transformation-node) node.
  - Этот узел будет использоваться для создания тела сообщения в Telegram.  
- Узел Б: [**Узел REST API Call**](/docs/user-guide/rule-engine-2-0/external-nodes/#rest-api-call-node).
- Этот узел будет отправлять полезную нагрузку Telegram-сообщения в настроенную конечную точку REST. В нашем случае, это Telegram REST API.

## Создание Telegram-бота

[BotFather](https://telegram.me/botfather) - основной бот, которые поможет вам [создавать](https://core.telegram.org/bots#6-botfather) новых ботов и менять их настройки.

Как только создание бота будет завершено, вы можете сгенерировать токен авторизации для вашего нового бота.
Токен представляет собой строку, которая выглядит следующим образом - **'110201543:AAHdqTcvCH1vGWJxfSeofSAs0K5PALDsaw'** that is required to authorize the bot. 
    
Предусловия:

 - Telegram-бот создан

### Получение ID чата

В следующем шаге нам нужно извлечь ID чата. Он нужен для отравки сообщения по HTTP API. 

Существует несколько способов получить данный ID:

 - Сперва надо отправить боту любое сообщение:
 
    - в приватный чат; 
    
       ![image](/images/gateway/telegram-bot/private-msg-to-bot.png)    
    
    - в группу, в которую ваш бот был добавлен в качестве участника.
    
       ![image](/images/gateway/telegram-bot/msg-to-bot-in-chat.png)    
      
    <br> где **IoT_Bot** - имя Telegram-бота.

 - Затем откройте браузер и перейдите по следующей ссылке:

```bash
https://api.telegram.org/bot"YOUR_BOT_TOKEN"/getUpdates

"YOUR_BOT_TOKEN" должен быть заменен токеном аутентификации вашего бота, например:

https://api.telegram.org/bot110201543:AAHdqTcvCH1vGWJxfSeofSAs0K5PALDsaw/getUpdates
```



В исходящих данных вы сможете найти поле **'id'**. Это и будет, так называемый, chat_id. 

 - Первый вариант:

![image](/images/gateway/telegram-bot/first-option.png)

 - Второй вариант:

![image](/images/gateway/telegram-bot/second-option.png)

Затем вы можете начать настраивать движок правил для использования расширения Rest API Call.

## Настройка цепочек правил

В данном руководства мы ипользовали цепочку правил из урока руководства [создание и удаление сигналов тревоги](/docs/iot-gateway/create-clear-alarms).
Мы изменили цепочку правил **Create & Clear Alarms**, добавив узлы, о которых говорилось в разделе [Флоу сообщений](/docs/iot-gateway/integration-with-telegram-bot/#message-flow). Также мы переименовали данную цепочку правил так: **Create/Clear Alarms & send notifications to Telgram**.

<br/>На скриншотах показано, как данная цепочка должны выглядеть:
 
  - **Create/Clear Alarms & send notifications to Telgram:**

![image](/images/gateway/telegram-bot/send-to-telegram-chain.png)

 - **Root Rule Chain:**

![image](/images/gateway/telegram-bot/root-rule-chain.png)

<br/> 

Скачайте приложенний [**файл JSON**](/docs/iot-gateway/resources/create_clear_alarms___send_notifications_to_telgram.json) для цепочки правил **Create/Clear Alarms & send notifications to Telgram**.

В следующем разделе показано, как настроить данную цепочку с нуля.
<br/> 

### Настройка **Create/Clear Alarm & Send Email**

#### Добавление необходимых узлов

В данной цепочке вы создадите 2 узла по инструкциям, описанным в следующих разделах:
 
##### Узел A: **Transform Script**

- Добавьте узел **Transform Script** и соедините его с узлом **Create Alarm** типом отношений **Created**.
<br>Этот узел будет использоваться для создания тела уведомительного сообщения.
 <br>Тело шаблона должно содержать 2 параметра: 
  
   - chat_id;
  
   -  text.
  
   это пример исходящего сообщения:
  
```json
{"chat_id" : "PUT YOUR CHOSEN CHAT_ID", "text" : "SOME MESSAGE YOU WANT TO RECEIVE"}
```
  
 - Для этого используйте следующий скрипт: 
 
 {% highlight javascript %}
 var newMsg ={};
 newMsg.text = '"' +  msg.name + '"' + " alarm was created for device: " + '"' + metadata.deviceName + '"';
 newMsg.chat_id = 337878729; //has to be replaced by the actual chat id
 return {msg: newMsg, metadata: metadata, msgType: msgType};{% endhighlight %}
      
- Заполните поле названия так **New telegram message**.  
  
![image](/images/gateway/telegram-bot/transform-script.png)
   
##### Узел Б: **REST API Call**
- Добавьте узел**REST API Call** и соедините его с узлом **Transform Script** с помощью отношений типа **Success**.
<br>Этот узел будет отправлять полную полезную нагрузку сообщения в настроенную конечную точку REST. В нашем случае, это Telegram REST API.
  <br>В рамках этот руководства мы используем команду **'/sendMessage'**, чтобы вызвать Telegram Bot API для отправки сообщения.

  
- Заполните поля входными данными, приведенными в следующей таблице:

  <table style="width: 25%">
    <thead>
        <tr>
            <td><b>Field</b></td><td><b>Input Data</b></td>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Name</td>
            <td>REST API telegram Call</td>
        </tr>
        <tr>
            <td>Endpoint URL pattern</td>
            <td>https://api.telegram.org/bot"YOUR_BOT_TOKEN"/sendMessage</td>
        </tr>
        <tr>
            <td>Request method</td>
            <td>POST</td>
        </tr>
        <tr>
            <td>Header</td>
            <td>Content-Type</td>
        </tr>
        <tr>
            <td>Value</td>
            <td>application/json</td>
        </tr>
     </tbody>
  </table>
     
![image](/images/gateway/telegram-bot/rest-api-telegram-node.png)


## Отправка телеметрии и проверка

Для отправки телеметрии устройства мы будем использовать Rest APIs, [APIs загрузки телеметрии](/docs/reference/http-api/#telemetry-upload-api). Для этого нам нужно скопировать токен доступа устройства из **Thermostat Home**. 

![image](/images/gateway/telegram-bot/copy-token.png)


Допустим, температура столба = 99. Сигнал тревоги должен быть создан:

{% highlight bash %}
curl -v -X POST -d '{"temperature":99}' http://localhost:8080/api/v1/$ACCESS_TOKEN/telemetry --header "Content-Type:application/json"

**нужно заменить $ACCESS_TOKEN действительным токеном устройства**
{% endhighlight %}

Сообщение не будет отправлено в приложение Telegram при обновлении сигнала тревоги: только при его создании.

В конечном счете, мы сможем удостовериться, что в полученном сообщении переданы корректные значения:

- первый вариант:

![image](/images/gateway/telegram-bot/msg-received-first-way.png)


- второй вариант: 

![image](/images/gateway/telegram-bot/msg-received-second-way.png)


Также вы можете:

  - настроить детали сигнала тревоги в узле Create and Clear Alarm.
    
  - настроить дашборд, добавив виджет сигнала тревоги для визуализации данных.
  
  - определить дополнительную логику для работы с сигналами тревоги, например, отправку почтовых сообщений.

Чтобы узнать, как это сделать, перейдите по ссылкам в разделе **Смотрите также**.
  
<br/>

## Смотрите также

- [Создание и удаление сигналов тревоги: детали:](/docs/user-guide/rule-engine-2-0/tutorials/create-clear-alarms-with-details/#step-2-createupdate-alarm) - как настраивать функцию деталей сигнала тревоги в узле Alarm.

- [Создание и удаление сигналов тревоги: настройка дашборда](/docs/user-guide/rule-engine-2-0/tutorials/create-clear-alarms-with-details/#configure-device-and-dashboard) - как добавить виджет сигналов тревоги в дашборд.

- [Отправка сообщения по почте](/docs/user-guide/rule-engine-2-0/tutorials/send-email/).
 
