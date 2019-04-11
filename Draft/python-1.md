---
title: Python
date: 2018-10-24 13:03:25
tags:
---
1. 
python 命令可以查看 Python 的版本号和信息


2. 
Mac 使用 Homebrew 安装 Python 的命令：
brew install python3 

3. 
CPython 这个东西叫 Python 解释器
IPython 是基于 CPython 之上的一个交互式解释器
CPython用 >>> 作为提示符，而 IPython 用In [序号]: 作为提示符

4.
Python还允许用r''表示''内部的字符串默认不转义

5.
Python允许用'''...'''的格式表示多行内容
例子：
>>> print('''line1
... line2
... line3''')
line1
line2
line3

6. 
对于单个字符的编码，Python提供了ord()函数获取字符的整数表示，chr()函数把编码转换为对应的字符：

>>> ord('A')
65
>>> ord('中')
20013
>>> chr(66)
'B'
>>> chr(25991)
'文'