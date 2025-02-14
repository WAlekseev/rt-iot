---
layout: docwithnav
title: Архитектура движка правил
description: Архитектура движка правил

---

* TOC
{:toc}

Движок правил Ростелеком основан на двух основных компонентах: модели актора и очереди сообщений.

<br/>

![image](/images/user-guide/rule-engine-2-0/rule-engine-architecture.png)
 
### Модель актора

Модель актора обеспечивает высокую производительность и параллельную обработку сообщений с транспортного уровня устройства до тех пор, пока не будут выполнены API-вызовы на стороне сервера. IoT Ростелеком использует собственное воплощение акторской системы, которая заточена специально под наш случай использования. Существует два основных актора, связанных с Rule Engine: актор Rule Chain и актор Rule node.

#### Актор Rule Chain

Актор Rule Chain отвечает за конфигурацию узлов правил, маршрутизацию сообщений между ними и обработку put and ack-команд  очереди. Каждый актор Rule Chain представляет собой единую цепочку правил, настроенную пользователем. Актор Rule Chain является родительским для нескольких акторов Rule Node.

#### Актор Rule Node

<![endif]--> Актор Rule Node отвечает за обработку входящих сообщений. Логика обработки сообщений легко настраивается. Существует множество встроенных реализаций Rule Nodes, и вы также можете разработать свои собственные, пользовательские их реализации.
Дополнительные сведения смотрите в руководстве
[Rule Node Development](/docs/user-guide/contribution/rule-node-development/).
