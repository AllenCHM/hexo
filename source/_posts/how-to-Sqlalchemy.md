---
title: Sqlalchemy使用方法
date: 2016-10-19 10:37:24
tags:
- Sqlalchemy
- 技巧集
---
## Sqlalchemy 用法


**in用法**

    SELECT id, name FROM user WHERE id in (123,456)

<!-- more -->
sqlalchemy解决方案：

方案1、

    session.query(MyUserClass).filter(MyUserClass.id.in_((123, 234))).all()

方案2、

    session.execute(
    select(
        [MyUserTable.c.id, MyUserTable.c.name],
        MyUserTable.c.id.in_((123, 234))
    )).fetchall()
    
方案3、

    myList = [123, 234]
    select = sqlalchemy.sql.select([user_table.c.id, user_table.c.name], user_table.c.id.in_(myList))
    result = con.execute(select)
    for row in result:
        process(row)
