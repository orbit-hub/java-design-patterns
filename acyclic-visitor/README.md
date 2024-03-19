---
title: Acyclic Visitor
category: Behavioral
language: en
tag:
  - Decoupling
  - Extensibility
---

## Intent

无循环访问者模式将操作与对象层次结构解耦，
允许您在不直接修改对象结构的情况下添加新操作。

## Explanation

Real world example

> 我们有现代阶级的等级制度。在这个层次结构中的调制解调器需要访问的外部算法基于
> 过滤条件(是Unix或DOS兼容的调制解调器)。

In plain words

> 无循环访问器允许将函数添加到现有的类层次结构中，而无需修改层次结构。
[WikiWikiWeb](https://wiki.c2.com/?AcyclicVisitor) says

> The Acyclic Visitor pattern allows new functions to be added to existing class hierarchies without affecting those 
> hierarchies, and without creating the dependency cycles that are inherent to the GangOfFour VisitorPattern.

**Programmatic Example**

Here's the `Modem` hierarchy.

```java
public abstract class Modem {
  public abstract void accept(ModemVisitor modemVisitor);
}

public class Zoom extends Modem {
  ...
  @Override
  public void accept(ModemVisitor modemVisitor) {
    if (modemVisitor instanceof ZoomVisitor) {
      ((ZoomVisitor) modemVisitor).visit(this);
    } else {
      LOGGER.info("Only ZoomVisitor is allowed to visit Zoom modem");
    }
  }
}

public class Hayes extends Modem {
  ...
  @Override
  public void accept(ModemVisitor modemVisitor) {
    if (modemVisitor instanceof HayesVisitor) {
      ((HayesVisitor) modemVisitor).visit(this);
    } else {
      LOGGER.info("Only HayesVisitor is allowed to visit Hayes modem");
    }
  }
}
```

Next we introduce the `ModemVisitor` hierarchy.

```java
public interface ModemVisitor {
}

public interface HayesVisitor extends ModemVisitor {
  void visit(Hayes hayes);
}

public interface ZoomVisitor extends ModemVisitor {
  void visit(Zoom zoom);
}

public interface AllModemVisitor extends ZoomVisitor, HayesVisitor {
}

public class ConfigureForDosVisitor implements AllModemVisitor {
  ...
  @Override
  public void visit(Hayes hayes) {
    LOGGER.info(hayes + " used with Dos configurator.");
  }
  @Override
  public void visit(Zoom zoom) {
    LOGGER.info(zoom + " used with Dos configurator.");
  }
}

public class ConfigureForUnixVisitor implements ZoomVisitor {
  ...
  @Override
  public void visit(Zoom zoom) {
    LOGGER.info(zoom + " used with Unix configurator.");
  }
}
```

Finally, here are the visitors in action.

```java
    var conUnix = new ConfigureForUnixVisitor();
    var conDos = new ConfigureForDosVisitor();
    var zoom = new Zoom();
    var hayes = new Hayes();
    hayes.accept(conDos);
    zoom.accept(conDos);
    hayes.accept(conUnix);
    zoom.accept(conUnix);   
```

Program output:

```
    // Hayes modem used with Dos configurator.
    // Zoom modem used with Dos configurator.
    // Only HayesVisitor is allowed to visit Hayes modem
    // Zoom modem used with Unix configurator.
```

## Class diagram

![alt text](./etc/acyclic-visitor.png "Acyclic Visitor")

## Applicability

This pattern can be used:

* When you need to add a new function to an existing hierarchy without the need to alter or affect that hierarchy.
* When there are functions that operate upon a hierarchy, but which do not belong in the hierarchy itself. e.g. the ConfigureForDOS / ConfigureForUnix / ConfigureForX issue.
* When you need to perform very different operations on an object depending upon its type.
* When the visited class hierarchy will be frequently extended with new derivatives of the Element class.
* When the recompilation, relinking, retesting or redistribution of the derivatives of Element is very expensive.

## Tutorials

* [Acyclic Visitor Pattern Example](https://codecrafter.blogspot.com/2012/12/the-acyclic-visitor-pattern.html)

## Consequences

Benefits:

* No dependency cycles between class hierarchies.
* No need to recompile all the visitors if a new one is added.
* Does not cause compilation failure in existing visitors if class hierarchy has a new member.

Trade-offs:

* Violates [Liskov's Substitution Principle](https://java-design-patterns.com/principles/#liskov-substitution-principle) by showing that it can accept all visitors but actually only being interested in particular visitors.
* Parallel hierarchy of visitors has to be created for all members in visitable class hierarchy.

## Related patterns

* [Visitor Pattern](https://java-design-patterns.com/patterns/visitor/)

## Credits

* [Acyclic Visitor by Robert C. Martin](http://condor.depaul.edu/dmumaugh/OOT/Design-Principles/acv.pdf)
* [Acyclic Visitor in WikiWikiWeb](https://wiki.c2.com/?AcyclicVisitor)
* [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.amazon.com/gp/product/0201633612/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0201633612&linkCode=as2&tag=javadesignpat-20&linkId=675d49790ce11db99d90bde47f1aeb59)
* [Head First Design Patterns: A Brain-Friendly Guide](https://www.amazon.com/gp/product/0596007124/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0596007124&linkCode=as2&tag=javadesignpat-20&linkId=6b8b6eea86021af6c8e3cd3fc382cb5b)
