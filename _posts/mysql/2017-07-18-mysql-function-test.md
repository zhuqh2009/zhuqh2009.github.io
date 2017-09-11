---
layout:     post
title:      MySql常用函数积累
category: blog
description: 开发中常碰到的mySql函数需求
---


### Mysql 常见的函数使用

#### 1、替换字段中的关键字--- REPLACE

```
update tableName set name=REPLACE(name,'peter','pt')

遇到时: 保存的\n 的换行符，呈现web页面中需要替换成<br/>

update tableName set name=REPLACE(name,'\n','<br/>')

```


More to write.....

