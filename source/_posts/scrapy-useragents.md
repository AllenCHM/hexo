---
title: scrapy-useragents
date: 2016-10-16 11:01:50
tags:
- scrapy
- user-agent
- 翻译
---

## scrapy-useragents


这是scrapy的一个中间件。允许对每一个request请求设置随机的User-Agent。

##配置
创建一个文件，每行一条user－agent。
可以通过nginx日志记录查找到user-agent。在nginx日志目录下面执行如下命令：
<!-- more -->    

    egrep -h -o '"[^"]+" "[^"]+"$' * | cut -d '"' -f 2 | sort -u > user-agents.txt
    	
    
可以通过USER_AGENTS_LIST_FILE来设置文件名

然后在middleware列表中添加它
    
    DOWNLOADER_MIDDLEWARES = {
    		#关闭默认的user agent中间件
    		'scrapy.contrib.downloadermiddleware.useragent.UserAgnetMiddleware': None,
    		'scrapy_useragents.middleware.UserAgentMiddleware': 400,
    	}
  -
