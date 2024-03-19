---
title: Execute Around
category: Idiom
language: en
tag:
 - Extensibility
---

## Intent

Execute Around习语将用户从应该总是在和之前执行的某些操作中解放出来
之后的商业方法。这方面的一个很好的例子是资源分配和再分配离开
用户只能指定如何处理资源。

## Explanation

Real-world example

>需要提供一个类来将文本字符串写入文件。使…变得容易
>用户，则服务类自动打开和关闭该文件。用户只需要这样做
>指定将什么内容写入哪个文件。     

In plain words

> Execute Around idiom handles boilerplate code before and after business method.  

[Stack Overflow](https://stackoverflow.com/questions/341971/what-is-the-execute-around-idiom) says

> 基本上，它是一种模式，你写一个方法来做一些总是需要的事情，例如。
>资源分配和清理，并让调用者传入“我们想要用
>资源”。

**Programmatic Example**

`SimpleFileWriter` class implements the Execute Around idiom. It takes `FileWriterAction` as a
constructor argument allowing the user to specify what gets written into the file.

```java
@FunctionalInterface
public interface FileWriterAction {
  void writeFile(FileWriter writer) throws IOException;
}

@Slf4j
public class SimpleFileWriter {
    public SimpleFileWriter(String filename, FileWriterAction action) throws IOException {
        LOGGER.info("Opening the file");
        try (var writer = new FileWriter(filename)) {
            LOGGER.info("Executing the action");
            action.writeFile(writer);
            LOGGER.info("Closing the file");
        }
    }
}
```

The following code demonstrates how `SimpleFileWriter` is used. `Scanner` is used to print the file
contents after the writing finishes.

```java
FileWriterAction writeHello = writer -> {
    writer.write("Gandalf was here");
};
new SimpleFileWriter("testfile.txt", writeHello);

var scanner = new Scanner(new File("testfile.txt"));
while (scanner.hasNextLine()) {
LOGGER.info(scanner.nextLine());
}
```

Here's the console output.

```
21:18:07.185 [main] INFO com.iluwatar.execute.around.SimpleFileWriter - Opening the file
21:18:07.188 [main] INFO com.iluwatar.execute.around.SimpleFileWriter - Executing the action
21:18:07.189 [main] INFO com.iluwatar.execute.around.SimpleFileWriter - Closing the file
21:18:07.199 [main] INFO com.iluwatar.execute.around.App - Gandalf was here
```

## Class diagram

![alt text](./etc/execute-around.png "Execute Around")

## Applicability

Use the Execute Around idiom when

* An API requires methods to be called in pairs such as open/close or allocate/deallocate.

## Credits

* [Functional Programming in Java: Harnessing the Power of Java 8 Lambda Expressions](https://www.amazon.com/gp/product/1937785467/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=1937785467&linkCode=as2&tag=javadesignpat-20&linkId=7e4e2fb7a141631491534255252fd08b)
