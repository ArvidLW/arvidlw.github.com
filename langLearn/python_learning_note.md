# python学习笔记

***

```python
#_*_ coding:UTF-8  //代码中使用中文注释
# coding=gbk
!# /usr/bin/python 跨平台注释
```

顶部的:# -*- coding: utf-8 -*-目前看来有三个作用。

如果代码中有中文注释，就需要此声明
比较高级的编辑器（比如我的emacs），会根据头部声明，将此作为代码文件的格式。
程序会通过头部声明，解码初始化 u”人生苦短”，这样的unicode对象，（所以头部声明和代码的存储格式要一致）

# Python文件设置编码

***

 Python文件设置编码utf-8 （文件前面加上 #encoding=utf-8)

input(u"哈哈");其中u为指定编码

pass占位符，替代一系列属性和方法组成的主体
python 中self，相当于this，只不过python中的self需要明确写出
对象创建不需要new

类的方法必须有一个self参数，但是方法被调用时，不用传递这个参数。

self.direction #公有的实例属性
self.__direction #私有的实例属性

访问私有属性，_类名__私有属性名 #但这种方法一般用于开发阶段，而最好的方法是定义相关方法
eg:fly=Fly();print fly._Fly__direction

数据属性可以不被预先定义，被初次使用的时候则被创建赋值。
```python
__bases__
__dict__
__moudle__
__doc__
__name__
```

@staticmethod指令来定义静态方法
or
staticmethod(方法名)来定义静态方法

实例方法：只能通过实例来调用
类方法，用@classmethod指令来声明

类方法隐含调用参数是类，而实例方法隐含调用参数是类的实例p194

python内置方法p195
```python
__init__(self,...) #在创建完对象之后被调用
__del__(self) 
__new__(self,*args,**ked) #创建对象时被调用，该方法中包含__init__,如果被重写后那么可以不包含__init__
__str__(self) #在使用print语句时被调用
.......
```
python是一种完全面向对象的动态语言，可以动态的增加方法

继承 class class_name(fatherclass_name)

python类命名空间
```python
if __name__=='__main__'
```
新式类：从内置类型创建的子类，object为新式类的超类
```python
__slots__ #一个类变量，包含一个类中的实例属性
```
新式类都要继承object

局部命名空间
全局命名空间
内建命名空间

捕获异常的方式：三种
捕获所有异常
```python
try:
	a=b
except Exception,data:
	print Exception,data
```
使用traceback查看异常
import traceback
```python
try:
	a=b
except:
	print traceback.print_exc()
```
采用sys模块回溯最后的异常
```python
import sys
try:
	a=b
except:
	info=sys.exc_info()
	print info
```
urlretrieve获取远程文件保存到本地

# Socket

***

## 客户端通信过程

Socket对象
bind()方法
connect()方法
send()方法
recv()方法
close()方法

## 服务端套接字

Socket对象
bind()方法
listen()方法
accept()方法
send()方法
recv()方法
close()方法

windows不支持fork

GUI工具包，流行wxWidget(wxPython)与PyQt,Tkinter为标准包，而PyQt目前最流行

wxPython开发工具很多，分别为WxGlade,WxFormBuilder,Boa-constructor.开发工具可以生成python代码

# IDLE

***

## 快捷键

alt+n 下一条命令

alt+p 上一条命令

## 保存不了

包含中文则保存不了，或者编码有问题。

如果包含中文，则必须加入编码格式即可：

```python
#_*_ coding:UTF-8 
```

清屏：ctrl+L

指定utf-8乱码，是由于windows默认为GBK编码，解释器也默认为gbk，所以用gbk去解释utf-8会出现乱码。但由于是指定了头，所以会出现部分乱码。

```python
import sys
sys.getdefaultencoding() #得到系统默认编码
```

# 语言特性

***

字符串*8 会将字符串连在一起8次

缩进用tab键

tab键：缩进与提示

## read

## 输出

用print输出可以将\n\r直接在屏幕上用换行表示出来





