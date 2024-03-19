---
title: Event Queue
category: Concurrency
language: en
tag:
 - Game programming
---

## Intent
事件队列设计模式(也称为消息队列)的目的是将事件队列之间的关系解耦
系统内事件的发送者和接收者。通过解耦双方，它们不与事件队列交互
同时进行。从本质上讲，事件队列以异步方式处理和处理请求
系统可以被描述为一个先进先出的设计模式模型。事件队列是一个合适的模式，如果存在
可访问性有限的资源(即音频或数据库)，但是，您需要提供对所有请求的访问
它寻找这个资源。在从队列中访问事件时，程序也将其从队列中删除。
![alt text](./etc/event-queue-model.png "Event Queue Visualised")

## Explanation 

Real world example

> 现代电子邮件系统是事件队列设计模式背后的基本过程的一个例子。当一封电子邮件
> 发送后，发送方继续他们的日常工作，而不需要接收方立即响应。
> 此外，收信人可以在闲暇时自由访问和处理电子邮件。因此，这个过程
> 将发送方和接收方解耦，这样它们就不需要同时与队列交互。

In plain words

>发送方和接收方之间的缓冲区提高了系统的可维护性和可扩展性。事件队列通常是用于组织和执行进程间通信(IPC)。

Wikipedia says

> 消息队列(也称为事件队列)实现两个或多个进程之间的异步通信模式
> 线程，即发送方和接收方不需要同时与队列交互。

Key drawback

> 由于事件队列模型解耦了发送方-接收方关系，这意味着事件队列设计模式是
> 不适用于发送方需要响应的场景。例如，这是在线中的一个突出功能
> 因此，在多人游戏中，这种方法需要深思熟虑。

**Programmatic Example**

Upon examining our event-queue example, here's the app which utilised an event queue system.

```java
import javax.sound.sampled.UnsupportedAudioFileException;
import java.io.IOException;

public class App {

    /**
     * Program entry point.
     *
     * @param args command line args
     * @throws IOException                   when there is a problem with the audio file loading
     * @throws UnsupportedAudioFileException when the loaded audio file is unsupported
     */
    public static void main(String[] args) throws UnsupportedAudioFileException, IOException,
            InterruptedException {
        var audio = Audio.getInstance();
        audio.playSound(audio.getAudioStream("./etc/Bass-Drum-1.wav"), -10.0f);
        audio.playSound(audio.getAudioStream("./etc/Closed-Hi-Hat-1.wav"), -8.0f);

        LOGGER.info("Press Enter key to stop the program...");
        try (var br = new BufferedReader(new InputStreamReader(System.in))) {
            br.read();
        }
        audio.stopService();
    }
}
```

Much of the design pattern is developed within the Audio class. Here we set instances, declare global variables and establish 
the key methods used in the above runnable class.

```java
public class Audio {
    private static final Audio INSTANCE = new Audio();

    private static final int MAX_PENDING = 16;

    private int headIndex;

    private int tailIndex;

    private volatile Thread updateThread = null;

    private final PlayMessage[] pendingAudio = new PlayMessage[MAX_PENDING];

    // Visible only for testing purposes
    Audio() {

    }

    public static Audio getInstance() {
        return INSTANCE;
    }
}
```

The Audio class is also responsible for handling and setting the states of the thread, this is shown in the code segments
below.

```java
/**
 * This method stops the Update Method's thread and waits till service stops.
 */
public synchronized void stopService() throws InterruptedException {
    if (updateThread != null) {updateThread.interrupt();}
    updateThread.join();
    updateThread = null;
}

/**
 * This method check the Update Method's thread is started.
 * @return boolean
 */
public synchronized boolean isServiceRunning() {
    return updateThread != null && updateThread.isAlive();}

/**
 * Starts the thread for the Update Method pattern if it was not started previously. Also when the
 * thread is ready it initializes the indexes of the queue
 */
public void init() {
    if (updateThread == null) {
        updateThread = new Thread(() -> {
            while (!Thread.currentThread().isInterrupted()) {
                update();
            }});}
    startThread();
}

/**
 * This is a synchronized thread starter.
 */
private synchronized void startThread() {
    if (!updateThread.isAlive()) {
        updateThread.start();
        headIndex = 0;
        tailIndex = 0;
    }
}
```

New audio is added into our event queue in the playSound method found in the Audio class. The update method is then utilised
to retrieve an audio item from the queue and play it to the user.

```java
public void playSound(AudioInputStream stream, float volume) {
    init();
    // Walk the pending requests.
    for (var i = headIndex; i != tailIndex; i = (i + 1) % MAX_PENDING) {
      var playMessage = getPendingAudio()[i];
      if (playMessage.getStream() == stream) {
        // Use the larger of the two volumes.
        playMessage.setVolume(Math.max(volume, playMessage.getVolume()));
        // Don't need to enqueue.
        return;
      }
    }
    getPendingAudio()[tailIndex] = new PlayMessage(stream, volume);
    tailIndex = (tailIndex + 1) % MAX_PENDING;
}
```

Within the Audio class are some more methods with assist the construction of the event-queue design patterns, they are 
summarised below.

- getAudioStream() = returns the input stream path of a file
- getPendingAudio() = returns the current event queue item 


## Class diagram
![alt text](./etc/model.png "Event Queue")

## Applicability

Use the Event Queue Pattern when

* The sender does not require a response from the receiver.
* You wish to decouple the sender & the receiver.
* You want to process events asynchronously.
* You have a limited accessibility resource and the asynchronous process is acceptable to reach that.

## Credits

* [Mihaly Kuprivecz - Event Queue] (http://gameprogrammingpatterns.com/event-queue.html)
* [Wikipedia - Message Queue] (https://en.wikipedia.org/wiki/Message_queue)
* [AWS - Message Queues] (https://aws.amazon.com/message-queue/)
