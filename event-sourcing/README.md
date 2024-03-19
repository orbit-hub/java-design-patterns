---
title: Event Sourcing
category: Architectural
language: en
tag:
  - Performance
  - Cloud distributed
---

## Intent
不只是在域中存储数据的当前状态，而是使用仅追加存储来记录对该数据所采取的全部操作系列。存储充当记录系统，
可用于实体化域对象。通过避免同步数据模型和业务域，这可以简化复杂域中的任务，同时提高性能、可伸缩性和响应性。
它还可以为事务性数据提供一致性，并维护完整的审计跟踪和历史记录，从而支持补偿操作。
## Class diagram
![alt text](./etc/event-sourcing.png "Event Sourcing")

## Applicability
在以下情况下使用事件来源模式

*即使应用程序状态具有复杂的关系数据结构，您也需要非常高性能的持久化应用程序状态
*您需要日志更改您的应用程序状态和能力，以恢复任何时刻的状态。
*您需要通过重放过去的事件来调试生产问题。

## Real world examples

* [The Lmax Architecture](https://martinfowler.com/articles/lmax.html)

## Credits

* [Martin Fowler - Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)
* [Event Sourcing in Microsoft's documentation](https://docs.microsoft.com/en-us/azure/architecture/patterns/event-sourcing)
* [Reference 3: Introducing Event Sourcing](https://msdn.microsoft.com/en-us/library/jj591559.aspx)
* [Event Sourcing pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/event-sourcing)
