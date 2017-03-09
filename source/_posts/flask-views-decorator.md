---
title: flask-views-decorator
date: 2016-10-20 18:54:41
tags:
- flask
- decorator
---

## Flask中视图装饰器的应用

### 1、用于过滤未登录用户

<!-- more -->
	from functools import wraps
	from flask import g, request, redirect, url_for

	def login_required(f):
    		@wraps(f)
    		def decorated_function(*args, **kwargs):
        		if g.user is None:
            			return redirect(url_for('login', next=request.url))
        		return f(*args, **kwargs)
    		
		return decorated_function
	

	#注意route装饰器应该在最外层
	@app.route('/secret_page')
	@login_required
	def secret_page():
    		pass

### 2、用于缓存装饰器，缓存一些
