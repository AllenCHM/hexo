---
title: flask错误集合
date: 2016-10-19 00:24:09
tags:
- flask
- 错误集
---

## Flask错误集


#### 使用Flask-Migrate初始化数据库出错
<!-- more -->    

    python manage.py db init

**错误内容**: alembic.util.CommandError: Directory migrations already exists

**原因**: 提前创建了migrations文件夹

**解决方法**: 删除migrations文件夹即可


#### 使用Flask-Migrate更新数据库不成功

解决方法：
先运行：

    python manage.py db migrate -m "inital migration"

在运行：

    python manage.py db upgrade


#### alembic.util.exc.CommandError: Can't locate revision identified by 'b863c498c6e7'

解决办法：

删除数据库中的alembic_version表，重新init

