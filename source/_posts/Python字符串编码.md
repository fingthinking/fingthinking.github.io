---
title: Python字符串编码及文件输出
date: 2017-08-10 15:25:08
tags:
	- Python
---
> ***注意：部分浏览器会拦截MathJax生成数学公式，如出现无法显示公式，请允许浏览器加载脚本***

**以下内容为Python2.7.x，不适用于Python3.x**

## unicode、str与编码方式
### 1.unicode与str  
在Python2.x中的字符串"表示"使用的是str,Python3.x中默认的则是unicode，unicode与str的区别在于使用单个字符还是双字符存储。

```python

# -*- coding=utf-8 -*-
str_1 = u"我爱中国hehe"
str_2 = "我爱中国hehe"
print type(str_1), str_1, len(str_1), str_1[0] # <type 'unicode'> 我爱中国hehe 8 我 
print type(str_2), str_2, len(str_2), str_2[0] # <type 'str'> 我爱中国hehe 16 �
```
在上面的代码中可以看出:  

- 当使用u前缀的时候，是以unicode方式存储的字符串，而字符串长度就是字符的长度；
- 当不使用u前缀的时候，是以str方式存储的字符串，而字符串的长度是字节的长度；

综上，当使用汉字的时候如果需要统计字数，则需要使用unicode字符串，但需要**`注意`**的是使用unicode编码的时候print可以正常打印到控制台，但是如果需要重定向到文件中，则需要将unicode转码成str.

### 2. 字符串与编码格式，decode与encode
在[unicode与str](#1-unicode与str)一节中的`# -*- coding=utf-8 -*-`定义了使用str方式存储汉字的时候的编码，即默认在内存中的存储方式将使用utf-8编码，如“我爱中国”将每个汉字占用两个字节，但"hehe"会占用1个字节，因此才有了`len(str_2) = 16`;

Python字符串编码异常是一个比较常见的异常，如:

```python
UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-3: ordinal not in range(128)
```
接上文，给出Demo,再进行分析:

```python
str_3 = str_1.encode("utf-8")
print type(str_3), str_3, str_2 == str_3    # <type 'str'> 我爱中国hehe True

str_4 = str_1.encode("gbk")
print type(str_4), str_4, len(str_4)        # <type 'str'> �Ұ��й�hehe 12
print str_3 == str_4, str_2 == str_4        # False False

str_5 = str_4.decode("gbk")
print type(str_5), str_5, str_5 == str_1    #   <type 'unicode'> 我爱中国hehe True

str_6 = str_2.decode("utf-8")
print type(str_5), str_6, str_6 == str_1    # <type 'unicode'> 我爱中国hehe True
```
在Python中字符串编码的两种格式为unicode,str; 其中`unicode -> str`是encode操作，而`str -> unicode`则需要decode;

其实在Python的内部字符串的表示都是str类型的，或者说如果Python有类型的话，应该是char[]数组类型，每个char占1个字节，而str和unicode是Python中的对象表现形式。如果对Python的struct模块有所了解的话,就会明白在其他语言中存在byte类型的，但是Python是通过str来表示byte数组，因此str的本质是Python中的byte[]类型。

#### encode: 编码  
> 将Python对象编码成内存字节数据byte[]即str;

#### decode: 解码
> 将内存字节数据byte[]解码成Python对象;

有了上面的知识，就可以解释代码中的结果了:

- 第2行: unicode通过encode方法编码成utf-8的str对象,因此str\_2 == str\_3才会成立，因为str_2在内存中使用的是utf-8格式存储;
- 第4行: unicode通过encode方法编码成gbk的str对象,因此str\_4 != str\_2,同时gbk编码是一个汉字占用两个字节，因此长度为12,参考[UTF-8编码规则](http://www.cnblogs.com/chenwenbiao/archive/2011/08/11/2134503.html)
- 第6行: gbk格式的str对象通过decode方法再次解码成为unicode对象
- 第8行: utf8格式的str对象通过decode方法解码称谓unicode对象

## print与unicode

上面的代码如果通过命令`python demo.py`直接执行, 不会出现任何问题，但是如果将输出重定向`python demo.py > demo.dat`,则会报`UnicodeEncodeError`的错误。

原因: print直接输出unicode实质上是进行了encode(coding)的操作,其中coding与输出对象的编码设置有关，但重定向到文件中时不知道文件编码，则会尝试encode(defaultcoding即"ascii")操作，因此会报错。
解决方案：

### 1. 手动编码

```python
print u"".decode('utf-8')
```
使用该方式，明确将输出对象编码为utf-8的str(上文提到实质为byte[]),就可以将文本输出为utf-8格式文本；
### 2. 改变Python默认编码

```python
import sys
reload(sys)
sys.setdefaultencoding('utf8')
```
使用上面的代码可以改变Python内部的默认编码方式，此时默认encode的时候将自动encode("utf-8"),此时输出到文件将会是utf-8格式文本，同理设置setdefaultencoding('gbk'), 则输出的文本为gbk格式；

**注意：** reload(sys)这一句是必须写的，因为Python加载的时候会加载site.py文件，但是在site.py中554行有如下代码：

```python
if hasattr(sys, "setdefaultencoding"):
    del sys.setdefaultencoding
```
会将setdefaultencoding给删除，因此如果不reload的话是找不到`sys.setdefaultencoding `方法的。


> 本人小白，能力有限，如有错误，欢迎评论拍砖！！！
