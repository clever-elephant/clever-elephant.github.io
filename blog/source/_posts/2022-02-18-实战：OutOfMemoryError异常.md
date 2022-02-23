---
title: 实战：OutOfMemoryError异常
date: 2022-02-18
tags: jvm
categories: Java学习
cover: /img/cover/jvm.jpg
---

# 实战：OutOfMemoryError异常

## 1 Java堆异常

- 异常代码

```java
/**
 * VM Args: -Xms1m -Xmx1m -XX:+HeapDumpOnOutOfMemoryError -XX:-UseGCOverheadLimit
 * Java堆OutOfMemoryError
 *
 **/
public class HeapOOM {
    static class OOMObject {
    }

    public static void main(String[] args) {
        List<OOMObject> list = new ArrayList<>();
        while (true) {
            list.add(new OOMObject());
        }
    }
}
```

- 异常信息

```shell
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid4184.hprof ...
Heap dump file created [12525670 bytes in 0.056 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at per.zida.jvm.error.HeapOOM.main(HeapOOM.java:20)
```

## 2 虚拟机栈和本地方法栈溢出

> 栈容量是通过`-Xss`参数来设定的，在《Java虚拟机规范》中描述了两种异常：
>
> - 如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出`StackOverflowError`异常
> - 如果虚拟机的栈内存允许动态扩展，当扩展栈容量无法申请到足够的内存时，将抛出`OutOfMemoryError`异常

```java
public class JavaVMStackSOF {

    private int stackLength;

    public void stackLeak() {
        stackLength++;
        stackLeak();
    }

    public static void main(String[] args) {
        JavaVMStackSOF sof = new JavaVMStackSOF();
        try {
            sof.stackLeak();
        } catch (Throwable e) {
            System.out.println(sof.stackLength);
            e.printStackTrace();
        }
    }
}
```

## 3 方法区和运行时常量池溢出

