---
title: 【SQLAlchemy 1.1 文档翻译】之 概述
date: 2017-03-10 18:54:41
tags:
- SQLAlchemy
- 文档翻译
categories: SQLAlchemy 1.1 文档翻译
---

## 概述

SQLAlchemy SQL工具包和对象关系映射器是一套用于处理数据库的Python综合工具。它具有
几个不同的功能区域，可以单独使用或组合使用。其主要组件如下所示，将有依赖性的组件组合为一层：

![SQLAlchemy 组件](/uploads/sqlalchemy_overview_0.png)

上面，SQLAlchemy的两个最重要的部分是对象关系映射器和SQL表达式语言。SQL表达式
可以独立于ORM使用。当使用ORM时，SQL表达式语言仍然是面向公众的API的一部分，
因为它在对象关系配置和查询中使用。

## 文档概述

文档分为三个部分：SQLAlchemy ORM， SQLAlchemy Core和Dialects。

在SQLAlchemy ORM文档中，引入并详细描述了对象关系映射器。新用户应该从对象关系教程开始。
如果要使用为您自动构建的高级SQL，以及管理Python对象，请继续阅读本教程。

在SQLAlchemy Core文档中，SQLAlchemy的SQL和数据库集成和描述服务的广度都被记录在案，
其核心是SQL表达式语言。SQL表达式语言是一个独立于ORM包的工具包，它可用于构造可操作的
SQL表达式，可以通过编程方式构造，修改和执行，返回类似游标的结果集。
与ORM的以域为中心的使用模式相反，表达式语言提供了以模式为中心的使用范例。
新用户应该从这里开始使用SQL表达式语言教程。SQLAlchemy引擎，连接和池服务也在
SQLAlchemy Core文档中进行了描述。

在Dialects文档中，提供了所有支持数据库和DBAPI的参考文档。


## 代码的例子

工作代码示例（主要涉及ORM）包含在SQLAlchemy中。所有包括的示例应用程序的描述在[ORM示例](http://docs.sqlalchemy.org/en/latest/orm/examples.html)。

wiki上还有各种各样涉及核心SQLAlchemy的ORM结构例子。参见 [Theatrum Chemicum](https://bitbucket.org/zzzeek/sqlalchemy/wiki/UsageRecipes)。


## 安装指南

### 支持的平台

SQLAlchemy已针对以下平台进行测试：

* cPython从2.6版本开始，通过2.xx系列
* cPython版本3，在所有3.xx系列
* Pypy 2.1或更高

`在0.9版本中更改： Python 2.6现在是支持的
最低Python版本。`

当前不支持的平台包括Jython和IronPython。Jython已经被过去支持，
并且在将来的版本中也可以被支持，这取决于Jython本身的状态。

### 支持的安装方法

SQLAlchemy安装是通过基于setuptools的标准Python方法，
通过setup.py直接引用或通过使用pip或其他与setuptools兼容的方法。

`在版本1.1中更改： setuptools现在是setup.py文件所需的; 不再支持plain distutils安装。`

### 通过PIP安装

当pip可用时，可以从PYPI下载，并通过以下步骤中安装：

```
pip install SQLAlchemy
```

这个命令将从PYPI库下载最新发布的SQLAlchemy版本，
并将其安装到系统中。

为了安装最新的预发布版本，例如1.1.0b1，pip要求使用--pre标志：

```
pip install -pre SQLAlchemy
```

在上面，如果最新版本是预发布，它将被安装，而不是最新发布的版本。

### 安装使用的setup.py

否则，您可以使用setup.py脚本从分发版安装：

```
python setup.py install
```

### 安装C扩展

SQLAlchemy包括C扩展，为处理结果集提供额外的速度提升。
扩展在2.xx和3.xx系列的cPython上都受支持。

`setup.py`将自动构建扩展，如果检测到适当的平台。
如果C扩展的构建由于缺少编译器或其他问题而失败，则设置过程将输出警告消息，
并在完成时重新运行不带C扩展的构建，报告最终状态。

要运行构建/安装而不尝试编译C扩展，可以指定DISABLE_SQLALCHEMY_CEXT
环境变量。这种情况是用于特殊的测试环境，或者在通常的“重建”机制不能克服的罕见的兼容性/
构建问题的情况下：

```
export DISABLE_SQLALCHEMY_CEXT=1; python setup.py install
```

`
在版本1.1中更改：旧--without-cextensions标志已从安装程序中删除，因为它依赖于
setuptools的已弃用的功能。
`

### 在Python 3上安装

SQLAlchemy直接在Python 2或Python 3上运行，并且可以安装在任何环境中，无需任何调整
或代码转换。

### 安装数据库API

SQLAlchemy设计用于为特定数据库构建的DBAPI实现，并支持最流行的数据库。Dialects中的各个
数据库部分枚举每个数据库的可用DBAPI，包括外部链接。

### 检查已安装的SQLAlchemy版本

本文档涵盖SQLAlchemy版本1.1。如果你在一个已经安装了SQLAlchemy的系统上工作，
请从Python提示符中检查版本，如下所示：

```
>>> import sqlalchemy
>>> sqlalchemy.__version__ # doctest: +SKIP
1.1.0
```
