---
title: 数据库学习之SQL拾遗
date: 2017-03-12 22:54:41
tags:
- SQL
- 数据库学习
categories: 数据库学习
---

## 数据检索

 **行0** 检索出来的第一行wei行0而不是行1。因此,LIMIT 1,1将检索出第二行而不是第一行
  

**ORDER BY** 默认排序为升序(**ASC**)(从A到Z)。使用**DESC**关键字,为降序


检索价格在5和10之间的所有产品
```
SELECT prod_name, prod_price
FROM products
WHERE prod_price BETWEEN 5 AND 10;
```


NULL 无值(no value),它与字段包含0、空字符串或仅仅包含空格不同

范围没有价格(空prod_price字段,不是价格为0)的所有产品
```
SELECT prod_name
FROM products
WHERE prod_price IS NULL;
```


检索供应商1002和1003制造的所有产品
```
SELECT prod_name, prod_price
FROM products
WHERE vend_id in (1002, 1003)
ORDER BY prod_name;
```

* 在使用长的合法选项清单时,IN操作符的语法更清楚且更直观

* 在使用IN时,计算的次序更容易管理

* IN操作符一般比OR操作符清单执行更快

* IN可以包含其他SELECT语句,使得能够更动态的建立WHERE子句


**通配符**(wildcard)用来匹配值的一部分的特殊字符

**搜索模式**(search pattern)由字面值、通配符或两者组合构成的搜索条件

**%** 表示**任何字符出现任意次数**

```
SELECT prod_id, prod_name
FROM products
WHERE prod_name LIKE 'jet%';
```

|prod_id|prod_name|
|---|---|
|JP1000|JetPACK 1000|
|JP10002|JetPack 1002|


根据MySQL的配置方式,搜索可以是区分大小写的。

**_**只匹配单个字符

使用通配符技巧

* 不要过度使用通配符。如果其他操作符能达到相同的目的,应该使用其他操作符。

* 在确实需要使用通配符时,出发绝对有必要,否则不要把它们用在搜索模式的开始处。把通配符置于搜索
模式的开始处,搜索起来是最慢的。

* 仔细注意通配符的位置。如果放错地方,可能不会返回想要的数据。


'''
SELECT prod_name
FROM products
WHERE prod_name REGEXP '.000'
ORDER BY prod_name;
'''


LIKE匹配整个列。如果被匹配的文本在列值中出现,LIKE将不会找到它,相应的行也不被返回(
除非使用通配符)。而REGEXP在列值内进行匹配,如果被匹配的文本在列值中出现,REGEXP将会找到
它,相应的行将被返回

REGEXP匹配整个列值(从而起与LIKE相同的作用)可以配合^和$定位符(anchor)即可。

正则表达式匹配不区分大小写。为区分大小写,可以使用BINARY关键字。如
```
WHERE prod_name REGEXP BINARY 'JetPack .000'
```







## 联结表

各表中通过某些常用的值(即关系设计中的***关系(Relational)***)互相关联

**外键(foreign key)**为某个表中的一列,它包含另一个表的主键值,定义了两个表之间的关系。


**可伸缩性(scale)** 能够适应不断增加的工作量而不失败。设计良好的数据库或应用程序称之为可伸缩性好(scale well0)


	SELECT vend_name, prod_name, prod_price
	FROM vendors, products
	WHERE vendors.vend_id = products.vend_id
	ORDER BY vend_name, prod_name;
	
***完全限定列名*** 在引用的列可能出现二义性，必须使用完全限定列名**（用一个点分割的表名和列名）**	


***笛卡尔积cartesian product*** 检索出来的行的数目是第一个表中的行数乘以第二个表中的行数

	例如：
	SELECT vend_name, prod_name, prod_price
	FROM vendors, products
	ORDER BY vend_name, prod_name;
	
	 
等值联结（equijoin）,基于两个表之间的相等测试。也称为内部连接

	SELECT vend_name, prod_name, prod_price
	FROM vendors INNER JOIN products
	ON vendors.vend_id = products.vend_id;
		
		
多表联结

显示编号为20005的订单中的物品
	
	SELECT prod_name, vend_name, prod_price, quantity
	FROM orderitems, products, vendors
	WHERE products.vend_id = vendors.vend_id
		AND orderitems.prod_id = products.prod_id
		AND order_num = 200005;
		
		
返回订购产品tnt2的客户列表。

子查询方式

	SELECT cust_name, cust_contact
	FROM customers
	WHERE cust_id IN (
			SELECT cust_id
			FROM orders
			WHERE order_num IN (
				SELECT order_num
				FROM orderitems
				WHERE prod_id = 'TNT2'	
				)
			);
			  
  
