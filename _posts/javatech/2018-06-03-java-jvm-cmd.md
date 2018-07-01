---
layout:     post
title:      java JVM 调优常用CMD
category: blog
description: Java JVM memory,gc,cpu....
---


### Java JVM
First查看java进程：
   ```
    ps -ef|grep java
	jps -l （显示java进程的Id和软件名称）
	jps -lmv（显示java进程的Id和软件名称；显示启动main输入参数；虚拟机参数）
   ```

jmap:
```
jmap -heap pid  查看进程堆内存使用情况，包括使用的GC算法、堆配置参数和各代中堆内存使用情况

jmap -histo[:live] pid  查看堆内存中的对象数目、大小统计直方图，如果带上live则只统计活对象

jmap -dump:format=b,file=dumpFileName  用jmap把进程内存使用情况dump到文件中，再用jhat分析查看
```

jstat:
```
jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
exp: jstat -gcutil pid

S0  — Heap上的 Survivor space 0 区已使用空间的百分比
S1  — Heap上的 Survivor space 1 区已使用空间的百分比
E   — Heap上的 Eden space 区已使用空间的百分比
O   — Heap上的 Old space 区已使用空间的百分比
P   — Perm space 区已使用空间的百分比
YGC — 从应用程序启动到采样时发生 Young GC 的次数
YGCT– 从应用程序启动到采样时 Young GC 所用的时间(单位秒)
FGC — 从应用程序启动到采样时发生 Full GC 的次数
FGCT– 从应用程序启动到采样时 Full GC 所用的时间(单位秒)
GCT — 从应用程序启动到采样时用于垃圾回收的总时间(单位秒)

```

jstack:
```
```


dump:
```
```

查看GC常用命令：
```
```

查看CPU常用命令：
```
```

查看磁盘状况常用命令：
```
```

查看内存状况常用命令：
```
```

查看IO状况常用命令：
```
```

查看网络常用命令：
```
```

查看线程Thread状况常用命令：
```
```

###  Tools

dump内存分析工具：






