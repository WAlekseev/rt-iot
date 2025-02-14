---
layout: docwithnav
title: Групповая настройка
description: Документация к IoT платформе Ростелеком
---
* TOC
{:toc}

## Обзор групповой настройки

IoT-платформа Ростелеком предоставляет возможность групповой настройки с использованием CSV-файла следующих типов сущностей: 
 - **Устройства** 
 - **Активы**


Различные сущности могут иметь следующие параметры:

 - **Атрибуты** - статические или полустатические пары «ключ-значение», связанные с сущностями IoT-платформы. Например, серийный номер, модель, версия прошивки;
 - **Телеметрические данные** - временной ряд данных, доступный для хранения, визуализации и обработки. Например, температура, влажность, уровень заряда батареи и т.д.;
 - **Учетные данные** - используются устройством для подключения к серверу платформы с помощью приложений, запущенных на устройстве.
   
## Импортируйте сущности

Чтобы создать несколько сущностей одновременно, вам нужно создать CSV-файл, где каждая строка будет отвечать за создание одной сущности с заданными параметрами.<br/>
Если вам не нужно добавлять настройки для конкретного объекта, оставьте эту ячейку пустой.<br/> 
Существует три зарезервированных названия параметров: имя, тип и метка; каждое имеет предопределенный тип столбца.

### Шаг 1: Выберите файл

Загрузите CSV-файл в систему.

<img data-gifffer="/images/user-guide/bull-provisioning/bulk-provision-step-1.gif" />

### Шаг 2: Импортируйте настройки

Для загруженного файла необходимо настроить следующие параметры:

 - **CSV delimiter** - разделительный символ между значениями в строке данных;
 - **Первая строка содержит имена столбцов** - если эта опция активирована, то первая строка файла будет использоваться в качестве значения по умолчанию для имен параметров на следующем шаге;
 - **Обновить атрибуты/телеметрию** -  если этот параметр активен, то для всех сущностей, имена которых совпадают с имеющимся в системе IoT Ростелеком сущностями, значения параметров будут обновлены. В противном случае для всех сущностей, имена которых уже записаны в системе платформы, будет высвечено сообщение об ошибке.
 
<img data-gifffer="/images/user-guide/bull-provisioning/bulk-provision-step-2.gif" /> 

### Шаг 3: Выберите тип столбцов

На этом этапе вам необходимо установить соответствие между столбцами загруженного файла и типом данных в IoT-платформе Ростелеком. Вы также можете установить/изменить имя по умолчанию для attribute/telemetry-ключа.

<img data-gifffer="/images/user-guide/bull-provisioning/bulk-provision-step-3.gif" />  

### Шаг 4: Создание новых сущностей

Обработка входных данных.

### Шаг 5: Готово
  
Результат выполнения запроса: количество созданных/обновленных сущностей и количество ошибок, возникших во время выполнения.

<img data-gifffer="/images/user-guide/bull-provisioning/bulk-provision-step-5.gif" />


## Вариант использования

Предположим, мы хотим создать одновременно 10 устройств и дать им токен доступа.<br/><br/>
Пример файла:
{% capture tabspec %}sample-file
A,test-import-device.csv,text,resources/test-import-device.csv,/docs/user-guide/resources/test-import-device.csv{% endcapture %} 
{% include tabs.html %}
**Примечание:** файл должен содержать не менее двух столбцов: имя сущности и её тип.<br/>

<br/>Файл был создан с помощью редактора CSV-файлов, он содержит данные для 10 устройств. Кроме того, параметр **Data2** был опущен для устройства**Device 5**и для него создан не будет.

#### Загрузите файл

Перейдите в раздел **Устройства** -> **Импорт устройства**

Загрузите тестовый файл: **test-import-device.csv**

![image](/images/user-guide/bull-provisioning/import-device-select-file.png)

#### Импортируйте настройки

 - **CSV delimiter** - выберите редакторский разделительный символ. В тестовом файле разделителем является “,”;
 - **Первая строка содержит имена столбцов** - тестовый файл содержит имена столбцов, поэтому мы оставляем этот параметр выбранным;
 - **Обновить атрибуты/телеметрию** - снимите этот флажок, так как мы собираемся добавлять новые устройства, а не обновлять параметры для существующих устройств в IoT-платформе Ростелеком.
 
![image](/images/user-guide/bull-provisioning/import-device-config.png)

#### Выберите типы столбцов

В первом столбце таблицы отображается первая строка файла, содержащая данные.<br/>
Поскольку флажок **Первая строка содержит имена столбцов** был установлен на предыдущем шаге, значения для третьего столбца уже были инициированы на основе первой строки документа.<br/>
Изменим некоторые атрибуты. Измените тип столбца в третьей строке на **Временные ряды** и задайте значение attributes/telemetry-ключа, например, **Температуру**.<br/>
Последняя строка в следующей таблице отвечает за токен, поэтому измените **Серверный атрибут** на **Токен доступа**. <br/><br/>

![image](/images/user-guide/bull-provisioning/import-device-column-type.png)<br/>
**Примечание:** такие типы столбцов, как «имя», «тип» и «токен доступа», могут быть выбраны только для одной строки. 

#### Импорт завершен

После завершения процесса создания будет показана статистическая информация.
В следующем примере мы видим, что 8 устройств были созданы успешно, и при создании 2 устройств произошла ошибка. Причина заключается в том, что в данном тестовом файле устройства 1, 2 и 3 имеют один и тот же токен. Система IoT Ростелеком запрещает это.

![image](/images/user-guide/bull-provisioning/import-device-info-created.png)<br/>
 
