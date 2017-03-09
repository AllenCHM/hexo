---
title: sqlalchemy错误集
date: 2016-10-27 14:55:16
tags:
- 错误集
- sqlalchemy
---

1、sqlachemy中批量删除的问题

db.session.query(Post).filter(Post.id.in_(ids)).delete()

报错：

	sqlalchemy.exc.InvalidRequestError
<!-- more -->
原因：

删除记录时，默认会尝试删除 session 中符合条件的对象，而 in 操作估计还不支持，于是就出错了。解决办法就是删除时不进行同步，然后再让 session 里的所有实体都过期：

解决办法：

session.query(User).filter(User.id.in_((1, 2, 3))).delete(synchronize_session=False)
session.commit() # or session.expire_all()


update操作也有同样的参数，如果后面立刻提交了，那么加上synchronize_session=False参数会更快
