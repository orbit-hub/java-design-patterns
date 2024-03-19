---
title: Anti-corruption layer
category: Architectural
language: en
tag:
  - Cloud distributed
  - Decoupling
  - Microservices
---

## Intent

在不共享相同语义的不同子系统之间实现一个farade或适配器层。这一层将一个子系统发出的请求转换为另一个子系统。
使用此模式可确保应用程序的设计不受对外部子系统的依赖关系的限制。这种模式最早是由Eric Evans在领域驱动设计中描述的。

## Explanation

### Context and problem
大多数应用程序依赖于其他系统提供某些数据或功能。
例如，当遗留应用程序迁移到现代系统时，
它可能仍然需要现有的遗留资源。
新功能必须能够调用遗留系统。
对于逐渐迁移来说尤其如此，
随着时间的推移，大型应用程序的不同功能被转移到现代系统中。

通常，这些遗留系统存在质量问题，例如复杂的数据模式或过时的api。
遗留系统中使用的特性和技术可能与更现代的系统有很大不同。
为了与遗留系统进行互操作，新应用程序可能需要支持过时的基础设施、协议、数据模型、api、
或者其他您不会在现代应用程序中添加的功能。

维护新系统和遗留系统之间的访问可以迫使新系统至少遵守一些遗留系统的api或其他语义。
当这些遗留特性存在质量问题时，支持它们会“破坏”原本设计干净的现代应用程序。
类似的问题可能出现在开发团队无法控制的任何外部系统上，而不仅仅是遗留系统。

### Solution
Isolate the different subsystems by placing an anti-corruption layer between them. 
This layer translates communications between the two systems, allowing one system to remain unchanged 
while the other can avoid compromising its design and technological approach.

### Programmatic example
#### Introduction
The example shows why the anti-corruption layer is needed.
Here are 2 shop-ordering systems: `Legacy` and `Modern`. \
(
*It is important to state the separation is very conditional, and is drawn for learning purposes*. 
*In reality the pattern does not depend on the so-called ageness but rather relies on the different domain models.*)

The aforementioned systems have different domain models and have to operate simultaneously.
Since they work independently the orders can come either from the `Legacy` or `Modern` system.
Therefore, the system that receives the legacyOrder needs to check if the legacyOrder is valid and not present in the other system.
Then it can place the legacyOrder in its own system.

But for that, the system needs to know the domain model of the other system and to avoid that, 
the anti-corruption layer(ACL) is introduced. 
The ACL is a layer that translates the domain model of the `Legacy` system to the domain model of the `Modern` system and vice versa.
Also, it hides all other operations with the other system, uncoupling the systems.

#### Domain model of the `Legacy` system
```java
public class LegacyOrder {
    private String id;
    private String customer;
    private String item;
    private String qty;
    private String price;
}
```
#### Domain model of the `Modern` system
```java
public class ModernOrder {
    private String id;
    private Customer customer;

    private Shipment shipment;

    private String extra;
}
public class Customer {
    private String address;
}
public class Shipment {
    private String item;
    private String qty;
    private String price;
}
```
#### Anti-corruption layer
```java
public class AntiCorruptionLayer {

    @Autowired
    private ModernShop modernShop;

    @Autowired
    private LegacyShop legacyShop;

    public Optional<LegacyOrder> findOrderInModernSystem(String id) {
        return modernShop.findOrder(id).map(o -> /* map to legacyOrder*/);
    }

    public Optional<ModernOrder> findOrderInLegacySystem(String id) {
        return legacyShop.findOrder(id).map(o -> /* map to modernOrder*/);
    }

}
```
#### The connection
Wherever the `Legacy` or `Modern` system needs to communicate with the counterpart 
the ACL needs to be used to avoid corrupting the current domain model.
The example below shows how the `Legacy` system places an order with a validation from the `Modern` system.
```java
public class LegacyShop {
    @Autowired
    private AntiCorruptionLayer acl;

    public void placeOrder(LegacyOrder legacyOrder) throws ShopException {

        String id = legacyOrder.getId();

        Optional<LegacyOrder> orderInModernSystem = acl.findOrderInModernSystem(id);

        if (orderInModernSystem.isPresent()) {
            // order is already in the modern system
        } else {
            // place order in the current system
        }
    }
}
```

### Issues and considerations
 - The anti-corruption layer may add latency to calls made between the two systems.
 - The anti-corruption layer adds an additional service that must be managed and maintained.
 - Consider how your anti-corruption layer will scale.
 - Consider whether you need more than one anti-corruption layer. You may want to decompose functionality into multiple services using different technologies or languages, or there may be other reasons to partition the anti-corruption layer.
 - Consider how the anti-corruption layer will be managed in relation with your other applications or services. How will it be integrated into your monitoring, release, and configuration processes?
 - Make sure transaction and data consistency are maintained and can be monitored.
 - Consider whether the anti-corruption layer needs to handle all communication between different subsystems, or just a subset of features.
 - If the anti-corruption layer is part of an application migration strategy, consider whether it will be permanent, or will be retired after all legacy functionality has been migrated.
 - This pattern is illustrated with distinct subsystems above, but can apply to other service architectures as well, such as when integrating legacy code together in a monolithic architecture.

## Applicability

Use this pattern when:

 - A migration is planned to happen over multiple stages, but integration between new and legacy systems needs to be maintained.
 - Two or more subsystems have different semantics, but still need to communicate.

This pattern may not be suitable if there are no significant semantic differences between new and legacy systems.

## Tutorials

* [Microsoft - Anti-Corruption Layer](https://learn.microsoft.com/en-us/azure/architecture/patterns/anti-corruption-layer)
* [Amazon - Anti-Corruption Layer](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/acl.html)

## Credits

* [Domain-Driven Design. Eric Evans](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215)