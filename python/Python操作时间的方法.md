# python 操作时间和日期的方法

`python`

---


## 1. 操作时间和日期的方法

http://www.jb51.net/article/47957.htm

时间相关的都要用到 `datetime` 和 `time` 这两个标准库模块

### 1. 将字符串的时间转成时间戳

```
>>> import time
>>> a = "2017-5-17 18:25:39"
>>> timeArray = time.strptime(a,"%Y-%m-%d %H:%M:%S")
>>> timeStamp = int(time.mktime(timeArray))
>>> timeStamp
1495016739
```

### 2. 将字符串转成 datetime 格式

```
>>> import datetime
>>> t = "2017-5-18-14-34-24"
>>> datetime.datetime.strptime(t,"%Y-%m-%d-%H-%M-%S")
datetime.datetime(2017, 5, 18, 14, 34, 24)
```

### 3. 格式更改

如 a = "2017-5-17 18:25:39" ，想改为 a = "2017/5/17 18:25:39"

**先转换为时间数组，然后再转换为其他格式**

```
>>> import time
>>> a = "2017-5-17 18:25:39"
>>> timeArray = time.strptime(a,"%Y-%m-%d %H:%M:%S")
>>> otherStyle = time.strftime("%Y/%m/%d %H:%M:%S",timeArray)
>>> otherStyle
'2017/05/17 18:25:39'
```

### 4. 时间戳转换为指定格式日期

#### 方法一： 利用localtime()转换为时间数组,然后格式化为需要的格式

```
>>> import time
>>> timeStamp = 1495016739
>>> timeArray = time.localtime(timeStamp)
>>> otherStyle = time.strftime("%Y/%m/%d %H:%M:%S",timeArray)
>>> otherStyle
'2017/05/17 18:25:39'
```

#### 方法二：使用 datetime

```
>>> import datetime
>>> timeStamp = 1495016739
>>> dateArray = datetime.datetime.utcfromtimestamp(timeStamp)
>>> otherStyle = dataArray.strftime("%Y-%m-%d %H:%M:%S")
>>> otherStyle
'2017-05-17 10:25:39'
```

### 5. 获取当前时间并转换为指定日期格式

#### 方法一： 使用 time

```
>>> import time
>>> now = int(time.time())
>>> timeArray = time.localtime(now)
>>> otherStyle = time.strftime("%Y-%m-%d %H:%M:%S",timeArray)
>>> otherStyle
'2017-05-18 10:55:28'
```

#### 方法二： 使用 datetime

```
>>> import datetime
>>> now = datetime.datetime.now()
>>> otherStyle = now.strftime("%Y-%m-%d %H:%M:%S")
>>> otherStyle
'2017-05-18 10:58:10'
```

### 6. 获取三天前的时间的方法

```
>>> import time
>>> import datetime
>>> threeDayAgo = (datetime.datetime.now()-datetime.timedelta(days=3))
>>> otherStyle = threeDayAgo.strftime("%Y-%m-%d %H:%M:%S")
>>> otherStyle
'2017-05-15 11:07:11'
```
**注意：**  `timedelta()` 方法的参数有： `days`,`hours`,`seconds`,`microseconds`

####  格式转换

```
>>> import time
>>> import datetime
>>> now = datetime.datetime.now()  # 获取当前时间，type 为： datetime.datetime
>>> now
datetime.datetime(2017, 5, 18, 11, 13, 25, 877000)
>>> type(now)
<type 'datetime.datetime'>


>>> now.timetuple()  # 获取 datetime.datetime 的 timetuple()
time.struct_time(tm_year=2017, tm_mon=5, tm_mday=18, tm_hour=11, tm_min=13, tm_sec=25, tm_wday=3, tm_yday=138, tm_isdst=-1)
>>> timeStamp = int(time.mktime(now.timetuple()))  # 使用 time 转换为时间戳
>>> timeStamp
1495077205

>>> nowTimetuple = time.localtime(timeStamp)  # 使用 time.localtime() 又可以将时间戳转成 timetuple
>>> nowTimetuple
time.struct_time(tm_year=2017, tm_mon=5, tm_mday=18, tm_hour=11, tm_min=13, tm_sec=25, tm_wday=3, tm_yday=138, tm_isdst=0)
>>>
```

### 7. 给定时间戳，计算该时间的几天前时间

先转换成 `datetime.datetime` 格式，然后再使用 `timedelta()` 方法计算

```
>>> import time
>>> import datetime
>>> now = int(time.time())
>>> dateArray = datetime.datetime.utcfromtimestamp(now)
>>> dateArray
datetime.datetime(2017, 5, 18, 3, 24, 5)
>>> threeDayAgo = dateArray - datetime.timedelta(days=3)
>>> threeDayAgo
datetime.datetime(2017, 5, 15, 3, 24, 5)
```

### 8. 用 Python 计算昨天和明天的日期

首先使用 `datetime.date.today()` 获取到今天的日期，然后使用 `datetime.datetime.timedelta()` 计算日期

```
>>> import datetime
>>> today = datetime.date.today()   # 获取到今天的日期
>>> today
datetime.date(2017, 5, 18)
>>> yesterday = today - datetime.timedelta(days=1)  # 计算昨天的日期
>>> yesterday
datetime.date(2017, 5, 17)
>>> tomorrow = today + datetime.timedelta(days=1)  # 计算明天的日期
>>> tomorrow
datetime.date(2017, 5, 19)
```

### 9. 使用 Python 打印当前时间

#### 方法一： 使用 time
```
>>> import time
>>> time.strftime("%Y-%m-%d %H:%M:%S")
'2017-05-18 11:52:06'
```

#### 方法二： 使用 detetime

```
>>> import datetime
>>> datetime.datetime.now().timetuple()
time.struct_time(tm_year=2017, tm_mon=5, tm_mday=18, tm_hour=11, tm_min=53, tm_sec=9, tm_wday=3, tm_yday=138, tm_isdst=-1)
>>> datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
'2017-05-18 11:54:00'
```
### 附：日期和时间的格式化参数

```
%a 星期几的简写
%A 星期几的全称
%b 月分的简写
%B 月份的全称
%c 标准的日期的时间串
%C 年份的后两位数字
%d 十进制表示的每月的第几天
%D 月/天/年
%e 在两字符域中，十进制表示的每月的第几天
%F 年-月-日
%g 年份的后两位数字，使用基于周的年
%G 年分，使用基于周的年
%h 简写的月份名
%H 24小时制的小时
%I 12小时制的小时
%j 十进制表示的每年的第几天
%m 十进制表示的月份
%M 十时制表示的分钟数
%n 新行符
%p 本地的AM或PM的等价显示
%r 12小时的时间
%R 显示小时和分钟：hh:mm
%S 十进制的秒数
%t 水平制表符
%T 显示时分秒：hh:mm:ss
%u 每周的第几天，星期一为第一天 （值从0到6，星期一为0）
%U 第年的第几周，把星期日做为第一天（值从0到53）
%V 每年的第几周，使用基于周的年
%w 十进制表示的星期几（值从0到6，星期天为0）
%W 每年的第几周，把星期一做为第一天（值从0到53）
%x 标准的日期串
%X 标准的时间串
%y 不带世纪的十进制年份（值从0到99）
%Y 带世纪部分的十制年份
%z，%Z 时区名称，如果不能得到时区名称则返回空字符。
%% 百分号
```