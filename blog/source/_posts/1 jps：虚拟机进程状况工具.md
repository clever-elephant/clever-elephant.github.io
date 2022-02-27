```
title: 虚拟机性能监控，故障处理工具
date: 2022-02-27
tags: jvm
categories: Java学习
cover: /img/cover/jvm.jpg
```

## 1 jps：虚拟机进程状况工具

jps可以列出正在运行的虚拟机进程，并显示虚拟机执行主类名称以及这些进程的本地唯一id。

jps命令格式：

```shell
jps [options] [hostid]
```

| 选项 | 作用                                                 |
| :--: | :--------------------------------------------------- |
|  -q  | 只输出LVMID，省略主类的名称                          |
|  -m  | 输出虚拟机进程启动时传递给主类main()函数的参数       |
|  -l  | 输出主类的全名，如果进程执行的事jar包，则输出jar路径 |
|  -v  | 输出虚拟机进程启动时的JVM参数                        |

# 2 jstat:虚拟机统计信息检测工具

jstat是用于监视虚拟机各种运行状态信息的命令行工具，它可以显示本地或者远程虚拟机进程中的类加载，内存，垃圾收集，即时编译等运行时数据，在没有GUI图形界面，只提供纯文本控制台环境的服务器上，它将是运行期间定位虚拟机性能问题的常用工具。

jstat命令格式：

```shell
jstat [option vmid [interval[s|ms] [count]]]
```

> 对于命令格式中的VMID与LVMID需要特别说明下：如果是本地虚拟机进程，VMID与LVMID是一致的；如果是远程虚拟机进程，那VMID的格式应当是
>
> ```shell
> [protocol:][//]lvmid[@hostname[:port]/servername]
> ```
>
> 参数`interval`和`count`代表查询间隔和次数，如果省略这两个参数，说明只查询一次。假设需要每250毫秒查询一次进程2764垃圾收集状况，一共查询20次，命令如下：
>
> ```shell
> jstat -gc 2764 250 20
> ```

| 选项              | 作用                                                         |
| ----------------- | ------------------------------------------------------------ |
| -class            | 监视类加载，卸载数量，总空间以及类装载所耗费时间             |
| -gc               | 监视Java堆状况，包括Eden区，2个Survivor区，老年代，永久代等的容量，已用空间，垃圾收集时间合计等信息 |
| -gccapacity       | 监视内容与-gc基本相同，但输出主要关注Java堆各个区域使用到的最大，最小空间 |
| -gcutil           | 监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比 |
| -gccause          | 与-gcutil功能一样，但是会额外输出导致上次垃圾收集产生的原因  |
| -gcnew            | 监视新生代垃圾收集状况                                       |
| -gcnewcapacity    | 监视内容与-gcnew基本相同，输出主要关注使用到的最大，最小空间 |
| -gcold            | 监视老年代垃圾收集状况                                       |
| -gcoldcapacity    | 监视内容与-gcold基本相同，输出主要关注使用到最大，最小空间   |
| -gcpermcapacity   | 输出永久代使用到的最大，最小空间                             |
| -compiler         | 输出即时编译器编译过的方法，耗时等信息                       |
| -printcompilation | 输出已经被即时编译的方法                                     |

jstat执行样例：

```shell
jps
4533 NettyApplication
jstat -gcutil 4533
  S0     S1     E      O      M     CCS    YGC     YGCT     FGC    FGCT     CGC    CGCT       GCT   
  0.00   0.00  88.46  12.79  95.26  93.05      8     0.044     3     0.119     -         -     0.163
```

查询结果表明：这台服务器的新生代`Eden区`（E，表示Eden）使用了 的空间，2个`Survivor区`（S0，S1，表示Survivor0，Survivor1）里面都是空的，`老年代`（O，表示Old）和`永久代`（P，Permanent）则分别使用了。 和。的空间。程序运行以来共发生`Minor GC` （YGC，Young GC）。次，总耗时。秒；发生`Full GC`（FGC，表示Full GC）。次，总耗时。秒；所有GC总耗时（GCT，表示GC Time） 为。秒

# 3 jinfo:Java配置信息工具