联结查询

	SELECT cust_name, cust_contact
	FROM customers, orders, orderitems
	WHERE customers.cust_id = orders.cust_id
		AND orderitems.order_num = orders.order_num
		AND prod_id = 'TNT2';
		  
		  
    show databases;  -- 列出所有数据库
    show create database db_name; -- 查看数据库的DDL
    show tables;  -- 列出默认数据库的所有表
    show tables from db_name; -- 列出制定数据库的所有表
    show table status; -- 查看表的描述性信息
    show table status from db_name;
    show create table tb1_name; -- 查看表的DDL
    show columns from tb1_name; -- 查看列信息
    show index from tb1_name; -- 查看索引信息
    show columns from domaininfo like 's%'; -- s表示开头
    show columns from domaininfo where field = 'sysdomain';
    SHOW STATUS; -- 用于显示广泛的服务器状态信息;
    SHOW GRANTS; -- 用来显示授予用户(所有用户或特定用户)的安全权限;
    SHOW ERRORS; -- 显示服务器错误信息
    SHOW WARNINGS; -- 显示服务器警告信息
    

## SHOW 语句 
 
SHOW COLUMNS FROM customers

返回中包括字段名、数据类型、是否允许NULL、键信息、默认值以及其他信息。也可以
使用*DESCRIBE customers*查询

|Field|Type|Null|Key|Default|Extra|
|---|---|---|---|---|---|
|cust_id|int(11)|NO|PRI|NULL|auto-increment|
|cust_name|char(50)|NO||||
|cust_address|char(50)|YES||NULL||
|cust_city|char(50)|YES||NULL||
|cust_state|char(50)|YES||NULL||
|cust_zip|char(50)|YES||NULL||
|cust_country|char(50)|YES||NULL||



## 数据分组

返回供应商1003提供的产品数目

```
    SELECT COUNT(*) AS num_prods
    FROM products
    WHERE vend_id = 1003;
```
输出

| num_prods |
|---|
| 7 |


分组允许把数据分为多个逻辑组,以便能对每个组进行聚集计算。
```
SELECT vend_id, COUNT(*) AS num_prods
FROM products
GROUP BY vend_id;
```

输出

|vend_id|num_prods|
|---|---|
|1001|3|
|1002|2|
|1003|3|
|1004|4|

因为使用GROUP BY,就不必指定要计算和估值的每个组了,系统会自动完成。

* GROUP BY 子句可以包含任意数目的列。这使得能对分组进行嵌套,为数据分组提供更细致的控制

* 如果在GROUP BY子句中嵌套了分组,数据将在最后规定的分组上进行汇总。换句话说,在建立分组时,
指定的所有列都一起计算(所以不能从个别的列取回数据)

* GROUP BY子句中列出的每个列都必须是检索列或有效的表达式(但不能是聚集函数)。如果在SELECT中使用
表达式,则必须在GROUP BY 子句中指定相同的表达式。不能使用别名。

* 除聚集计算语句外,SELECT语句中的每个列都必须在GROUP BY子句中给出

* 如果分组列中具有NULL值,则NULL将作为一个分组返回。如果列中有多行NULL值,它们将分为一组

* GROUP BY子句必须出现在WHERE子句之后,ORDER BY子句之前。


使用WITH ROLLUP关键字,可以得到每个分组以及每个分组汇总级别(针对每个分组)的值
```
SELECT vend_id, COUNT(*) AS num_prods
FROM products
GROUP BY vend_id WITH ROLLUP;
```


HAVING支持所有WHERE操作符,唯一的差别是WHERE过滤行,而HAVING过滤分组。
```
SELECT cust_id, COUNT(*) AS orders
FROM orders
GROUP BY cust_id
HAVING COUNT(*) >= 2;
```

|cust_id|orders|
|---|---|
|1001|2|


WHERE在数据分组前进行过滤,HAVING在数据分组后进行过滤。意味着,WHERE排除的行不包括在分组中。这可能会改变计算
值,从而影响HAVING子句中基于这些值过滤掉的分组。

```
SELECT vend_id, COUNT(*) AS num_prods
FROM products
WHERE prod_price >= 10
GROUP BY vend_id
HAVING COUNT(*) >=2;
```

|vend_id|num_prods|
|---|---|
|1003|4|
|1005|2|


分组和排序

|ORDER BY|GROUP BY|
|---|---|
|排序产生的输出|分组行。但输出可能不是分组的顺序|
|任意列都可以使用(甚至非选择的列也可以使用)|只可能使用选择列或表达式列,而且必须使用每个选择列表达式|
|不一定需要|如果于聚集函数一起使用列(或表达式),则必须使用|

```
SELECT order_num, SUM(quantity*item_price) AS ordertotal
FROM orderitems
GROUP BY order_num
HAVING SUM(quantity*item_price) >= 50
ORDER BY ordertotal;
```

|order_num|ordertotal|
|20006|55.00|
|20008|125.00|
|20005|149.00|


