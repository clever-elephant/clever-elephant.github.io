---
title: 实战：OutOfMemoryError异常
date: 2022-02-18
tags: jvm
categories: Java学习
cover: /img/cover/jvm.jpg
---

# 实战：OutOfMemoryError异常

## 1 Java堆异常

```java
/**
 * VM Args: -Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
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

