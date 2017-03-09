---
title: Flask-cookies使用教程
date: 2016-10-14 14:27:37
tags: 
- Flask-cookies 
- Flask技巧
---

## Cookie介绍
cookie以text文件的形式 被存放在用户的电脑上。目的是记住和跟踪用户使用数据，提高访问体验，做网站统计。

<!-- more -->
一个Request 对象包含了cookie属性。这是一个由cookie变量和对应的值组成的字典对象，客户端发送。除此以外cookie也可以保存它的过期时间，路径和站点域名。

在Flask中，cookies在response中设置。在一个视图函数中使用make_response()函数生成response对象返回。然后使用set_cookie()函数在response对象中存放cookie。

读取cookie很简单，使用get()方法从request.cookies属性中读取。

在下面的Flask应用中，打开'/'页面的到一个简单的表单。
```
@app.route('/')
def index():
    return render_template('index.html')
```
HTML页面包含一个text输入框
```
<html>
    <body>
        <form action = "/setcookie" method="post">
            <p><h3>Enter userID</h3></p>
            <p><input type ='text' name='nm' /></p>
            <p><input type='submit' value='login' /></p>
        </form>
    </body>
</html>
```

这个form表单提交到'/setcookie'链接， 相关的视图函数将设置一个cookie名称为userID， 重新加载其它页面。
```
@app.route('/setcookie', methods=['post', 'get'])
def setcookie():
    if request.method == "POST":
        user = request.form[u'nm']
        resp = make_response(render_template('readcookie.html'))
        resp.set_cookie('userID', user)
        return resp
```

'readcookie.html'包含了一个超级链接，指向getcookie()视图函数。getcookie()读取和显示cookie值。
```
@app.route('/getcookie')
def getcookie():
    name = request.cookies.get('userID')
    return '<h1> welcome' + name + '</h1>'
```
