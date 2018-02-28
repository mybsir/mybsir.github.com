---
layout: post
title:  Python time 模块
categories: Python
description: Python time 模块
keywords: Python time 模块
---
# 时间模块
> **Info** 时间模块是python中最常用的模块之一了，关于时间，python中相关的模块有time和
> datetime


## Time 模块
一、在python中，通常有这几种方式来表示时间：
- 时间戳(timestamp)
通常来说，时间戳表示的是从1970年1月1日00:00:00开始按秒计算的偏移量。我们运行"type(time.time())",返回的是float类型
- 格式化的时间字符串
- 元组(struct_time)
struct_time元组共有9个元素：(年、月、日、时、分、秒，一年中第几周、一年中第几天、夏令时)

二、time模块常用函数
1、time.time() 返回当前时间的时间戳

```python
>>> import time
>>> time.time()
1472543906.061125
>>>
```

2、time.localtime([secs]) 将一个时间戳转换为当前时区的struct_time.secs参数为提供，则以当前时间为准

```python
>>> time.localtime()
time.struct_time(tm_year=2016, tm_mon=8, tm_mday=30, tm_hour=16, tm_min=1, tm_sec=13, tm_wday=1, tm_yday=243, tm_isdst=0)
>>> time.localtime(1372543906)
time.struct_time(tm_year=2013, tm_mon=6, tm_mday=30, tm_hour=6, tm_min=11, tm_sec=46, tm_wday=6, tm_yday=181, tm_isdst=0)
```

3、time.gmtime([secs]) 和time.localtime([secs])类似，gmtime()方法是将一个时间戳转换为UTC时区(0时区)的struct_time

```python
>>> time.gmtime()
time.struct_time(tm_year=2016, tm_mon=8, tm_mday=30, tm_hour=8, tm_min=4, tm_sec=34, tm_wday=1, tm_yday=243, tm_isdst=0)
```
4、time.mktime(t)    将一个struct_time转化为时间戳

```python
>>> time.mktime(time.localtime())
1472544385.0
```

5、time.sleep(secs)  线程推迟指定的时间运行。单位为秒

6、time.strftime(format[,t])     把一个代表时间的元组或者struct_time(如由time.localtime()和time.gmtime()返回)转化为格式化的时间字符串。如果t未指定，将传入time.localtime().如果元组中任何一个元素越界，ValueError的错误将会被抛出
```python
>>> time.strftime("%Y-%m-%d %H:%M:%S",time.localtime())
'2016-08-30 16:21:05'
>>> time.strftime("%j  %U  %w")
'243  35  2'
```

7、time.strptime(string[,format])    把一个格式化时间字符串转化为struct_time。实际上它和strftime()是逆操作

```python
>>> time.strptime("2016-08-30 16:20:32", "%Y-%m-%d %H:%M:%S")
time.struct_time(tm_year=2016, tm_mon=8, tm_mday=30, tm_hour=16, tm_min=20, tm_sec=32, tm_wday=1, tm_yday=243, tm_isdst=-1)
```

> **Tag**
> 时间格式化参数

```python
格式      含义
%a       本地(locale)简化星期名称
%A       本地完整星期名称
%b       本地简化月份名称
%B       本地完整月份名称
%c       本地相应的日期和时间表示
%d       一个月中的第几天(01-31)
%H       一天中的第几个小时(24小时制,00-23)
%I       第几个小时(12小时制,01-12)
%j       一年中的第几天(001-366)
%m       月份(01-12)
%M       分钟(00-59)
%p       本地AM或者PM的对应符
%S       秒(00-59)
%U       一年中的星期数(00-53星期天是一个星期的开始)。第一个星期天之前的所有天数都放在第0周
%w       一个星期中的第几天(0-6,0是星期天)
%W       和%U基本相同，不同的是%W以星期一为一个星期的开始
%x       本地相应日期
%X       本地相应时间
%y       去掉世纪的年份(00-99)
%Y       完整的年份
%Z       时区的名字(如果不存在为空字符)
%%       %字符
```
## 时间对象
- datetime

```python
>>> import datetime
>>> now = datetime.datetime.now()
>>> now
datetime.datetime(2016, 10, 27, 10, 17, 19, 794172)
>>> type(now)
<type 'datetime.datetime'>
```

- timestamp

```python
>>> import time
>>> time.time()
1477534712.387493
>>>
```
- time tuple

```python
>>> import time
>>> time.localtime()
time.struct_time(tm_year=2016, tm_mon=10, tm_mday=27, tm_hour=10, tm_min=19, tm_sec=9, tm_wday=3, tm_yday=301, tm_isdst=0)
```
- string

