---
layout: post
title:  Python system 模块
categories: Python
description: Python system 模块
keywords: Python system 模块
---
# SYS模块
> **Info** sys模块包括了一组非常实用的服务，内涵很多函数方法和变量，用来处理Python运行时配置以及资源，从而可以与当前程序之外的系统环境交互，如：Python解释器

### sys.argv
这个是sys最常用的函数，用来在程序外部传递参数

示例脚本sys.py如下
```python
import sys

print sys.argv[0]
print sys.argv[1]
print sys.argv[2]
print sys.argv[3]
```

来看一下效果
```json
python sys.py argv1 argv2 argv3
sys.py
argv1
argv2
argv3
```
argv[0]代表的是脚本文件本身，argv[1]代表传入的第一个参数，argv[2]代表传入的第二个参数...

### sys.platform()
获取当前平台信息，Linux、windows or Mac

|system|platform|
|-|-|
|Linux(kernel 2.X & 3.X)|linux2|
|Windows|win32|
|Windwos/Cygwin|cygwin|
|Mac OS X|darwin|
|OS/2|os2|
### sys.exit(n)
执行到主程序的末尾时，解释器会自动退出。但是如果需要中途退出程序，可以调用sys.exit函数，它带有一个可选的整数参数返回给调用它的程序。这意味着你可以在主程序中捕获对sys.exit的调用。
> **Note** 0是正常退出，其他为不正常，可抛异常事件供捕获
