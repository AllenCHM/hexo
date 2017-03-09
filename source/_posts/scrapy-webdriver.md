---
title: scrapy_webdriver
date: 2016-10-15 22:07:29
tags:
- scrapy
- selenium
- scrapy-webdrive
- 翻译
---

## scrapy-webdriver

使用selenium webdriver爬取

没有很好的测试，可能会有很多bug。
<!-- more -->



### 安装

没有上传到pypi上，可以使用下面命令安装：
    
    pip installhttps://github.com/sosign/scrapy-webdriver/archive/master.zip
    
或者编写setup.py文件如下：
   
    setup(
    	install_requires=[
    			'scrapy_webdriver',
    			....,
    		],
    	dependency_links = [
    			'https://github.com/sosign/scrapy-webdriver/archive/master.zip'
    		],
    		....,
    )
    
    
    
    
### 配置
在你的scrapy项目中添加如下设置：
    
    DOWNLOAD_HANDLERS = {
    	'http':'scrapy_webdriver.download.WebdriverDownloadHandler',
	'https':'scrapy_webdriver.download.WebdriverDownloadHandler',
    }
    
    
    SPIDER_MIDDLEWARES = {
        'scrapy_webdriver.middlewares.WebdrverSpiderMiddleware':543,
    }
    
    WEBDRIVER_BROWSER = 'PhantomJS'
    #或者其它selenium.webdriver中的驱动
    ＃或者你自定义的浏览器驱动'your_package.CustomWebdriverClass'
    #或者用一个实际类代替字符串
    
    
    ＃浏览器驱动的配置
    WEBDRIVER_OPTIONS = {
    	'service_args':['--debug=true', '--load-images=false', '--webdriver-loglevel=debug']
    }
    
    
### 使用

为了使用webdriver来下载，需要使用类提供的scrapy_webdriver.http.WebdriverRequest代替以前的scrapy.Request

    from scrapy_webdrvier.http import WebdriverRequest
        yield WebdriverRequest('http://www.baidu.com')
    
可用的参数包括：method, body, headers, cookies 
    
    
    
