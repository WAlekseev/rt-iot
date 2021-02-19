---
layout: docwithnav
title: Начало работы с движком правил
description: Документация к IoT платформе Ростелеком
---

* TOC
{:toc}

## Что такое Rule Engine?
Движок правил - это простая в использовании платформа для построения рабочих процессов на основе событий. Существует 3 основных компонента:

- **Сообщение** - любое входящее событие. Это могут быть входящие данные от устройств, событие жизненного цикла устройства, событие REST API, RPC-запрос и т. д.
- **Узел правил** - функция, которая выполняется при входящем сообщении. Существует множество различных типов узлов, которые могут фильтровать, преобразовывать сообщения или отвечать на них некоторым действием. 
- **Цепочка правил** - узлы связаны друг с другом отношениями, поэтому исходящее сообщение от узла правил отправляется следующим подключенным узлам.


## Варианты использования
Это тонко настраиваемый фреймворк для комплексной обработки событий. Вот некоторые распространенные случаи использования, которые можно настроить с помощью цепочек правил:

- Проверка и модификация данных для входящей телеметрии или атрибутов перед их сохранением в базе данных.
- Копируйте телеметрию или атрибуты с устройств и переносите их на связанные ресурсы, чтобы объединить телеметрию. Например, данные с нескольких устройств могут быть объединены в связанный актив.
- Создавайте/обновляйте/удаляйте сигналы тревоги, исходя из определенных условий.
- Вызывайте действия, основанные на событиях жизненного цикла устройства. Например, создавайте оповещения, если устройство переключается в режим онлайн/оффлайн.
- Загружайте дополнительные данные, необходимые для обработки. Например, устанавливайте пороговое значение температуры для устройства, определенное в атрибутах тенанта или в устройствах клиентов.
- Инициируйте REST API-вызовы для внешних систем.
- Отправляйте электронные письма, когда происходит комплексное событие, и используйте атрибуты других сущностей в шаблоне электронного письма.
- Учитывайте предпочтения пользователя при обработке событий.
- Выполняйте RPC-вызовы на основе определенного условия.
- Интегрируйте платформу с внешними конвейерами, такими как Kafka, Spark, AWS services и т. д.

## Пример с "Hello-World"
Предположим, что ваше устройство использует датчик DHT22 для считывания и передачи температуры на IoT-платформу Ростелеком.  
Датчик DHT22 может измерять температуру от -40°C до +80°C.

В этом руководстве мы настроим узел правил для хранения всех температур в диапазоне от -40 до 80°C и занесем все остальные показания в системный журнал.

#### Добавление узла считывания температуры
В пользовательском интерфейсе платформы перейдите в раздел **Цепочка правил** и откройте раздел **Корневая цепочка правил**.

![image](/images/user-guide/rule-engine-2-0/tutorials/getting-started/initial-root-chain.png)

Перетащите узел правила **Script Filter** в цепочку. Откроется окно конфигурации узла. Мы будем использовать этот скрипт для проверки данных:

{% highlight javascript %}
return typeof msg.temperature === 'undefined' 
        || (msg.temperature >= -40 && msg.temperature <= 80);
{% endhighlight %}

![image](/images/user-guide/rule-engine-2-0/tutorials/getting-started/script-config.png)

Если свойство температуры не определено или температура допустима, скрипт вернет значение **True**, в противном случае он вернет значение **False**. 
Если скрипт возвращает **True**, то входящее сообщение будет перенаправлено на следующие узлы, связанные с True-отношениями.
 
Теперь мы сделаем так, чтобы все **запросы телеметрии** проходили через этот сценарий проверки. Нам нужно удалить существующую связь **Post Telemetry** 
между узлами **Message Type Switch** и **Save Telemetry**:

![image](/images/user-guide/rule-engine-2-0/tutorials/getting-started/remove-relation.png)
  
И соединить узлы **Message Type Switch** и **Script Filter** с помощью отношения **Post Telemetry**:
   
![image](/images/user-guide/rule-engine-2-0/tutorials/getting-started/realtion-window.png)

