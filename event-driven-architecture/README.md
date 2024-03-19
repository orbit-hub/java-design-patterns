---
title: 事件驱动架构
category: Architectural
language: en
tag:
 - Reactive
---

## Intent
使用事件驱动的体系结构向其他应用程序发送和通知对象的状态更改。
## Class diagram
![alt text](./etc/eda.png "Event Driven Architecture")

## Applicability
在以下情况下使用事件驱动架构

* 你想创建一个松散耦合的系统
* 你想建立一个反应更灵敏的系统
* 你想要一个更容易扩展的系统

## Real world examples

* 收费API Chargify通过各种事件公开支付活动(https://docs.chargify.com/api-events)
* Amazon的AWS Lambda，允许您执行响应事件的代码，例如更改Amazon S3桶，更新Amazon DynamoDB表，或由应用程序或设备生成的自定义事件。(https://aws.amazon.com/lambda)
* MySQL运行基于事件的触发器，如数据库表上发生的插入和更新事件。
## Credits

* [Event-driven architecture - Wikipedia](https://en.wikipedia.org/wiki/Event-driven_architecture)
* [What is an Event-Driven Architecture](https://aws.amazon.com/event-driven-architecture/)
* [Real World Applications/Event Driven Applications](https://wiki.haskell.org/Real_World_Applications/Event_Driven_Applications)
* [Event-driven architecture definition](http://searchsoa.techtarget.com/definition/event-driven-architecture)
