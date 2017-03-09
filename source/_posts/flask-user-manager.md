---
title: flask开发过程中的用户认证
date: 2016-10-24 11:14:47
tags:
- flask
- 用户认证
- 技巧集
---

常用的第三方包
Flask-Login:管理已登录用户的用户会话
Werkzeug: 计算密码散列值并进行核对
itsdangerous: 生成并核对加密安全令牌
Flask-Mail:发送与认证相关的电子邮件
Flask-Bootstrap:Html模板
Flask-WTF：Web表单

<!-- more -->
**如何确保数据库中的密码安全**

若想保证数据库中用户密码的安全，关键在于不能存储密码本身，而要存储密码的散列值。

计算密码散列值的函数接收密码作为输入，使用一种或多种加密算法转换密码，最终得到一个和原始密码没有关系的字符序列。

核对密码时，密码散列值可替代原始密码，因为计算散列值的函数是可复现的：只要输入一样，结果就一样。


**使用Werkzeug实现密码散列**

generate_password_hash(password, method=pbkdf2:sha1, salt_length=8)
以字符串形式输出密码的散列值。method和salt_length的默认值就能满足大多数需求。

check_password_hash(hash, password)
比对密码散列值与用户输入的密码。返回值为True表明密码正确


	from werkzeug.security import generate_password_hash, check_password_hash
	
	class User(db.Model):
		#。。。
		password_hash = db.Column(db.String(128))

		@property
		def password(self):
			raise AttributeError('password is not a readable attribute')

		@password.setter
		def password(self, password):
			self.password_hash = generate_password_hash(password)

		def verify_password(self, password):
			return check_password_hash(self.password_hash, password)


**使用Flask-login认证用户**
flask-login 是专门用来管理用户认证系统中的认证状态，且不依赖特定的认证机制。

如果想使用flask-login, User模型必须实现几个方法，如下：
is_authenticated():	如果用户已经登录，必须返回True,否则返回False
is_active():		如果允许用户登录，必须返回true，否则返回false，如果要禁用账户，可以返回false
is_anonymous()		对普通用户必须返回False
get_id()		必须返回用户的唯一标识符，使用Unicode编码字符串


也可以使用flask-login中的UserMixin类，该类包含了这些方法的默认实现，且能满足大多数雪球。

	from flask.ext.login import UserMixin

	class User(UserMixin, db.Model):
		__tablename__ = u'users'
		email = db.Column(db.String(64), unique=True, index=True)

Flask-login在程序的工厂函数中初始化：

	from flask.ext.login import LoginManager

	login_manager = LoginManager()
	login_manager.session_protection = 'strong'
	login_manager.login_view = 'auth.login'

	def create_app(config_name):
		login_manager.init_app(app)

LoginManager对象的session_protection属性可以设为None， 'basic'或'strong'，以提供不同等级的安全等级防止用户会话遭修改。设为'strong'时，Flask-Login会记录客户端IP地址和浏览器的用户代理信息，如果发现异动就登出用户。login_view属性设置登录页面的端点。

最后，flask-login要求程序实现一个回调函数，使用指定的标识符加载用户。
app/models.py:	加载用户的回调函数

	from . import login_manager
	
	@login_manager.user_loader
	def load_user(user_id):
		return User.query.get(int(user_id))

加载用户的回调函数接收以Unicode字符串形式表示的用户标识符。如果能找到用户，这个函数必须返回用户对象；否则应该返回None。

未完待续
