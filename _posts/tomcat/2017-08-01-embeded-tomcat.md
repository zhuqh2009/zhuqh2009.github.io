---
layout:     post
title:      从SpringBoot观察tomcat
category: blog
description: 从SpringBoot观察tomcat
---


### JVM线程、tomcat线程

JVM线程include(user thread, daemon thread);
当守护线程守护的用户线程都退出的时候，JVM就exit了。
默认创建的Thread是Daemon thread。


SprintBoot下的tomcat：
tomcat启动后，会有
container thread  （用户线程）防止JVM退出，await退出命令
ContainerBackgroundProcessor(daemon)session expire things
NioBlockingSelector.BlockPoller(daemon selectPoll线程) 就绪连接选择
http-nio-xxxx-x (daemon) http request/resp data处理线程池线程






