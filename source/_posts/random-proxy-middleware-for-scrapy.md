---
title: random proxy middleware for scrapy(scracpy的随记代理中间件)
date: 2016-10-15 22:03:35
tags:
- scrapy
- proxy middleware
- 翻译
---

# Random proxy middleware for scrapy(scrapy的随机代理中间件)


在处理scrapy的requests时，使用随机的从代理ip列表中抽取一个使用， 避免ip被禁用，提高爬取速度。

<!-- more -->

可以通过 [http://www.hidemyass.com](http://www.hidemyass.com)获取代理ip列表


### 安装
快速方法： 

	    
	pip install scrapy_proxies
	

或者下载源代码并运行：
    
	python setup.py install
    
### 设置
    
    #失败后重试次数
    RETRY_TIMES = 10
    #重试的错误码
    RETRY_HTTP_CODES = [500, 503, 504, 400, 403, 404, 408]
    DOWNLOADER_MIDDLEWARES = {
    	'scrapy.downloadermiddlewares.retry.RetryMiddleware':90,
    	'scrapy_proxies.RandomProxy':100,
    	'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware':110,
    }
    PROXY_LIST = '/path/to/proxy/list.text'
    
    
#### 注意
对于scrapy老版本（1.0.0以前的版本）。需要使用scrapy.contrib.downloadermiddleware.retry.RetryMiddleware 和scrapy.contrib.downloadermiddleware.httpproxy.HttpProxyMiddleware  中间件代替


### 爬虫使用
如果你设置了不过滤重复请求 dont_filter = True.在每一个回调中，返回得到目标页面，检查页面logo或者其它标志性元素。

    if not hxs.select('//get/site/logo'):
    	yield Request(url=response.url, dont_filter=True)