```python
>>> import datetime
>>> datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
'2015-01-12 23:13:08'
```
- date

```python
>>> import datetime
>>> datetime.datetime.now().date()
datetime.date(2015, 1, 12)
```
## datetime基本操作
- 1. 获取当前datetime

```python
>>> import datetime
>>> datetime.datetime.now()
datetime.datetime(2015, 1, 12, 23, 26, 24, 475680)
```

- 2. 获取当天date

```python
>>> datetime.date.today()
datetime.date(2015, 1, 12)
```
- 3. 获取明天/前N天

```python
明天
>>> datetime.date.today() + datetime.timedelta(days=1)
datetime.date(2015, 1, 13)
三天前
>>> datetime.datetime.now()
datetime.datetime(2015, 1, 12, 23, 38, 55, 492226)
>>> datetime.datetime.now() - datetime.timedelta(days=3)
datetime.datetime(2015, 1, 9, 23, 38, 57, 59363)
```
- 4. 获取当天开始和结束时间(00:00:00 23:59:59)

```python
>>> datetime.datetime.combine(datetime.date.today(), datetime.time.min)
datetime.datetime(2015, 1, 12, 0, 0)
>>> datetime.datetime.combine(datetime.date.today(), datetime.time.max)
datetime.datetime(2015, 1, 12, 23, 59, 59, 999999)
```

- 5. 获取两个datetime的时间差

```python
>>> (datetime.datetime(2015,1,13,12,0,0) - datetime.datetime.now()).total_seconds()
44747.768075
```
- 6. 获取本周/本月/上月最后一天


```python
本周
>>> today = datetime.date.today()
>>> today
datetime.date(2015, 1, 12)
>>> sunday = today + datetime.timedelta(6 - today.weekday())
>>> sunday
datetime.date(2015, 1, 18)
本月
>>> import calendar
>>> today = datetime.date.today()
>>> _, last_day_num = calendar.monthrange(today.year, today.month)
>>> last_day = datetime.date(today.year, today.month, last_day_num)
>>> last_day
datetime.date(2015, 1, 31)
获取上个月最后一天（可能跨年）
>>> import datetime
>>> today = datetime.date.today()
>>> first = datetime.date(day=1, month=today.month, year=today.year)
>>> lastMonth = first - datetime.timedelta(days=1)
```
## 关系转换
- datetime <=> string
  - datetime -> string
  ```python
  >>> import datetime
  >>> datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
  '2015-01-12 23:13:08'
  ```
  - string -> datetime
  ```python
  >>> import datetime
  >>> datetime.datetime.strptime("2014-12-31 18:20:10", "%Y-%m-%d %H:%M:%S")
  datetime.datetime(2014, 12, 31, 18, 20, 10)
  ```
- datetime <=> timetuple
  - datetime -> timetuple
  ```python
  >>> import datetime
  >>> datetime.datetime.now().timetuple()
  time.struct_time(tm_year=2015, tm_mon=1, tm_mday=12, tm_hour=23, tm_min=17, tm_sec=59, tm_wday=0, tm_yday=12, tm_isdst=-1)
  ```
  - timetuple -> datetime
  ```python
  timetuple => timestamp => datetime [看后面datetime<=>timestamp]
  ```
- datetime <=> date
  - datetime -> date
  ```python
  >>> import datetime
  >>> datetime.datetime.now().date()
  datetime.date(2015, 1, 12)
  ```
  - date -> datetime
  ```python
  >>> datetime.date.today()
  datetime.date(2015, 1, 12)
  >>> today = datetime.date.today()
  >>> datetime.datetime.combine(today, datetime.time())
  datetime.datetime(2015, 1, 12, 0, 0)
  >>> datetime.datetime.combine(today, datetime.time.min)
  datetime.datetime(2015, 1, 12, 0, 0)
  ```
- datetime <=> timestamp
  - datetime -> timestamp
  ```python
  >>> now = datetime.datetime.now()
  >>> timestamp = time.mktime(now.timetuple())
  >>> timestamp
  1421077403.0
  ```

  - timestamp -> datetime
  ```python
  >>> datetime.datetime.fromtimestamp(1421077403.0)
  datetime.datetime(2015, 1, 12, 23, 43, 23)
  ```


> **Tag**
> 三种时间格式转换

![PNG](../images/time.jpg)

部分转载http://www.wklken.me/posts/2015/03/03/python-base-datetime.html
