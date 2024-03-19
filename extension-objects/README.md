---
title: Extension objects
category: Behavioral
language: en
tag:
 - Extensibility
---

# Extention Objects Pattern

## Intent
预期对象的接口将来需要扩展。额外的接口由扩展对象定义。

## Explanation
Real-world example

假设你正在为客户端开发一款基于java的游戏，在开发过程中，有人提出了一些新功能。
扩展对象模式使您的程序能够以最小的重构来适应不可预见的变化，特别是在将附加功能集成到项目中时。
In plain words

> 扩展对象模式用于在不修改对象的核心类的情况下动态地向对象添加功能。它是一种行为设计模式，
> 用于向程序中的现有类和对象添加新功能。
> 这种模式为程序员提供了扩展/修改类功能的能力，而不必重构现有的源代码。
Wikipedia says

> In object-oriented computer programming, an extension objects pattern is a design pattern added to an object after the original object was compiled. The modified object is often a class, a prototype or a type. Extension object patterns are features of some object-oriented programming languages. There is no syntactic difference between calling an extension method and calling a method declared in the type definition.

**Programmatic example**

利用扩展对象模式的目的是实现新的特性/功能，而不必重构每个类。
下面的例子展示了在游戏中的敌人类扩展实体中使用这种模式:
Primary App class to execute our program from.
```java
public class App {
    public static void main(String[] args) {
        Entity enemy = new Enemy("Enemy");
        checkExtensionsForEntity(enemy);
    }

    private static void checkExtensionsForEntity(Entity entity) {
        Logger logger = Logger.getLogger(App.class.getName());
        String name = entity.getName();
        Function<String, Runnable> func = (e) -> () -> logger.info(name + " without " + e);

        String extension = "EnemyExtension";
        Optional.ofNullable(entity.getEntityExtension(extension))
                .map(e -> (EnemyExtension) e)
                .ifPresentOrElse(EnemyExtension::extendedAction, func.apply(extension));
    }
}
```
Enemy class with initial actions and extensions.
```java
class Enemy extends Entity {
    public Enemy(String name) {
        super(name);
    }

    @Override
    protected void performInitialAction() {
        super.performInitialAction();
        System.out.println("Enemy wants to attack you.");
    }

    @Override
    public EntityExtension getEntityExtension(String extensionName) {
        if (extensionName.equals("EnemyExtension")) {
            return Optional.ofNullable(entityExtension).orElseGet(EnemyExtension::new);
        }
        return super.getEntityExtension(extensionName);
    }
}
```
EnemyExtension class with overriding extendAction() method.
```java
class EnemyExtension implements EntityExtension {
    @Override
    public void extendedAction() {
        System.out.println("Enemy has advanced towards you!");
    }
}
```
Entity class which will be extended by Enemy.
```java
class Entity {
    private String name;
    protected EntityExtension entityExtension;

    public Entity(String name) {
        this.name = name;
        performInitialAction();
    }

    protected void performInitialAction() {
        System.out.println(name + " performs the initial action.");
    }

    public EntityExtension getEntityExtension(String extensionName) {
        return null;
    }

    public String getName() {
        return name;
    }
}
```
EntityExtension interface to be used by EnemyExtension.
```java
interface EntityExtension {
    void extendedAction();
}
```
Program output:
```markdown
Enemy performs the initial action.
Enemy wants to attack you.
Enemy has advanced towards you!
```
In this example, the Extension Objects pattern allows the enemy entity to perform unique initial actions and advanced actions when specific extensions are applied. This pattern provides flexibility and extensibility to the codebase while minimizing the need for major code changes.


## Class diagram
![Extension_objects](./etc/extension_obj.png "Extension objects")

## Applicability
Use the Extension Objects pattern when:

* you need to support the addition of new or unforeseen interfaces to existing classes and you don't want to impact clients that don't need this new interface. Extension Objects lets you keep related operations together by defining them in a separate class
* a class representing a key abstraction plays different roles for different clients. The number of roles the class can play should be open-ended. There is a need to preserve the key abstraction itself. For example, a customer object is still a customer object even if different subsystems view it differently.
* a class should be extensible with new behavior without subclassing from it.

## Real world examples

* [OpenDoc](https://en.wikipedia.org/wiki/OpenDoc)
* [Object Linking and Embedding](https://en.wikipedia.org/wiki/Object_Linking_and_Embedding)
