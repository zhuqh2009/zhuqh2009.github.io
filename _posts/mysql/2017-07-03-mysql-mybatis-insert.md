---
layout:     post
title:      MyBatis insert Statement的常用办法
category: blog
description: 常见的MyBatis insert声明语句用法
---


### MyBatis insert 声明使用

#### 1、一般的insert
```
//一般的insert,会返回成功插入的条数
<insert id="insertEntity" parameterType="Entity">
 insert into.....
</insert>

```

#### 2、使用生成的键值之 useGeneratedKeys="true"

```
<insert id="insertEntity" useGeneratedKeys="true"
    keyProperty="id" parameterType="Entity">
  insert into.....
</insert>


class Entity{
  private Long id;
  //other fields
}

keyProperty指定了需要生成的目标字段， jdbc生成键值的getGenereatedKeys方法会用keyProperty的值。
一般来说，都是主键ID自增， insert后会赋值Entity中的id字段。

mapper生成器常命名这种方式的id 为 "insertSelective"

```

#### 3、自助生成键值之 selectKey

对于不支持自动生成类型的数据库或可能不支持自动生成主键的JDBC，2中的getGenereatedKeys方法肯定不可用，自然
useGeneratedKeys="true"就不起作用了

```
\\官网提供的一个说是很傻的生成id的方式

<insert id="insertEntity">
  <selectKey keyProperty="id" resultType="int" order="BEFORE">
    select CAST(RANDOM()*1000000 as INTEGER) a from SYSIBM.SYSDUMMY1
  </selectKey>
  insert into...
</insert>

keyProperty	selectKey 语句结果应该被设置的目标属性。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。
keyColumn	匹配属性的返回结果集中的列名称。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。
resultType	结果的类型。MyBatis 通常可以推算出来，但是为了更加确定写上也不会有什么问题。MyBatis 允许任何简单类型用作主键的类型，包括字符串。如果希望作用于多个生成的列，则可以使用一个包含期望属性的 Object 或一个 Map。
order	这可以被设置为 BEFORE 或 AFTER。如果设置为 BEFORE，那么它会首先选择主键，设置 keyProperty 然后执行插入语句。如果设置为 AFTER，那么先执行插入语句，然后是 selectKey 元素 - 这和像 Oracle 的数据库相似，在插入语句内部可能有嵌入索引调用。
statementType	与前面相同，MyBatis 支持 STATEMENT，PREPARED 和 CALLABLE 语句的映射类型，分别代表 PreparedStatement 和 CallableStatement 类型。
```


### 4、 不属于mybatis范畴， 全局一致性递增，并获得当前所递增的值
//业务场景， 按访问序号抽奖

#### 4.1 记录插入递增
如果是自增的字段，插入记录后返回的id值就是当前操作的全局id值。 
这是借助了数据库的自增，实现全局同步递增。

#### 4.2 单记录，值++ ，值递增

多线程，多点的状况下，递增操作肯定要保证同步的。

简单的业务，不需借助redis，zookeeper等服务实现锁时，就尽量借用DB支持的方式实现同步。

mysql的Innodb是行锁的， 所以update一条记录是同步的。

当update之后，紧接着进行select操作，却不能保证获取到的是刚递增后的值(两条语句不是业务块, 非原子)。有解决办法吗？


#### 4.3 LAST_INSERT_ID()


LAST_INSERT_ID() 能够很好的保留当前操作的现场数据。

LAST_INSERT_ID()是存在于连接对象里的，可以对其进行存取。

```
update gobal_table set value=LAST_INSERT_ID(value+1) where name='keyName';select LAST_INSERT_ID();

只要保证这两条语句是同一个连接连续执行的即可。使用的连接池是没有问题的。
mybatis里就可以写到mapper文件中的一个<select></select>中

再者，加入初始化的value

insert into global (name, value) values (#{keyName}, LAST_INSERT_ID(1)) ON DUPLICATE KEY UPDATE value=LAST_INSERT_ID(value+1) ; select LAST_INSERT_ID();

虽然两条语句不会保证是原子的，但是select返回的是是前一条插入的value的值，这是由同一个连接保证的，不会被其他线程的其他数据库连接的操作所污染。

验证： 可以借助java.util.concurrent.CountDownLatch 创建几百个线程进行测试。

```


