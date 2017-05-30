# python正则表达式学习笔记

`python` `正则表达式`


---

## 1. 正则表达式语法

### 1.1 正则表达式常用符号

http://www.cnblogs.com/huxi/archive/2010/07/04/1771073.html

|符号|含义|
|:--|:--|
|.|匹配任意字符，不包括换行符 \n |
|^|匹配字符串的开始|
|$|匹配字符串的结束或者字符串的结尾新的一行的开始|
|*|匹配前一个字符 0 次或者 无限次|
|+|匹配前一个字符 1 次或者 无限次|
|[]|匹配字符集合里的字符|
|\||或者，A|B 表示匹配 A 或者 B|
|?|匹配前一个字符 0 次或者 1 次|
|.*|贪心算法|
|.*?|非贪心算法|
|()|括号内的数据作为结果返回|
|\A|只匹配字符串的开始|
|\Z|只匹配字符串的结束|
|\b|只匹配出现在开始和结尾的空字符|
|\B|只匹配不出现在开始和结果的空字符|
|\d|只匹配数字，相当于[0-9]|
|\D|匹配非数字，相当于[^0-9]|
|\s|匹配任意的空字符，相当于[\t\n\r\f\v]|
|\S|匹配任意的非空字符，相当于[^\t\n\r\f\v]|
|\w|匹配任意的字符，相当于[a-zA-z0-9].如果是 rs.L 模式，则会匹配[0-9]和在当前环境被认定为字符的字符串|
|\W|匹配所有非 \\w 匹配的字符|
|\\\|匹配反斜杠 \\|


### 1.2 正则表达式匹配模式

|匹配模式|含义|
|:--|:--|
|re.I|不区分大小写进行匹配|
|re.L|Make \w, \W, \b, \B, dependent on the current locale|
|re.M|多行匹配模式，^ 匹配多行字符串的开始， $ 匹配多行字符串的结束|
|re.S|"."匹配所有的字符，包括换行符 \n|
|re.X|忽略空格和注释|
|re.U|Make \w, \W, \b, \B, dependent on the Unicode locale|

### 1.3 正则表达式常用方法

|方法名|含义|
|:--|:--|
|findall|匹配所有符合规律的内容，返回包含结果的列表|
|search|匹配并提取第一个符合规律的内容，返回一个正则表达式对象(object)|
|match|在字符的开始匹配正则表达式|
|sub|替换符合规律的内容，返回替换后的值|
|subn|替换符合规律的内容，返回替换后的值和替换的个数|
|split|使用正则对字符串进行分割|
|escape|用于将string中的正则表达式元字符如*/+/?等之前加上转义符再返回，在需要大量匹配元字符时有那么一点用|

## 2. 正则表达式实例

### 2.1 match(string[, pos[, endpos]]) | re.match(pattern, string[, flags]): 

这个方法将从`string`的`pos`下标处起尝试匹配pattern:
    
* 如果`pattern`结束时仍可匹配，则返回一个`Match`对象；
* 如果匹配过程中`pattern`无法匹配，或者匹配未结束就已到达`endpos`，则返回`None`。 

* `pos`和`endpos`的默认值分别为 0 和`len(string)`，`re.match()`无法指定这两个参数；
* 参数 `flags` 用于编译 `pattern` 时指定匹配模式。 

注意：这个方法并不是完全匹配。当`pattern`结束时若`string`还有剩余字符，仍然视为成功。想要完全匹配，可以在表达式末尾加上边界匹配符`'$'`。 

```
#!/usr/bin/python
# -*- coding: UTF-8 -*- 
 
import re
print(re.match('www', 'www.runoob.com').span())  # 在起始位置匹配
print(re.match('com', 'www.runoob.com'))         # 不在起始位置匹配
```
以上实例运行输出结果为：
```
(0, 3)
None
```


```
#!/usr/bin/python
import re
 
line = "Cats are smarter than dogs"
 
matchObj = re.match( r'(.*) are (.*?) .*', line, re.M|re.I)
 
if matchObj:
   print "matchObj.group() : ", matchObj.group()
   print "matchObj.group(1) : ", matchObj.group(1)
   print "matchObj.group(2) : ", matchObj.group(2)
else:
   print "No match!!"
```
以上实例的结果为：
```
matchObj.group() :  Cats are smarter than dogs
matchObj.group(1) :  Cats
matchObj.group(2) :  smarter
```


### 2.2 search(string[, pos[, endpos]]) | re.search(pattern, string[, flags]): 

这个方法用于查找字符串中可以匹配成功的子串。从`string`的`pos`下标处起尝试匹配`pattern`:
* 如果`pattern`结束时仍可匹配，则返回一个`Match`对象；
* 如果无法匹配，则将`pos`加 1 后重新尝试匹配；直到`pos=endpos` 时仍无法匹配则返回`None`。 
* `pos`和`endpos`的默认值分别为 0 和`len(string)`,`re.search()`无法指定这两个参数，
* 参数 `flags` 用于编译 `pattern` 时指定匹配模式。 

```
#!/usr/bin/python
# -*- coding: UTF-8 -*- 
 
import re
print(re.search('www', 'www.runoob.com').span())  # 在起始位置匹配
print(re.search('com', 'www.runoob.com').span())         # 不在起始位置匹配
```
以上实例运行输出结果为：

```
(0, 3)
(11, 14)
```


```
#!/usr/bin/python
import re
 
line = "Cats are smarter than dogs";
 
searchObj = re.search( r'(.*) are (.*?) .*', line, re.M|re.I)
 
if searchObj:
   print "searchObj.group() : ", searchObj.group()
   print "searchObj.group(1) : ", searchObj.group(1)
   print "searchObj.group(2) : ", searchObj.group(2)
else:
   print "Nothing found!!"
```
以上实例执行结果如下：
```
searchObj.group() :  Cats are smarter than dogs
searchObj.group(1) :  Cats
searchObj.group(2) :  smarter
```


### 2.3 findall(string[, pos[, endpos]]) | re.findall(pattern, string[, flags]): 

搜索string，以列表形式返回全部能匹配的子串。

```
import re
 
p = re.compile(r'\d+')
print p.findall('one1two2three3four4')
 
### output ###
# ['1', '2', '3', '4']
```

### 2.4 re.sub(pattern, repl, string, count=0, flags=0)

参数：
* `pattern` : 正则中的模式字符串。
* `repl` : 替换的字符串，也可为一个函数。
* `string` : 要被查找替换的原始字符串。
* `count` : 模式匹配后替换的最大次数，默认 0 表示替换所有的匹配。


```
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
import re
 
phone = "2004-959-559 # 这是一个国外电话号码"
 
# 删除字符串中的 Python注释 
num = re.sub(r'#.*$', "", phone)
print "电话号码是: ", num
 
# 删除非数字(-)的字符串 
num = re.sub(r'\D', "", phone)
print "电话号码是 : ", num
```

以上实例执行结果如下：

```
电话号码是:  2004-959-559 
电话号码是 :  2004959559
```