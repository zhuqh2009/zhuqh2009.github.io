---
layout:    post
title:     storm 启动关闭脚本
category: blog
description:      storm 远程批量启动关闭脚本
---


### supervisor关停脚本

#! /bin/sh

function killJStormOnHost(){
 echo "Kill jstorm on server:$1"
 existed=`ssh username@$1 'ps -ef| grep supervisor|grep -v grep'`
 echo "$existed"
 if [ ! -n "$existed" ]; then
  echo -e "Nothing to kill!\n\n"
  return
 else
  echo -e "Something to kill!\n\n" 
  ssh username@$1 'sudo ps -ef | grep supervisor|grep -v grep| awk "{print \$2}"| xargs sudo kill'
  echo "Killed supervisor on host:$1 ........................."
 fi
}

function stopHostJStorm(){
 echo "Param HostId:$1"
 echo "Host $1 jstorm killing..."
 killJStormOnHost "hostprefix${1}.wap.cn"
}

for((i=1;i<=8;i++));  
 do
  stopHostJStorm "$i"
  sleep 1
 done 
 
 
 ### supervisor启动脚本
 
 #! /bin/sh

function startSupervisorOnHost(){
 echo "Starting storm supervisor onHost:$1"
 ssh username@$1 'nohup sudo /home/corp/storm/apache-storm-1.0.2/bin/storm supervisor > /dev/null 2>&1 &'
 sleep 2
 echo "Started with pid below................."
 ssh username@$1 'sudo ps -ef | grep supervisor |grep -v grep| awk "{print \$2}"'
}

for((i=1;i<=8;i++));  
 do
  startSupervisorOnHost "hostprefix${i}.wap.cn"
  sleep 1
 done 