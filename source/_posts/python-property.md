---
title: python中的property装饰器用法
date: 2016-10-24 11:38:15
tags:
- python
- property
- 技巧集
---

@property装饰器负责把一个方法变成属性调用。
把一个getter方法变成属性，只需要加上@property就可以了，此时@property本身又创建了另一个装饰器@score.setter，负责把一个setter方法变成属性赋值。
<!-- more -->

    class Student(object):
	
	@property
	def score(self):
		return self._score

	@score.setter
	def score(self, value):
		if not isinstance(value, int):
			raise ValueError("score must be an integer!")
		if value < 0 or value > 100:
			raise ValueError("score must between 0~100")
		self._score = value
使用方法：

	>>> s = Student()
	>>> s.score = 30
	>>> s.score 
	30
	>>> s.score = 9999
	ValueError: score must between 0~100

这就意味着如果只定义getter方法，不定义setter方法就是一个只读属性

对于其它property应用使用dir(property查看)
