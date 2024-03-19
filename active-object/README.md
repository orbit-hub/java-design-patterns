---
title: Active Object
category: Concurrency
language: en
tag:
 - Performance
---


## Intent

活动对象设计模式为在并发系统中实现异步行为提供了一种安全可靠的方式。
它通过将任务封装在具有自己的线程和消息队列的对象中来实现这一点。
这种分离使主线程保持响应性，并避免了诸如直接线程操作或共享状态访问之类的问题。
## Explanation

实现活动对象模式的类将包含不使用“synchronized”方法的自同步机制。
Real-world example

>兽人以其野性和不可驯服的灵魂而闻名。似乎他们有自己的控制线程基于之前的行为。
> 
要实现一个具有自己的线程控制机制的生物，并且只公开其API而不公开其执行本身，我们可以使用活动对象模式。

**Programmatic Example**

```java
public abstract class ActiveCreature{
  private final Logger logger = LoggerFactory.getLogger(ActiveCreature.class.getName());

  private BlockingQueue<Runnable> requests;
  
  private String name;
  
  private Thread thread;

  public ActiveCreature(String name) {
    this.name = name;
    this.requests = new LinkedBlockingQueue<Runnable>();
    thread = new Thread(new Runnable() {
        @Override
        public void run() {
          while (true) {
            try {
              requests.take().run();
            } catch (InterruptedException e) { 
              logger.error(e.getMessage());
            }
          }
        }
      }
    );
    thread.start();
  }
  
  public void eat() throws InterruptedException {
    requests.put(new Runnable() {
        @Override
        public void run() { 
          logger.info("{} is eating!",name());
          logger.info("{} has finished eating!",name());
        }
      }
    );
  }

  public void roam() throws InterruptedException {
    requests.put(new Runnable() {
        @Override
        public void run() { 
          logger.info("{} has started to roam the wastelands.",name());
        }
      }
    );
  }
  
  public String name() {
    return this.name;
  }
}
```

We can see that any class that will extend the ActiveCreature class will have its own thread of control to invoke and execute methods.

For example, the Orc class:

```java
public class Orc extends ActiveCreature {

  public Orc(String name) {
    super(name);
  }

}
```

Now, we can create multiple creatures such as Orcs, tell them to eat and roam, and they will execute it on their own thread of control:

```java
  public static void main(String[] args) {  
    var app = new App();
    app.run();
  }
  
  @Override
  public void run() {
    ActiveCreature creature;
    try {
      for (int i = 0;i < creatures;i++) {
        creature = new Orc(Orc.class.getSimpleName().toString() + i);
        creature.eat();
        creature.roam();
      }
      Thread.sleep(1000);
    } catch (InterruptedException e) {
      logger.error(e.getMessage());
    }
    Runtime.getRuntime().exit(1);
  }
```

## Class diagram

![alt text](./etc/active-object.urm.png "Active Object class diagram")

## Applicability

* When you need to perform long-running operations without blocking the main thread.
* When you need to interact with external resources asynchronously.
* When you want to improve the responsiveness of your application.
* When you need to manage concurrent tasks in a modular and maintainable way.

## Tutorials

* [Android and Java Concurrency: The Active Object Pattern](https://www.youtube.com/watch?v=Cd8t2u5Qmvc)

## Consequences

Benefits

* Improves responsiveness of the main thread.
* Encapsulates concurrency concerns within objects.
* Promotes better code organization and maintainability.
* Provides thread safety and avoids shared state access problems.

Trade-offs

* Introduces additional overhead due to message passing and thread management.
* May not be suitable for all types of concurrency problems.

## Related patterns

* Observer
* Reactor
* Producer-consumer
* Thread pool

## Credits

* [Design Patterns: Elements of Reusable Object Software](https://amzn.to/3HYqrBE)
* [Concurrent Programming in Java: Design Principles and Patterns](https://amzn.to/498SRVq)
* [Learning Concurrent Programming in Scala](https://amzn.to/3UE07nV)
* [Pattern Languages of Program Design 3](https://amzn.to/3OI1j61)
