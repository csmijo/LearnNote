# Python2 编码格式笔记

`python`

---

## 1. Python2 的 encode 和 decode

Python 中，我们使用 `decode()` 和 `encode()` 方法进行解码和编码，但是经常会在文件读取文本出现各种错误，十分的崩溃的。这主要是对 Python 的编码格式理解的不足导致的。

在 Python 中使用 `Unicode` 类型作为编码的基础，也就是说：

```
    decode         encode
  str ---> unicode ---> str 
```

通过中间人 `unicode` 编码格式进行其他的各种格式之间的转换

#### 1.1 文件读取实例

以从文件读取数据为例，假如该文件编码格式为 `UCS-2 little Endian`(Fiddler 保存文件后文件的编码格式，坑了好久)， 希望读取文件的内容并进行进一步的处理。

按照常规的方式：
```
f = open('test.txt','r')
s = f.readline()
print s
```

结果发现控制台输出了一些数据，但是格式完全不正确。。

第一想法就觉得是文件编码的问题，修改了文件的默认编码方式为 `UTF-8` 后再次运行上面的脚本，没有问题，那么就进行转码喽。

使用 `chardet` 检测到文件的编码为 `UTF-16`，所以就尝试进行了如下的操作。后面介绍 `chardet`.

```
f = open('test.txt','r')
str = f.readline()
ss = str.decode('utf-16').encode('utf-8')
print ss
```

结果出现了这样的错误：
```
Traceback (most recent call last):
  ss = str.decode('utf-16').encode('utf-8')
  File "D:\program\python2.7.10\lib\encodings\utf_16.py", line 16, in decode
    return codecs.utf_16_decode(input, errors, True)
UnicodeDecodeError: 'utf16' codec can't decode byte 0x0a in position 1580: truncated data
```

于是就饶了一个很大的圈子，怎么转 `UCS-2` 到 `UTF-8` 等等，最后尝试使用 Python 提供的 `codecs` 进行文件的读取，这个包在 `open()` 函数可以指定编码的类型，如下：
```
import codecs

f = codecs.open('text.txt','r',"UTF-16")
str = f.readline()
print str
print type(str)
```

控制台打印信息为：
```
test

<type 'unicode'>
```

ok ，转成 `unicode` 就好办了，然后就可以使用 `encode()` 进行其他编码格式的转换了。

## 2 chardet 使用

`chardet` 模块可以检测字符编码，应该说是类似问题的终极解决。

### 2.1 安装 `chardet`
 
1. 方法一：可以使用 `pip install chardet` 来进行安装
2. 方法二：
    * 可以在官网 https://pypi.python.org/pypi/chardet 下载 `chardet_3.0.3.tar.gz` 包
    * 解压到 `Python` 安装目录的 `site-packages` 目录下
    * 进入到 `chardet_3.0.3.tar.gz` 的解压目录，运行 `python setup.py install` 即可运行安装
3. 遇到的坑：
    
在使用上述两种方法安装的时候，都出现了这样的错误：
```
pip install fails with “connection error: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed ”
```

应该是 `https` 证书认证的问题，于是在 `stackoverflow` 上找到了这样的解决方案：

```
pip install --index-url=http://pypi.python.org/simple/ --trusted-host pypi.python.org  chardet
```

尝试后安装成功

### 2.2 使用 chardet 检测文件的编码格式

代码如下：
```
import chardet

with open(path) as f:
    str = f.read()
    print chardet.detect(str)
```

控制台输出如下：

```
{'confidence': 1.0, 'language': '', 'encoding': 'UTF-16'}
```

这是一个字典，可以看到当前文件的编码方式为 `UTF-16`，这样就可以在 `codecs.open()` 方法中使用了。


---

[参考文献]

* [Python编码格式说明及转码函数encode和decode的使用](http://blog.csdn.net/mfcing/article/details/43058591)
* [pip install fails with “connection error: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:598)”](http://stackoverflow.com/questions/25981703/pip-install-fails-with-connection-error-ssl-certificate-verify-failed-certi)
* [Python | 多种编码文件（中文）乱码问题解决](http://jingyan.baidu.com/article/425e69e6e111a1be15fc1609.html)
* [Python_批量修改文件的编码格式](http://blog.csdn.net/vagerant/article/details/50350171)
