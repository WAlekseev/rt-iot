---
layout: docwithnav
title: Тенанты
---

* TOC
{:toc}

IoT платформа Ростелеком поддерживается [мультитенантность](https://en.wikipedia.org/wiki/Multitenancy) "из коробки".
Вы можете рассматривать тенанта платформы как отдельную бизнес-сущность: физическое лицо или организацию, которая владеет устройствами.

**Администратор платформы** может создавать сущностей с типом тенант.

![image](/images/user-guide/ui/tenants.png)

Администратор платформы также может создавать множество [пользователей](/docs/user-guide/ui/users) с ролью **Тенант-администратор** для каждого тенанта. Для этого нужно нажать кнопку "Управление тенант-админами" в деталях тенанта.
 
![image](/images/user-guide/ui/manage-tenant-admins.png) 
 
Тенант-администратор может выполнять следующие действия:
 
 - Обеспечение [девайсов](/docs/user-guide/ui/devices) и управление ими.
 - Обеспечение [активов](/docs/user-guide/ui/assets) и управление ими.
 - Создание [клиентов](/docs/user-guide/ui/customers) и управление ими.
 - Создание [дашбордов](/docs/user-guide/ui/dashboards) и управление ими.
 - Настройка [движка правил](/docs/user-guide/rule-engine-2-0/re-getting-started/)
 - Добавление и изменение дефолтных виджетов с помощью [библиотеки виджетов](/docs/user-guide/ui/widget-library).
 
 Все вышеперечисленные действия доступны через [REST API](/docs/reference/rest-api/)
