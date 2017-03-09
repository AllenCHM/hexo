---
title: flask
date: 2016-10-20 15:38:41
tags:
- flask
- 技巧集
---

1、使用SQLAlchemy时，db.session.commit()之后对象会自动添加id。或者使用db.session.refresh(f)。
暂时还不清楚refresh()是神马功能。

<!-- more -->

2、为flask的视图添加装饰器，必须使用warps.

warps作用之一就是改变函数的__name__。

如果自定义装饰器不使用warps，那么使用装饰器后原本不同函数名字都变成了相同的名字，在flask中会造成函数名冲突。

	AssertionError: View function mapping is overwriting an existing endpoint function

正确写法：

    
    def login_required(func):

    	@wraps(func)
    	def wrap(*args, **kwargs):
        	courier_id = request.headers.get('X-ID')
        	token = request.headers.get('X-TOKEN')
        	if verify_login_token(int(courier_id), token):
            		return func(*args, **kwargs)
        	else:
            		return response(ResponseCode.UNAUTHORIZED)

    	return wrap



3、flask 在函数中添加异常捕捉失败

无法捕捉Exception传个的e，代码如下：

	def check_error(func):
	@wraps(func)
	def _check(*args, **kwargs):
		try:
			return func(*args, **kwargs)
		except Exception, e:
			return make_response(jsonify({
				u'error':1,
				u'message':u'有参数传错了。。。呼叫后台',
			}))
	
	return _check
