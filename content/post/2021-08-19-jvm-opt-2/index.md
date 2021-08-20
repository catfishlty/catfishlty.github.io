---
title: JVM调优（二）
description: JVM调优 - 分析工具
date: 2021-08-19T10:47:00+0800
lastmod: '2021-08-19T17:07:00+0800'
image: jvm-white.jpg
slug: jvm-opt-2
categories:
    - jvm
tags:
    - JVM
---

## 简述

*Java* 自带很多工具能够辅助我们完成对 *Java* 应用的性能指标分析。
本文将介绍下列 *5* 种命令行工具。
- [jps](#jps)
- [jstat](#jstat)
- [jstack](#jstack)
- [jmap](#jmap)
- [jhat](#jhat)

## jps
*JPS* 全程 *Java Virtual Machine Process Status Tool*，是 *Java* 提供的一个显示当前所有 *Java* 进程 *PID* 的命令，适合简单查看当前 *Java* 进程的信息。

详情见：[jps - Java Virtual Machine Process Status Tool](https://docs.oracle.com/javase/7/docs/technotes/tools/share/jps.html)

### 运行格式

```bash
jps [ options ] [ hostid ]
```

### 参数介绍
| 参数 | 描述 |
| :-- | :-- |
| `-q` | 仅生成本地 *VM* 标识符列表，不打印 *main* 方法的类名、*JAR* 文件名和参数的输出。 |
| `-m` | 输出传递给 *main* 方法的参数。对于嵌入式 *JVM*，输出可能为 *null*。|
| `-l` | 输出应用程序主类的完整包名或应用程序 *JAR* 文件的完整路径名。 |
| `-v` | 输出 *JVM* 的参数。 |
| `-V` | 通过标志文件（*.hotspotrc* 文件或 *-XX:Flags='filename'* 参数指定的文件）输出传递给 *JVM* 的参数。|
| `-Joption` | 将参数传递给 *JPS* 调用的 *Java* 启动器。例如，`-J-Xms48m` 将启动内存设置为 48Mb。 `-J` 将选项传递给执行应用程序的底层 *VM* 是一种一般规范。|

### 使用

```bash
$ jps
30444 rest-service-0.0.1-SNAPSHOT.jar
9460 Jps

$ jps -q
30444
9460

$ jps -m
30236 Jps -m
30444 rest-service-0.0.1-SNAPSHOT.jar -XX:+UseG1GC

$ jps -l
31812 sun.tools.jps.Jps
30444 .\rest-service-0.0.1-SNAPSHOT.jar

$ jps -v
29260 Jps -Denv.class.path=.;C:\Program Files\AdoptOpenJDK\jdk-8.0.282.8-hotspot\\lib\dt.jar;C:\Program Files\AdoptOpenJDK\jdk-8.0.282.8-hotspot\\lib\tools.jar; -Dapplication.home=C:\Program Files\AdoptOpenJDK\jdk-8.0.282.8-hotspot -Xms8m
30444 rest-service-0.0.1-SNAPSHOT.jar

$ jps -J-Xms48m -v
16060 Jps -Denv.class.path=.;C:\Program Files\AdoptOpenJDK\jdk-8.0.282.8-hotspot\\lib\dt.jar;C:\Program Files\AdoptOpenJDK\jdk-8.0.282.8-hotspot\\lib\tools.jar; -Dapplication.home=C:\Program Files\AdoptOpenJDK\jdk-8.0.282.8-hotspot -Xms8m -Xms48m
30444 rest-service-0.0.1-SNAPSHOT.jar
```

## jstat
*jstat* 工具能够显示 *HotSpot Java* 虚拟机 (*JVM*) 的性能统计信息。目标 *JVM* 由其虚拟机标识符或下面描述的 *vmid* 参数来确定。

JDK 1.7: [jstat - Java Virtual Machine Statistics Monitoring Tool](https://docs.oracle.com/javase/7/docs/technotes/tools/share/jstat.html)
JDK 1.8: [Java Platform, Standard Edition Tools Reference - jstat](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html)

### 运行格式

```bash
jstat [ generalOption | outputOptions vmid [interval[s|ms] [count]] ]
```

### 输出参数

| 参数 | 描述 | 
|  :-- | :-- |
| `-help` | 显示帮助信息 |
| `-options` | 显示统计参数列表。 |

| 参数 | 显示内容 | 
|  :-- | :-- |
| `-class`	| 类加载器行为统计信息。 |
| `-compiler`	| *HotSpot JIT* 编译器行为统计信息。 |
| `-gc`	| *GC* 堆的行为统计。 |
| `-gccapacity`	| *Heap* 中对象年龄代的容量及其相应空间的统计。 |
| `-gccause` | 垃圾收集统计信息摘要（与 `-gcutil` 相同），以及上次和当前（如果适用）垃圾收集事件的原因。|
| `-gcnew`	| 新生代行为统计。 |
| `-gcnewcapacity`	| 新生代的大小及其对应的空间的统计信息。|
| `-gcold`	| 老年代和永久代的行为统计。(JDK 1.7); 老年代和元空间的行为统计。(JDK 1.8);|
| `-gcoldcapacity`	| 老年代容量的统计。 |
| `-gcmetacapacity` | 元空间容量的统计。(JDK 1.8) |
| `-gcpermcapacity`	| 永久代容量的统计。(JDK 1.7) |
| `-gcutil`	| 垃圾收集统计摘要。 |
| `-printcompilation`	| HotSpot 编译方法统计。 |

### 使用
#### `-class`
*Class loader* 统计信息

```bash
$ jstat -class 30444
Loaded  Bytes  Unloaded  Bytes     Time
  5544 10174.8        0     0.0       1.73
```

| 参数名称 | 描述 |
| :-- | :-- |
| *Loaded* | 加载的类的数量。 |
| *Bytes* | 加载的类大小，单位：*KB* |
| *Unloaded* | 卸载的类的数量。 |
| *Bytes* | 卸载的类大小，单位：*KB* |
| *Time* | 执行类加载和卸载操作所花费的时间。|

#### `-compiler`
*Java HotSpot VM Just-in-Time* 编译器统计信息

```bash
$ jstat -compiler 30444
Compiled Failed Invalid   Time   FailedType FailedMethod
    3084      1       0     3.29          1 java/lang/StringCoding encode
```

| 参数名称 | 描述 |
| :-- | :-- |
| *Compiled* | 执行的编译数。 |
| *Failed* | 编译任务失败数。 |
| *Invalid* | 失效的编译数。|
| *Time* | 执行编译任务所花费的时间。|
| *FailedType* | 上次失败编译的编译类型。|
| *FailedMethod* | 上次编译失败的类名和方法。|

#### `-gc`

堆 *GC* 的统计信息。 

```bash
$ jstat -gc 30444
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
3072.0 10752.0 3008.0  0.0   96256.0  56913.5   104960.0    7314.0   27440.0 25574.4 3632.0 3186.9      4    0.020   1      0.018    0.038
```

| 参数名称 | 描述 |
| :-- | :-- |
| *S0C* | 当前 *Survivor 0* 的容量 (*KB*)。|
| *S1C* | 当前 *Survivor 1* 的容量 (*KB*)。|
| *S0U* | 当前 *Survivor 0* 的使用大小 (*KB*)。|
| *S1U* | 当前 *Survivor 1* 的使用大小 (*KB*)。|
| *EC* | 当前 *Eden* 区的容量 (*KB*)。|
| *EU* | 当前 *Eden* 区的使用大小 (*KB*)。|
| *OC* | 当前 *Old* 区的容量 (*KB*)。|
| *OU* | 当前 *Old* 区的使用大小 (*KB*)。|
| *PC*| 永久代的容量 (*KB*)。(**JDK 1.7**)|
| *PU* | 永久代的使用大小 (*KB*)。(**JDK 1.7**)|
| *MC* | 元空间的容量 (*KB*)。(**JDK 1.8**)|
| *MU* | 元空间的使用大小 (*KB*)。(**JDK 1.8**)|
| *CCSC* | 类压缩空间的容量 (*KB*)。(**JDK 1.8**)|
| *CCSU* | 类压缩空间的使用大小 (*KB*)。(**JDK 1.8**)|
| *YGC* | *Young GC* 触发次数。|
| *YGCT* | *Young GC* 运行时间。|
| *FGC* | *Full GC* 触发次数。|
| *FGCT* | *Full GC* 运行时间。|
| *GCT* | 总 *GC* 时间。|

#### `-gccapacity`
内存池生成和空间容量。

```bash
$ jstat -gccapacity 30444
 NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      OGCMX       OGC         OC       MCMN     MCMX      MC     CCSMN    CCSMX     CCSC    YGC    FGC
 86528.0 1379328.0 143872.0 3072.0 10752.0  96256.0   173568.0  2759680.0   104960.0   104960.0      0.0 1073152.0  27440.0      0.0 1048576.0   3632.0      4     1
```

| 参数名称 | 描述 |
| :-- | :-- |
| *NGCMN* | 新生代最小容量 (*KB*)。 |
| *NGCMX* | 新生代最大容量 (*KB*)。 |
| *NGC* | 当前新生代容量 (*KB*)。 |
| *S0C* | 当前 *Survivor 0* 的容量 (*KB*)。 |
| *S1C* | 当前 *Survivor 1* 的容量 (*KB*)。 |
| *EC* | 当前 *Eden* 区的容量 (*KB*)。 |
| *OGCMN* | 老年代最小容量 (*KB*)。 |
| *OGCMX* | 老年代最大容量 (*KB*)。 |
| *OGC* | 当前老年代容量 (*KB*)。 |
| *OC* | 当前 *Old* 空间容量 (*KB*)。 |
| *PGCMN* | 最小永久代容量 (*KB*)。(**JDK 1.7**)|
| *PGCMX* | 最大永久代容量 (*KB*)。(**JDK 1.7**)|
| *PGC* | 当前永久代容量 (*KB*)。(**JDK 1.7**)|
| *PC* | 当前 *Permanent* 空间容量 (*KB*)。(**JDK 1.7**)|
| *MCMN* | 最小元空间容量 (*KB*)。 (**JDK 1.8**)|
| *MCMX* | 最大元空间容量 (*KB*)。 (**JDK 1.8**)|
| *MC* | 当前元空间大小 (*KB*)。 (**JDK 1.8**)|
| *CCSMN* | 最小类压缩空间容量 (*KB*)。 (**JDK 1.8**)|
| *CCSMX* | 最大类压缩空间容量 (*KB*)。 (**JDK 1.8**)|
| *CCSC* | 当前类压缩空间容量 (*KB*)。 (**JDK 1.8**)|
| *YGC* | *Young GC* 触发次数。|
| *FGC* | *Full GC* 触发次数。|

#### `-gccause`
此选项显示与 **[-gcutil](#-gcutil)** 选项相同的垃圾收集统计摘要信息，但包括上次垃圾收集事件和当前垃圾收集事件的原因（如果适用）。除了 **[-gcutil](#-gcutil)** 列出的之外，此选项还添加了以下列。

```bash
$ jstat -gccause 30444
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC

 97.92   0.00  59.13   6.97  93.20  87.74      4    0.020     1    0.018    0.038 Allocation Failure   No GC

```

| 参数名称 | 描述 |
| :-- | :-- |
| *LGCC* | 上次垃圾回收的原因。|
| *GCC* | 当前垃圾回收的原因。 |

#### `-gcnew`
新生代统计信息。

```bash
$ jstat -gcnew 30444
 S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT
3072.0 10752.0 3008.0    0.0  7  15 10752.0  96256.0  56913.5      4    0.020
```

| 参数名称 | 描述 |
| :-- | :-- |
| *S0C* | 当前 *Survivor 0* 的容量 (*KB*)。 |
| *S1C* | 当前 *Survivor 1* 的容量 (*KB*)。 |
| *S0U* | 当前 *Survivor 0* 的使用大小 (*KB*)。 |
| *S1U* | 当前 *Survivor 0* 的使用大小 (*KB*)。 |
| *TT* | 对象在新生代存活的次数。 |
| *MTT* | 对象在新生代存活的最大次数。 |
| *DSS* | 期望的 *Survivor* 区大小 (*KB*)。 |
| *EC* | *Eden* 区的容量 (*KB*)。 |
| *EU* | *Eden* 区的使用大小 (*KB*)。 |
| *YGC* | *Young GC* 触发次数。 |
| *YGCT* | *Young GC* 运行时间。 |

#### `-gcnewcapacity`
新生代空间大小统计。

```bash
$ jstat -gcnewcapacity 30444
  NGCMN      NGCMX       NGC      S0CMX     S0C     S1CMX     S1C       ECMX        EC      YGC   FGC
   86528.0  1379328.0   143872.0 459776.0   3072.0 459776.0  10752.0  1378304.0    96256.0     4     1
```

| 参数名称 | 描述 |
| :-- | :-- |
| *NGCMN* | 新生代最小容量 (*KB*)。 |
| *NGCMX* | 新生代最小容量 (*KB*)。 |
| *NGC* | 当前新生代容量 (*KB*)。 |
| *S0CMX* | 最大 *Survivor 0* 的容量 (*KB*)。 |
| *S0C* | 当前 *Survivor 0* 的容量 (*KB*)。 |
| *S1CMX* | 最大 *Survivor 1* 的容量 (*KB*)。 |
| *S1C* | 当前 *Survivor 1* 的容量 (*KB*)。 |
| *ECMX* | 最大 *Eden* 区的容量 (*KB*)。 |
| *EC* | 当前 *Eden* 区的容量 (*KB*)。 |
| *YGC* | *Young GC* 触发次数。 |
| *FGC* | *Full GC* 触发次数。 |

#### `-gcold`
- 老一代和永久代统计。(**JDK 1.7**)
- 老年代和元空间行为统计。(**JDK 1.8**)

```bash
$ jstat -gcold 30444
   MC       MU      CCSC     CCSU       OC          OU       YGC    FGC    FGCT     GCT
 27440.0  25574.4   3632.0   3186.9    104960.0      7314.0      4     1    0.018    0.038
```

| 参数名称 | 描述 |
| :-- | :-- |
| *PC*| 永久代的容量 (*KB*)。(**JDK 1.7**)|
| *PU* | 永久代的使用大小 (*KB*)。(**JDK 1.7**)|
| *MC* | 元空间的容量 (*KB*)。(**JDK 1.8**)|
| *MU* | 元空间的使用大小 (*KB*)。(**JDK 1.8**)|
| *CCSC* | 类压缩空间的容量 (*KB*)。(**JDK 1.8**)|
| *CCSU* | 类压缩空间的使用大小 (*KB*)。(**JDK 1.8**)|
| *OC* | 当前 *Old* 区的容量 (*KB*)。|
| *OU* | 当前 *Old* 区的使用大小 (*KB*)。|
| *YGC* | *Young GC* 触发次数。|
| *FGC* | *Full GC* 触发次数。|
| *FGCT* | *Full GC* 运行时间。|
| *GCT* | 总 *GC* 时间。|

#### `-gcoldcapacity`
老年代大小统计信息。

```bash
$ jstat -gcoldcapacity 30444
   OGCMN       OGCMX        OGC         OC       YGC   FGC    FGCT     GCT
   173568.0   2759680.0    104960.0    104960.0     4     1    0.018    0.038
```

| 参数名称 | 描述 |
| :-- | :-- |
| *OGCMN* | 老年代最小容量 (*KB*)。 |
| *OGCMX* | 老年代最大容量 (*KB*)。 |
| *OGC* | 当前老年代容量 (*KB*)。 |
| *OC* | 当前 *Old* 区的容量 (*KB*)。|
| *YGC* | *Young GC* 触发次数。|
| *FGC* | *Full GC* 触发次数。|
| *FGCT* | *Full GC* 运行时间。|
| *GCT* | 总 *GC* 时间。|

#### `-gcpermcapacity` (**JDK 1.7 Only**)
永久代大小统计信息

| 参数名称 | 描述 |
| :-- | :-- |
| *PGCMN* | 最小永久代容量 (*KB*)。|
| *PGCMX* | 最大永久代容量 (*KB*)。|
| *PGC* | 当前永久代容量 (*KB*)。|
| *PC* | 当前 *Permanent* 空间容量 (*KB*)。|
| *YGC* | *Young GC* 触发次数。|
| *FGC* | *Full GC* 触发次数。|
| *FGCT* | *Full GC* 运行时间。|
| *GCT* | 总 *GC* 时间。|

#### `-gcmetacapacity` (**JDK 1.8 Only**)
元空间大小统计信息。

```bash
$ jstat -gcmetacapacity 30444
   MCMN       MCMX        MC       CCSMN      CCSMX       CCSC     YGC   FGC    FGCT     GCT
       0.0  1073152.0    27440.0        0.0  1048576.0     3632.0     4     1    0.018    0.038
```

| 参数名称 | 描述 |
| :-- | :-- |
| *MCMN* | 最小元空间容量 (*KB*)。|
| *MCMX* | 最大元空间容量 (*KB*)。|
| *MC* | 当前元空间大小 (*KB*)。|
| *CCSMN* | 最小类压缩空间容量 (*KB*)。|
| *CCSMX* | 最大类压缩空间容量 (*KB*)。|
| *YGC* | *Young GC* 触发次数。|
| *FGC* | *Full GC* 触发次数。|
| *FGCT* | *Full GC* 运行时间。|
| *GCT* | 总 *GC* 时间。|

#### `-gcutil`
垃圾收集统计概要。

```bash
$ jstat -gcutil 30444
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
 97.92   0.00  59.13   6.97  93.20  87.74      4    0.020     1    0.018    0.038
```

| 参数名称 | 描述 |
| :-- | :-- |
| *S0* | *Survivor 0* 空间利用率。单位：*%*。 |
| *S1* | *Survivor 1* 空间利用率。单位：*%*。 |
| *E* | *Eden* 区空间利用率。单位：*%*。 |
| *O* | *Old* 区空间利用率。单位：*%*。 |
| *P* | *Permanent* 区空间利用率。单位：*%*。(**JDK 1.7**) |
| *M* | *Metaspace* 空间利用率。单位：*%*。(**JDK 1.8**) |
| *CCS* | 类压缩空间利用率。单位：*%*。(**JDK 1.8**) |
| *YGC* | *Young GC* 触发次数。|
| *YGCT* | *Young GC* 运行时间。|
| *FGC* | *Full GC* 触发次数。|
| *FGCT* | *Full GC* 运行时间。|
| *GCT* | 总 *GC* 时间。|

#### `-printcompilation`
Java HotSpot VM 编译器方法统计信息。

```bash
$ jstat -printcompilation 30444
Compiled  Size  Type Method
    3107    172    1 java/util/concurrent/locks/AbstractQueuedSynchronizer release
```

| 参数名称 | 描述 |
| :-- | :-- |
| *Compiled* | 最近编译的方法执行的编译任务数。 |
| *Size* | 最近编译的方法的字节码的字节数。 |
| *Type* | 最近编译的方法的编译类型。 |
| *Method* | 最近编译的方法的类名和方法名。类名使用斜线 (/) 而不是点 (.) 作为名称空间分隔符。方法名称是指定类中的方法。这两个字段的格式与 *HotSpot* `-XX:+PrintCompilation` 选项一致。 |

## jstack
*jstack* 是 *Java* 虚拟机自带的一种堆栈跟踪工具。jstack 为给定的 *Java* 进程或核心文件或远程调试服务器打印 *Java* 线程的 *Java* 堆栈跟踪。对于每个 *Java* 框架，将打印完整的类名、方法名、`bci`（字节码索引）和行号（如果有）。使用 `-m` 选项，*jstack* 命令使用程序计数器 (*PC*) 打印所有线程的 *Java* 和本机帧。对于每个原生帧，打印最接近 *PC* 的原生符号（如果可用）。当指定的进程在 *64* 位 *Java* 虚拟机上运行时，您可能需要指定 `-J-d64` 选项，例如：`jstack -J-d64 -m pid`。

[Oracle - Java Documentation - jstack](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstack.html)

### 格式
```bash
jstack [ options ] pid

jstack [ options ] executable core

jstack [ options ] [ server-id@ ] remote-hostname-or-IP
```

| 关键字 | 描述 |
| :-- | :-- |
| *options* | 命令行参数 |
| *pid* | 打印堆栈跟踪的进程 *ID*。该进程必须是 *Java* 进程。 |
| *executable* | 从中生成核心转储的 *Java* 可执行文件。 |
| *core* | 要打印堆栈跟踪的核心文件。 |
| *remote-hostname-or-IP* | 远程调试服务器主机名或 *IP* 地址。 |
| *server-id* | 当多个调试服务器在同一远程主机上运行时使用的可选唯一 *ID*。|

### 参数

| 参数名称 | 描述 |
| :-- | :-- |
| `-F` | 当 `jstack [-l] pid` 没有响应时强制进行堆栈转储。| 
| `-l` | 大量列出信息。打印有关锁的附加信息，例如拥有的`java.util.concurrent` 可拥有的同步器列表。 |
| `-m` | 打印具有 *Java* 和 *Native C/C++* 帧的混合模式堆栈跟踪。| 
| `-h` | 打印帮助消息。| 
| `-help` | 打印帮助消息。| 

### 使用

```bash
$ jstack 30444
2021-08-19 16:05:29
Full thread dump OpenJDK 64-Bit Server VM (25.282-b08 mixed mode):

"DestroyJavaVM" #31 prio=5 os_prio=0 tid=0x00000187de146800 nid=0x78e8 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"http-nio-8080-Acceptor" #30 daemon prio=5 os_prio=0 tid=0x00000187de145800 nid=0x4790 runnable [0x0000005d8d2ff000]
   java.lang.Thread.State: RUNNABLE
        at sun.nio.ch.ServerSocketChannelImpl.accept0(Native Method)
        at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:421)
        at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:249)
        - locked <0x000000076ed7ec98> (a java.lang.Object)
        at org.apache.tomcat.util.net.NioEndpoint.serverSocketAccept(NioEndpoint.java:551)
        at org.apache.tomcat.util.net.NioEndpoint.serverSocketAccept(NioEndpoint.java:80)
        at org.apache.tomcat.util.net.Acceptor.run(Acceptor.java:106)
        at java.lang.Thread.run(Thread.java:748)

"http-nio-8080-ClientPoller" #29 daemon prio=5 os_prio=0 tid=0x00000187de145000 nid=0x5848 runnable [0x0000005d8d1fe000]
   java.lang.Thread.State: RUNNABLE
        at sun.nio.ch.WindowsSelectorImpl$SubSelector.poll0(Native Method)
        at sun.nio.ch.WindowsSelectorImpl$SubSelector.poll(WindowsSelectorImpl.java:314)
        at sun.nio.ch.WindowsSelectorImpl$SubSelector.access$400(WindowsSelectorImpl.java:293)
        at sun.nio.ch.WindowsSelectorImpl.doSelect(WindowsSelectorImpl.java:174)
        at sun.nio.ch.SelectorImpl.lockAndDoSelect(SelectorImpl.java:86)
        - locked <0x000000076eef6158> (a sun.nio.ch.Util$3)
        - locked <0x000000076eef6148> (a java.util.Collections$UnmodifiableSet)
        - locked <0x000000076eef5ff8> (a sun.nio.ch.WindowsSelectorImpl)
        at sun.nio.ch.SelectorImpl.select(SelectorImpl.java:97)
        at org.apache.tomcat.util.net.NioEndpoint$Poller.run(NioEndpoint.java:793)
        at java.lang.Thread.run(Thread.java:748)

"http-nio-8080-exec-10" #28 daemon prio=5 os_prio=0 tid=0x00000187de143000 nid=0x5584 waiting on condition [0x0000005d8d0fe000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000076ee9e510> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:108)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:748)

"http-nio-8080-exec-9" #27 daemon prio=5 os_prio=0 tid=0x00000187de142800 nid=0x7bb8 waiting on condition [0x0000005d8cfff000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000076ee9e510> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:108)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:748)

"http-nio-8080-exec-8" #26 daemon prio=5 os_prio=0 tid=0x00000187de148800 nid=0x83b0 waiting on condition [0x0000005d8ceff000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000076ee9e510> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:108)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:748)

"http-nio-8080-exec-7" #25 daemon prio=5 os_prio=0 tid=0x00000187de149800 nid=0x8190 waiting on condition [0x0000005d8cdfe000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000076ee9e510> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:108)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:748)

"http-nio-8080-exec-6" #24 daemon prio=5 os_prio=0 tid=0x00000187de144000 nid=0x3574 waiting on condition [0x0000005d8ccff000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000076ee9e510> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:108)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:748)

"http-nio-8080-exec-5" #23 daemon prio=5 os_prio=0 tid=0x00000187de037000 nid=0x7b64 waiting on condition [0x0000005d8cbff000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000076ee9e510> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:108)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:748)

"http-nio-8080-exec-4" #22 daemon prio=5 os_prio=0 tid=0x00000187dcb9e000 nid=0x2a50 waiting on condition [0x0000005d8cafe000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000076ee9e510> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:108)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:748)

"http-nio-8080-exec-3" #21 daemon prio=5 os_prio=0 tid=0x00000187dcb9d800 nid=0x785c waiting on condition [0x0000005d8c9fe000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000076ee9e510> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:108)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:748)

"http-nio-8080-exec-2" #20 daemon prio=5 os_prio=0 tid=0x00000187ddede800 nid=0x44c0 waiting on condition [0x0000005d8c8fe000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000076ee9e510> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:108)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:748)

"http-nio-8080-exec-1" #19 daemon prio=5 os_prio=0 tid=0x00000187dd29d800 nid=0x4444 waiting on condition [0x0000005d8c7fe000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000076ee9e510> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:108)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:748)

"http-nio-8080-BlockPoller" #18 daemon prio=5 os_prio=0 tid=0x00000187dea3b800 nid=0x1d70 runnable [0x0000005d8c6fe000]
   java.lang.Thread.State: RUNNABLE
        at sun.nio.ch.WindowsSelectorImpl$SubSelector.poll0(Native Method)
        at sun.nio.ch.WindowsSelectorImpl$SubSelector.poll(WindowsSelectorImpl.java:314)
        at sun.nio.ch.WindowsSelectorImpl$SubSelector.access$400(WindowsSelectorImpl.java:293)
        at sun.nio.ch.WindowsSelectorImpl.doSelect(WindowsSelectorImpl.java:174)
        at sun.nio.ch.SelectorImpl.lockAndDoSelect(SelectorImpl.java:86)
        - locked <0x000000076ed81670> (a sun.nio.ch.Util$3)
        - locked <0x000000076ed815e8> (a java.util.Collections$UnmodifiableSet)
        - locked <0x000000076ed811e8> (a sun.nio.ch.WindowsSelectorImpl)
        at sun.nio.ch.SelectorImpl.select(SelectorImpl.java:97)
        at org.apache.tomcat.util.net.NioBlockingSelector$BlockPoller.run(NioBlockingSelector.java:313)

"container-0" #17 prio=5 os_prio=0 tid=0x00000187de6c4800 nid=0x5488 waiting on condition [0x0000005d8c5ff000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at org.apache.catalina.core.StandardServer.await(StandardServer.java:570)
        at org.springframework.boot.web.embedded.tomcat.TomcatWebServer$1.run(TomcatWebServer.java:197)

"Catalina-utility-2" #16 prio=1 os_prio=-2 tid=0x00000187dea67000 nid=0x66f4 waiting on condition [0x0000005d8c4fe000]
   java.lang.Thread.State: TIMED_WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000007746f4b70> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:215)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2078)
        at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:1093)
        at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:809)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:748)

"Catalina-utility-1" #15 prio=1 os_prio=-2 tid=0x00000187de4b8800 nid=0x3bf4 waiting on condition [0x0000005d8c3fe000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000007746f4b70> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:1088)
        at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:809)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:748)

"Service Thread" #10 daemon prio=9 os_prio=0 tid=0x00000187dc61f000 nid=0x2b7c runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C1 CompilerThread3" #9 daemon prio=9 os_prio=2 tid=0x00000187da84a800 nid=0x5184 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread2" #8 daemon prio=9 os_prio=2 tid=0x00000187da841800 nid=0x42f0 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread1" #7 daemon prio=9 os_prio=2 tid=0x00000187da840800 nid=0x5c34 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #6 daemon prio=9 os_prio=2 tid=0x00000187da844800 nid=0x4184 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Attach Listener" #5 daemon prio=5 os_prio=2 tid=0x00000187da838000 nid=0x2638 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Signal Dispatcher" #4 daemon prio=9 os_prio=2 tid=0x00000187da7e5000 nid=0x5218 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" #3 daemon prio=8 os_prio=1 tid=0x00000187da7b0800 nid=0x7a3c in Object.wait() [0x0000005d8b9ff000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000006c3630078> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)
        - locked <0x00000006c3630078> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:165)
        at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:216)

"Reference Handler" #2 daemon prio=10 os_prio=2 tid=0x00000187da7a8800 nid=0x685c in Object.wait() [0x0000005d8b8ff000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000006c36099b0> (a java.lang.ref.Reference$Lock)
        at java.lang.Object.wait(Object.java:502)
        at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
        - locked <0x00000006c36099b0> (a java.lang.ref.Reference$Lock)
        at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

"VM Thread" os_prio=2 tid=0x00000187da77d800 nid=0x2fd4 runnable

"GC task thread#0 (ParallelGC)" os_prio=0 tid=0x00000187bf85c800 nid=0x715c runnable

"GC task thread#1 (ParallelGC)" os_prio=0 tid=0x00000187bf85e000 nid=0x558c runnable

"GC task thread#2 (ParallelGC)" os_prio=0 tid=0x00000187bf85f800 nid=0x2b90 runnable

"GC task thread#3 (ParallelGC)" os_prio=0 tid=0x00000187bf861800 nid=0x83f4 runnable

"GC task thread#4 (ParallelGC)" os_prio=0 tid=0x00000187bf863800 nid=0x3198 runnable

"GC task thread#5 (ParallelGC)" os_prio=0 tid=0x00000187bf866000 nid=0x413c runnable

"GC task thread#6 (ParallelGC)" os_prio=0 tid=0x00000187bf869000 nid=0x607c runnable

"GC task thread#7 (ParallelGC)" os_prio=0 tid=0x00000187bf86b800 nid=0x49cc runnable

"VM Periodic Task Thread" os_prio=2 tid=0x00000187dc631000 nid=0x5018 waiting on condition

JNI global references: 1315
```

## jmap
打印进程、核心文件或远程调试服务器的共享对象内存映射或堆内存详细信息。
*jmap* 命令打印指定进程、核心文件或远程调试服务器的共享对象内存映射或堆内存详细信息。如果指定的进程在 *64* 位 *Java* 虚拟机 (*JVM*) 上运行，那么您可能需要指定 `-J-d64` 选项，例如：`jmap -J-d64 -heap pid`。

[Oracle - Java Documentation - jmap](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jmap.html)

### 格式

```bash
jmap [ options ] pid

jmap [ options ] executable core

jmap [ options ] [ pid ] server-id@ ] remote-hostname-or-IP
```

| 关键字 | 描述 |
| :-- | :-- |
| *options* | 命令行参数 |
| *pid* | 要为其打印内存映射的进程 *ID*。该进程必须是 *Java* 进程。 |
| *executable* | 从中生成核心转储的 *Java* 可执行文件。 |
| *core* | 要为其打印内存映射的核心文件。 |
| *remote-hostname-or-IP* | 远程调试服务器主机名或 *IP* 地址。 |
| *server-id* | 当多个调试服务器在同一远程主机上运行时使用的可选唯一 *ID*。|

### 参数

| 参数名称 | 描述 |
| :-- | :-- |
| `<no option>` | 当不使用任何选项时，*jmap* 命令打印共享对象映射。对于目标JVM 中加载的每个共享对象，都会打印出该共享对象文件的起始地址、映射大小和完整路径。。 |
| `-dump:[live,] format=b, file=filename` | 将 *hprof* 二进制格式的 *Java* 堆转储到指定文件。 `live` 是可选子选项，当指定时，只会转储堆中的活动对象，且会触发一次 *Full GC*。要浏览堆转储，您可以使用 [jhat](#jhat) 命令读取生成的 *Dump* 文件。 |
| `-finalizerinfo` | 打印有关等待完成的对象的信息。 |
| `-heap` | 打印使用的垃圾收集的堆摘要、头配置和按代计算的堆使用情况。此外，还打印了已存入常量池字符串([Interned String](https://blog.csdn.net/guluhan/article/details/82385808))的数量和大小。 |
| `-histo[:live]` | 打印堆的直方图。对于每个 *Java* 类，将打印对象数、内存大小（以*Byte* 为单位）和完整的类名。*JVM* 内部类以星号 (\*) 前缀打印。如果指定了 `live` 子选项，则只计算活动对象，且会触发 *Full GC*。 |
| `-clstats` | 打印 *Java* 堆的类加载器的统计信息。对于每个类加载器，它的名称、它的活跃程度、地址、父类加载器以及它已加载的类的数量和大小都被打印出来。 |
| `-F` | 强制执行。当 *pid* 没有响应时，将此选项与 `jmap -dump` 或 `jmap -histo` 选项一起使用。此模式不支持 *live* 子选项。 |
| `-h` | Prints a help message. |
| `-help` | Prints a help message. |
| `-Jflag` | 将 *JVM*参数传递给运行 `jmap` 命令的 *Java* 虚拟机。 |

### 使用

```bash
$ jmap 30444
Attaching to process ID 30444, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.282-b08
0x0000000067ec0000      8616K   C:\Program Files\AdoptOpenJDK\jdk-8.0.282.8-hotspot\jre\bin\server\jvm.dll
0x0000000068c90000      664K    C:\windows\System32\SYSFER.DLL
0x00007ff7f0680000      232K    C:\Program Files\AdoptOpenJDK\jdk-8.0.282.8-hotspot\bin\java.exe
0x00007ff88ce30000      956K    C:\Program Files\AdoptOpenJDK\jdk-8.0.282.8-hotspot\jre\bin\msvcr120.dll
0x00007ff88df40000      664K    C:\Program Files\AdoptOpenJDK\jdk-8.0.282.8-hotspot\jre\bin\msvcp120.dll
0x00007ff8bddc0000      72K     C:\Program Files\AdoptOpenJDK\jdk-8.0.282.8-hotspot\jre\bin\nio.dll
0x00007ff8bde40000      104K    C:\Program Files\AdoptOpenJDK\jdk-8.0.282.8-hotspot\jre\bin\net.dll
0x00007ff8c86d0000      88K     C:\Program Files\AdoptOpenJDK\jdk-8.0.282.8-hotspot\jre\bin\zip.dll
...
```

#### 打印堆信息
```bash
$ jmap -heap 30444
Attaching to process ID 30444, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.282-b08

using thread-local object allocation.
Parallel GC with 8 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 4238344192 (4042.0MB)
   NewSize                  = 88604672 (84.5MB)
   MaxNewSize               = 1412431872 (1347.0MB)
   OldSize                  = 177733632 (169.5MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 96468992 (92.0MB)
   used     = 964752 (0.9200592041015625MB)
   free     = 95504240 (91.07994079589844MB)
   1.0000643522843071% used
From Space:
   capacity = 5767168 (5.5MB)
   used     = 0 (0.0MB)
   free     = 5767168 (5.5MB)
   0.0% used
To Space:
   capacity = 8912896 (8.5MB)
   used     = 0 (0.0MB)
   free     = 8912896 (8.5MB)
   0.0% used
PS Old Generation
   capacity = 152567808 (145.5MB)
   used     = 10801024 (10.3006591796875MB)
   free     = 141766784 (135.1993408203125MB)
   7.079490845146048% used

12198 interned Strings occupying 1055840 bytes.
```

#### 打印直方图

```bash
$ jmap -histo 30444

 num     #instances         #bytes  class name
----------------------------------------------
   1:        174537       32059552  [C
   2:         18029       10376192  [B
   3:         12024        5422296  [I
   4:        120292        2887008  java.lang.String
   5:         23155        1576096  [Ljava.lang.Object;
   6:         15764        1008896  java.net.URL
   7:          9748         857824  java.lang.reflect.Method
   8:         34905         721848  [Ljava.lang.Class;
   9:          5977         660736  java.lang.Class
  10:         19036         609152  java.util.concurrent.ConcurrentHashMap$Node
  11:          6649         532944  [Ljava.util.WeakHashMap$Entry;
  12:          6217         490744  [S
  13:          6596         422144  org.springframework.boot.loader.jar.JarFileWrapper
  14:         10008         400320  java.lang.ref.Finalizer
  15:         12209         390688  org.springframework.boot.loader.jar.StringSequence
  16:          4053         361672  [Ljava.util.HashMap$Node;
  17:          8640         345600  java.util.LinkedHashMap$Entry
  18:         10670         341440  java.util.ArrayList$Itr
  19:         10071         322272  java.util.concurrent.locks.AbstractQueuedSynchronizer$Node
  20:          6645         318960  java.util.WeakHashMap
  21:          9621         307872  java.util.HashMap$Node
  22:         12209         293016  org.springframework.boot.loader.jar.JarURLConnection$JarEntryName
  23:          4851         271656  jdk.internal.org.objectweb.asm.Item
  24:           133         258512  [Ljava.util.concurrent.ConcurrentHashMap$Node;
  25:          7366         235712  java.lang.ref.ReferenceQueue
  26:          6632         212224  java.util.zip.ZipCoder
  27:          3646         204176  java.util.LinkedHashMap
  28:          2516         201280  org.springframework.boot.loader.jar.JarURLConnection
  29:          3454         193424  java.util.concurrent.ConcurrentHashMap$KeyIterator
  30:         11965         191440  java.lang.Object
  31:          6765         162360  java.util.ArrayDeque
  32:          3341         160368  java.util.zip.Inflater
  33:           168         158400  [Ljdk.internal.org.objectweb.asm.Item;
  34:          3588         143520  java.util.HashMap$KeyIterator
  35:          3614         137896  [Ljava.lang.reflect.Method;
  36:          1980         132072  [Ljava.lang.String;
  37:          1763         126936  java.lang.reflect.Field
  38:          1052         126240  org.springframework.boot.loader.jar.JarEntry
  39:          1556         124480  java.lang.reflect.Constructor
  40:          7368         117888  java.lang.ref.ReferenceQueue$Lock
  41:          4855         116520  java.lang.StringBuilder
  42:          2278         109344  org.springframework.util.ConcurrentReferenceHashMap$SoftEntryReference
  43:          2229         106992  java.util.HashMap
  44:          4451         106824  java.util.ArrayList
  45:          3169         101408  java.util.LinkedHashMap$LinkedKeyIterator
  46:          1644          92064  java.lang.invoke.MemberName
  47:          2297          91880  java.util.TreeMap$Entry
  48:          1914          91872  org.springframework.core.ResolvableType
  49:          3634          87216  java.util.Collections$UnmodifiableCollection$1
  50:          2153          86120  java.lang.invoke.MethodType
...
Total        767708       68849112
```

#### 打印直方图 - 存活对象

```bash
$ jmap -histo:live 30444

 num     #instances         #bytes  class name
----------------------------------------------
   1:         29265        2933672  [C
   2:         28974         695376  java.lang.String
   3:          5977         660736  java.lang.Class
   4:         17063         546016  java.util.concurrent.ConcurrentHashMap$Node
   5:          6121         538648  java.lang.reflect.Method
   6:          7804         477144  [Ljava.lang.Object;
   7:          3686         366600  [I
   8:          2560         321960  [B
   9:          6117         244680  java.util.LinkedHashMap$Entry
  10:          2719         235344  [Ljava.util.HashMap$Node;
  11:          2605         209424  [Ljava.util.WeakHashMap$Entry;
  12:          6069         194208  java.util.HashMap$Node
  13:         11619         185904  java.lang.Object
  14:           101         181904  [Ljava.util.concurrent.ConcurrentHashMap$Node;
  15:          7568         170520  [Ljava.lang.Class;
  16:          2553         163392  org.springframework.boot.loader.jar.JarFileWrapper
  17:          2759         154504  java.util.LinkedHashMap
  18:          3596         143840  java.lang.ref.Finalizer
  19:          2602         124896  java.util.WeakHashMap
  20:          3131         100192  java.lang.ref.ReferenceQueue
  21:          2589          82848  java.util.zip.ZipCoder
  22:          1002          80160  java.lang.reflect.Constructor
  23:          1058          76176  java.lang.reflect.Field
  24:          1308          73248  java.lang.invoke.MemberName
  25:          2589          62136  java.util.ArrayDeque
  26:          1261          60528  java.util.HashMap
  27:           482          57840  org.springframework.boot.loader.jar.JarEntry
  28:           941          52696  java.lang.Class$ReflectionData
  29:          1254          50160  java.lang.ref.SoftReference
  30:          3133          50128  java.lang.ref.ReferenceQueue$Lock
  31:           984          47232  java.util.zip.Inflater
  32:           937          46024  [Ljava.lang.String;
  33:           792          45120  [Ljava.lang.reflect.Method;
  34:          1090          43600  java.lang.invoke.MethodType
  35:           939          37560  java.util.TreeMap$Entry
  36:          1152          36864  java.lang.invoke.MethodType$ConcurrentWeakInternSet$WeakEntry
  37:           752          36096  org.springframework.core.ResolvableType
  38:             1          32784  [Ljava.util.concurrent.ForkJoinTask;
  39:          1013          32416  java.util.concurrent.locks.ReentrantLock$NonfairSync
  40:          1094          26256  java.util.ArrayList
  41:           806          25792  java.lang.invoke.DirectMethodHandle
  42:           128          24576  org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader$ConfigurationClassBeanDefinition
  43:           984          23616  java.util.zip.ZStreamRef
  44:           512          20480  org.springframework.util.ConcurrentReferenceHashMap$Segment
  45:           512          20088  [Lorg.springframework.util.ConcurrentReferenceHashMap$Reference;
  46:           351          19656  java.lang.Package
  47:           809          19416  sun.reflect.annotation.AnnotationInvocationHandler
  48:           535          15720  [Ljava.lang.CharSequence;
  49:           487          15584  org.springframework.boot.loader.jar.AsciiBytes
  50:           475          15200  java.lang.invoke.LambdaForm$Name
...
Total        215157       10801024
```

#### *Dump* 文件生成

```bash
$ jmap -dump:format=b,file=./dump.hprof 30444
Dumping heap to E:\temp\dump.hprof ...
Heap dump file created
```

#### *Dump* 文件生成 - 存活对象

```bash
$ jmap -dump:live,format=b,file=./dump_live.hprof 30444
Dumping heap to E:\temp\dump_live.hprof ...
Heap dump file created
```

## jhat
*jhat* 命令解析 *Java* 堆转储文件并启动 *Web* 服务器。* jhat* 命令允许您使用您喜欢的 *Web* 浏览器浏览堆转储。 *jhat* 命令支持预先设计的查询，例如显示已知类 *MyClass* 和对象查询语言 (*OQL*) 的所有实例。除了查询堆转储之外，*OQL* 与 *SQL* 类似。可以从 *jhat* 命令显示的 *OQL* 帮助页面获得有关 *OQL* 的帮助。使用默认端口，可在 http://localhost:7000/oqlhelp/ 获得 *OQL* 帮助

[Oracle - Java Documentation - jhat](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jhat.html)

### 参数

| 参数 | 描述 |
| :-- | :-- |
| `-stack false|true` | 关闭跟踪对象分配调用堆栈。如果堆转储中没有分配站点信息，则必须将该参数设置为 *false*。默认值为 *true*。 |
| `-refs false|true` | 关闭对对象引用的跟踪。默认为真。默认情况下，为堆中的所有对象计算返回指针，即指向指定对象（例如引用者或传入引用）的对象。 |
| `-port port-number` | 设置 *jhat HTTP* 服务器的端口。默认值为 *7000*。 |
| `-exclude exclude-file` | 指定一个文件，该文件列出应从可达对象查询中排除的数据成员。例如，如果文件列出了 *java.lang.String.value* ，那么无论何时计算从特定对象 *o* 可达的对象列表，都不会考虑涉及 *java.lang.String.value* 字段的引用路径。 |
| `-baseline exclude-file` | 指定基线堆转储。两个堆转储中具有相同对象 *ID* 的对象都被标记为不是新对象。其他对象被标记为新对象。这对于比较两个不同的堆转储很有用。 |
| `-debug int` | 设置此工具的调试级别。级别 *0* 表示没有调试输出。为更详细的模式设置更高的值。 |
| `-version` | 输出版本号并退出 |
| `-h` | 显示帮助消息并退出。 |
| `-help` | 显示帮助消息并退出。 |
| `-Jflag` | 将 *JVM* 参数传递给运行 *jhat* 命令的 *Java* 虚拟机。例如，`-J-Xmx512m` 使用 *512 MB* 的最大堆大小。对于分析 *Dump* 大小较大的文件，需要将 *JVM* 调至 *6GB* 或更高。 |

### 使用

该命令将分析 *dump_live.hprof* 堆转储文件，并启动 *Web* 服务，发布至本地 *8081* 端口。
```bash
$ jhat -port 8081 dump_live.hprof
Reading from dump_live.hprof...
Dump file created Thu Aug 19 16:47:50 CST 2021
Snapshot read, resolving...
Resolving 188752 objects...
Chasing references, expect 37 dots.....................................
Eliminating duplicate references.....................................
Snapshot resolved.
Started HTTP server on port 8081
Server is ready.
```
