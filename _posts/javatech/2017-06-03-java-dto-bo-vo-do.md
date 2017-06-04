---
layout:     post
title:      java 分层设计对象模型--POJO(VO/DTO/DO/BO)
category: blog
description: Java Layered Object Model
---


### Java POJO


What is POJO?
```
POJO是 Plain old Java object的缩写，其创造者是Martin Fowler.
```
```
这是Martin Fowler的陈述
In the talk we were pointing out the many benefits of encoding business logic into regular java objects rather than using Entity Beans. We wondered why people were so against using regular objects in their systems and concluded that it was because simple objects lacked a fancy name. So we gave them one, and it's caught on very nicely.
意思也就是说说：
用常规的JAVA对象比起用EJB里的Entity Bean有更多的好处，人们反对用常规java对象是因为常规java对象缺少一个漂亮的名字，所以作者就起了这也样一个POJO的名字。

总的来说，POJO就是封装了业务逻辑的简单地JAVA对象，不受任何特殊的限定（JavaBean:getter/setter命名，默认无参public构造方法，实现Serializable or Externalizable, private member）可以说JavaBean都是POJO，反之不亦然。
POJO不像EntityBean针对持久化需要的id字段，持久化注解等此类约束，也不会受制于任何框架。
它就是简单的灵活，不需要实现接口而具备业务能力，也不需要被设定注解。
```

### POJO在开发中

POJO在实际开发应用中有很多表现，比如DO（DAO）/DTO/BO/VO
阿里巴巴的JAVA开发规范中就提及：
```
领域模型命名规约 
1） 数据对象：xxxDO，xxx即为数据表名。 
2） 数据传输对象：xxxDTO，xxx为业务领域相关的名称。 
3） 展示对象：xxxVO，xxx一般为网页名称。 
4） POJO是DO/DTO/BO/VO的统称，禁止命名成xxxPOJO
```
其实DTO一般用在过程间的数据传输，实现序列号接口算不上一个严重违反POJO概念的地方，序列号接口只是一个标识，算不上什么严重的依赖。
```
业务对象：xxxBO, xxx为相应业务的名称，一般与其他O结合使用，可以包含子BO。
BO是包含某个业务的数据和行为的对象。DTO通常不具有业务行为，并且通常是某个DO或者BO的数据子集，在微服务系统中的RPC阶段，进行序列化进行传输。
当然要根据具体情况绝对是否有必要增加DTO，BO作为业务对象在微服务间传输也是可以的。
DTO通常为了减少字段的传输进行的子集映射。
POJO的使用比较灵活，要因地适宜，更不能过度设计。
```
  
### 结语

各种POJO其实是分层应用开发设计的一种体现。

Presentation layer： 表现层包含UI呈现逻辑

Controller layer： 控制层聚合服务

Service layer： 包含所有的业务逻辑

Persistence layer： 包含持久化逻辑