![image](/images/user-guide/rule-engine-2-0/tutorials/getting-started/connect-script.png)

Далее нам нужно соединить узлы **Script Filter** и **Save Telemetry**, используя отношение  **True**. Таким образом, вся действительная телеметрия будет сохранена:

![image](/images/user-guide/rule-engine-2-0/tutorials/getting-started/script-to-save.png)

Кроме того, мы свяжем узлы **Script Filter** и **Log Other**, используя отношение **False**. Так что вся недействительная телеметрия будет записана в системный журнал:

![image](/images/user-guide/rule-engine-2-0/tutorials/getting-started/false-log.png)

Нажмите кнопку «Сохранить», чтобы применить изменения.

#### Проверка результатов
Для проверки результатов нам нужно будет создать Устройство и отправить телеметрию на платформу. Перейдите в раздел **Устройства** и создайте новое:

![image](/images/user-guide/rule-engine-2-0/tutorials/getting-started/create-device.png)

Для публикации телеметрии устройства мы будем использовать [Rest API](/docs/reference/http-api/#telemetry-upload-api). Для этого нам нужно будет скопировать токен доступа устройства из **Thermostat Home**. 

![image](/images/user-guide/rule-engine-2-0/tutorials/getting-started/copy-access-token.png)

Давайте выставим температуру = 99.  Мы увидим, что телеметрия **не была** добавлена в разделе **Latest Telemetry** устройства:

{% highlight bash %}
curl -v -X POST -d '{"temperature":99}' http://localhost:8080/api/v1/$ACCESS_TOKEN/telemetry --header "Content-Type:application/json"
{% endhighlight %}

***вам нужно заменить $ACCESS_TOKEN на фактический токен устройства**

![image](/images/user-guide/rule-engine-2-0/tutorials/getting-started/empty-telemetry.png)


Выставим температуру = 24. Мы увидим, что телеметрия была успешно сохранена.

{% highlight bash %}
curl -v -X POST -d '{"temperature":24}' http://localhost:8080/api/v1/$ACCESS_TOKEN/telemetry --header "Content-Type:application/json"
{% endhighlight %}

![image](/images/user-guide/rule-engine-2-0/tutorials/getting-started/saved-ok.png)


## Смотрите также:

Вы можете пройти по следующим ссылкам для получения дополнительной информации об движке правил:

- [Обзор движка правил](/docs/user-guide/rule-engine-2-0/overview/)
- [Архитектура движка правил](/docs/user-guide/rule-engine-2-0/architecture/)
- [Отладка выполнения узла](/docs/user-guide/rule-engine-2-0/overview/#debugging)
- [Проверка входящей телеметрии](/docs/user-guide/rule-engine-2-0/tutorials/validate-incoming-telemetry/)
- [Преобразование входящей телеметрии](/docs/user-guide/rule-engine-2-0/tutorials/transform-incoming-telemetry/)
- [Преобразование телеметрии по последним записям](/docs/user-guide/rule-engine-2-0/tutorials/transform-telemetry-using-previous-record/)
- [Создание и удаление сигналов тревоги](/docs/user-guide/rule-engine-2-0/tutorials/create-clear-alarms/)
- [Отправка письма на сигнал тревоги](/docs/user-guide/rule-engine-2-0/tutorials/send-email/)
- [Создание сигнала тревоги, когда устройство отключено](/docs/user-guide/rule-engine-2-0/tutorials/create-inactivity-alarm/)
- [Проверка отношений между сущностями](/docs/user-guide/rule-engine-2-0/tutorials/check-relation-tutorial/)
- [RPC-запрос на связанные устройства](/docs/user-guide/rule-engine-2-0/tutorials/rpc-request-tutorial/)
- [Динамическая группировка устройства и удаление их из групп](/docs/user-guide/rule-engine-2-0/tutorials/add-devices-to-group/)
- [Объединение входящих потоков данных](/docs/user-guide/rule-engine-2-0/tutorials/aggregate-incoming-data-stream/)

<br/>
<br/>
