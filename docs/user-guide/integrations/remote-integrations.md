---
layout: docwithnav
assignees:
- ashvayka
title: Удаленные интеграции
description: Удаленные интеграции
---

* TOC
{:toc}

## Введение

Любую интеграцию IoT платформы Ростелеком можно выполнить удаленно из основной версии платформы. Это руководство содержит пошаговые инструкции по удаленному запуску интеграции платформы. В качестве примера мы запустим интеграцию MQTT, которая подключается к локальному брокеру MQTT и отправляет данные в облако. 

## Шаги для настройки платформы

### Step 1. Создание дефолтных Uplink и Downlink конвертеров

Создайте шаблоны конвертеров по восходящей и нисходящей линий связи и настройте их для работы в режиме отладки. 
Во время работы в данном отладки эти конвертеры будут записывать все входящие события, что поможет настроить преобразователи, как только вы начнете получать данные.

![image](/images/user-guide/integrations/remote/default-converters.gif)  

### Шаг 2. Создание удаленной интеграции

Создайте удаленную интеграцию, которая будет подключаться к локальному брокеру через порт 1883 и подписываться на все темы. Обратите внимание, что нужно выставить флаги “Отладка” и “Выполнить удаленно”.  

![image](/images/user-guide/integrations/remote/mqtt-integration.gif)

### Шаг 3. Сохранение учетных данных удаленной интеграции.

Скопируйте и вставьте ключ интеграции и пароль из раздела интеграции.

![image](/images/user-guide/integrations/remote/copy-integration-credentials.gif)

## Шаги установки удаленной интеграции

### Выберите свою платформу и установите

Можно установить интеграции платформы через Doker, Debian и rpm пакеты. Используйте один из следующих шагов

 * [Docker на Linux или Mac OS](/docs/user-guide/integrations/remote-integrations/#docker-on-linuxmac)
 * [Docker на Windows](/docs/user-guide/integrations/remote-integrations/#docker-on-windows) 
 * [Ubuntu](/docs/user-guide/integrations/remote-integrations/#ubuntu-server)
 * [CentOS/RHEL Server](/docs/user-guide/integrations/remote-integrations/#centosrhel-server)

### Docker на Linux/Mac

- **[Install Docker CE](https://docs.docker.com/engine/installation/)**

- **Выберите интеграцию для установки**


{% capture contenttogglespec %}
HTTP Integrations<br/><small>(HTTP, Sigfox, ThingPark, OceanConnect and <br/> T-Mobile IoT CDP)</small>%,%http%,%templates/install/integration/http-docker.md%br%
MQTT Integrations<br/><small>(MQTT, AWS IoT, IBM Watson, The Things Network)</small>%,%mqtt%,%templates/install/integration/mqtt-docker.md%br%
AWS SQS<br/> Integration<br/>%,%aws%,%templates/install/integration/aws-docker.md%br%
Azure Event Hub<br/>Integration<br/>%,%azure%,%templates/install/integration/azure-docker.md%br%
OPC UA<br/> Integration<br/>%,%opcua%,%templates/install/integration/opcua-docker.md%br%
TCP/UDP<br/> Integration<br/>%,%tcpudp%,%templates/install/integration/tcpudp-docker.md{% endcapture %}

{% include content-toggle.html content-toggle-id="remoteintegrationdockerinstall" toggle-spec=contenttogglespec %}


{% include templates/install/integration/advanced-config-docker.md %} 


- **Диагностика**


**Примечание** если вы столкнулись с ошибками, связанными с проблемами DNS, например

```bash
127.0.1.1:53: cannot unmarshal DNS message
```
{: .copy-code}

Вы можете настроить свою систему на использование общедоступных DNS-серверов Google. См.соответствующие инструкции для [Linux](opc-uas://developers.google.com/speed/public-dns/docs/using#linux) и [Mac OS](opc-uas://developers.google.com/speed/public-dns/docs/using#mac_os)

### Docker на Windows

- **[Установка Docker Toolbox for Windows](https://docs.docker.com/toolbox/toolbox_install_windows/)**

- **Выберите интеграцию для установки**


{% capture contenttogglespecwin %}
HTTP Integrations<br/><small>(HTTP, Sigfox, ThingPark, OceanConnect and <br/> T-Mobile IoT CDP)</small>%,%http%,%templates/install/integration/http-docker-windows.md%br%
MQTT Integrations<br/><small>(MQTT, AWS IoT, IBM Watson, The Things Network)</small>%,%mqtt%,%templates/install/integration/mqtt-docker-windows.md%br%
AWS SQS<br/> Integration<br/>%,%aws%,%templates/install/integration/aws-docker-windows.md%br%
Azure Event Hub<br/>Integration<br/>%,%azure%,%templates/install/integration/azure-docker-windows.md%br%
OPC UA<br/> Integration<br/>%,%opcua%,%templates/install/integration/opcua-docker-windows.md%br%
TCP/UDP<br/> Integration<br/>%,%tcpudp%,%templates/install/integration/tcpudp-docker-windows.md{% endcapture %}

{% include content-toggle.html content-toggle-id="remoteintegrationdockerinstallwin" toggle-spec=contenttogglespecwin %}


{% include templates/install/integration/advanced-config-docker.md %} 


- **Диагностика**


**Примечание** Примечание: если вы столкнулись с ошибками, связанными с проблемами DNS, например

```bash
127.0.1.1:53: cannot unmarshal DNS message
```

Вы можете настроить свою систему на использование общедоступных DNS-серверов  [Google](https://developers.google.com/speed/public-dns/docs/using#windows)


### сервер Ubuntu

 - Установка Java 8 (OpenJDK) 

{% include templates/install/ubuntu-java-install.md %}

 - **Выберите пакет интеграции для установки**
 
{% capture ubuntuinstallspec %}
HTTP Integrations<br/><small>(HTTP, Sigfox, ThingPark, OceanConnect and <br/> T-Mobile IoT CDP)</small>%,%http%,%templates/install/integration/http-ubuntu.md%br%
MQTT Integrations<br/><small>(MQTT, AWS IoT, IBM Watson, The Things Network)</small>%,%mqtt%,%templates/install/integration/mqtt-ubuntu.md%br%
AWS SQS<br/> Integration<br/>%,%aws%,%templates/install/integration/aws-ubuntu.md%br%
Azure Event Hub<br/>Integration<br/>%,%azure%,%templates/install/integration/azure-ubuntu.md%br%
OPC UA<br/> Integration<br/>%,%opcua%,%templates/install/integration/opcua-ubuntu.md%br%
TCP/UDP<br/> Integration<br/>%,%tcpudp%,%templates/install/integration/tcpudp-ubuntu.md{% endcapture %}

{% include content-toggle.html content-toggle-id="remoteintegrationinstallubuntu" toggle-spec=ubuntuinstallspec %} 

### CentOS/RHEL Server

 - Установите Java 8 (OpenJDK)

{% include templates/install/rhel-java-install.md %}

 - **•	Выберите интеграцию для установки**
 
{% capture rhelinstallspec %}
HTTP Integrations<br/><small>(HTTP, Sigfox, ThingPark, OceanConnect and <br/> T-Mobile IoT CDP)</small>%,%http%,%templates/install/integration/http-rhel.md%br%
MQTT Integrations<br/><small>(MQTT, AWS IoT, IBM Watson, The Things Network)</small>%,%mqtt%,%templates/install/integration/mqtt-rhel.md%br%
AWS SQS<br/> Integration<br/>%,%aws%,%templates/install/integration/aws-rhel.md%br%
Azure Event Hub<br/>Integration<br/>%,%azure%,%templates/install/integration/azure-rhel.md%br%
OPC UA<br/> Integration<br/>%,%opcua%,%templates/install/integration/opcua-rhel.md%br%
TCP/UDP<br/> Integration<br/>%,%tcpudp%,%templates/install/integration/tcpudp-rhel.md{% endcapture %}

{% include content-toggle.html content-toggle-id="remoteintegrationinstallrhel" toggle-spec=rhelinstallspec %} 

## Удаленная настройка интеграции

Настройка удаленной интеграции выполняется с помощью пользовательского интерфейса платформы, для чего не предусмотрена пошаговая инструкция. Изучите руководства и видеоуроки, связанные с конкретными интеграциями:

 - [HTTP](/docs/user-guide/integrations/http/)
 - [MQTT](/docs/user-guide/integrations/mqtt/)
 - [AWS IoT](/docs/user-guide/integrations/aws-iot/)
 - [IBM Watson IoT](/docs/user-guide/integrations/ibm-watson-iot/)
 - [Azure Event Hub](/docs/user-guide/integrations/azure-event-hub/)
 - [Actility ThingPark](/docs/user-guide/integrations/thingpark/)
 - [SigFox](/docs/user-guide/integrations/sigfox/)
 - [OceanConnect](/docs/user-guide/integrations/ocean-connect/)
 - [TheThingsNetwork](/docs/user-guide/integrations/ttn/)
 - [OPC-UA](/docs/user-guide/integrations/opc-ua/)
 - [TCP](/docs/user-guide/integrations/tcp/)
 - [UDP](/docs/user-guide/integrations/udp/)
 - [Custom](/docs/user-guide/integrations/custom/)

  
## Устранение неполадок при удаленной интеграции

Посмотрите файлы журнала. Их расположение зависит от используемой платформы и установочного пакета и упоминается в шагах установки.
 
