---
layout: post
title: Python基础——语法基础+函数
category: python
tags: [python, language]
---
{% include JB/setup %}

## 语法基础

### 内置函数

Python内置函数，官方英文文档参见[Built-in Functions](https://docs.python.org/2.7/library/functions.html).

| 函数签名 | 函数说明 |
|:-----|:----|
| abs(*x*) | 返回*x*的绝对值，*x*可以是普通整数、long整数或浮点数。当*x*是复数时，返回*x*的模。|
| all(*iterable*) | 当*iterable*为空，或所有元素均为true时返回`True`，否则返回`False`。 |
| any(*iterable*) | 若*iterable*中任意元素为true，则返回`True`，否则返回`False`。当*iterable*为空时，返回false。|
| basestring() | 抽象类型，是`str`和`unicode`的父类。该类型不可调用或实例化，但可以用来检测一个对象是否为`str`或`unicode`实例。`isinstance(obj, basestring)`等同于`isinstance(obj, (str, unicode))`。|
| bin(*x*) | 将整数转换成二进制字符串，返回结果是有效的python二进制表达式。如果*x*不是`int`对象，那么必须定义返回整数的`__index__()`函数，才可使用。|
| *class* bool([*x*]) | 返回一个布尔值，`True`或`False`。*x*被标准真值测试程序进行转换。如果*x*是false或被省略，则返回`False`，否则返回`True`。`bool`是`int`的子类，但是不能继续派生。|
| *class* bytearray([*source*[, *encoding*[, *errors*]]]) | 返回一个新的字节数组。`bytearray`类是一个*mutable sequence*的整数（0 <= x < 256）序列。拥有大部分mutable sequence的普通函数，参见[Mutable Sequence Types](https://docs.python.org/2.7/library/stdtypes.html#typesseq-mutable)，以及`str`的大部分方法，参加[String Methods](https://docs.python.org/2.7/library/stdtypes.html#string-methods)。 <br><br> 可选参数*source*有以下几种方式用于数组的初始化： <br>  *source*是*unicode*对象，则必须同时给定参数*encoding*， 则函数会调用`unicode.encode()`方法将其转换为字节数组;<br>  *source*是*integer*对象，按照其值创建该长度的数组，并用null字节初始化；<br>  *source*是兼容*buffer*接口的对象，该对象会被当作一个只读的buffer来初始化数组；<br>  *source*是一个*iterable*对象，其迭代元素必须属于[0, 255]的范围；<br><br>所有参数都忽略时，会创建一个长度为0的数组。|
| callable(*object*) | 如果*object*是可调用的返回`True`，否则返回`False`。注意，类是callable的（调用返回一个新对象）；当类对象中包含`__call__()`时，该实例才时callable的。|
| chr(*i*) | 以整数*i*为ASCII码，返回只包含一个字符的字符串。该函数是`ord()`函数的反过程。参数必须在[0, 255]范围内，否则会出现`ValueError`异常。<br><br>查看相关[`uuichr()`](https://docs.python.org/2.7/library/functions.html#unichr). |
| classmethod(*function*) | 为*function*返回一个类函数。<br><br>一个类函数隐式的接收一个类作为第一个参数，就像一个实例方法接收一个实例作为第一参数一样。声明一个类方法时，需要使用`@classmethod`方法，详见[Function definitions](https://docs.python.org/2.7/reference/compound_stmts.html#function)。<br><br>类方法可以在类上调用（例如`C.f()`），或者在一个实例上调用（例如`C().f()`）。调用时，它的实例会被忽略。如果在一个派生类上调用类方法，派生类的类对象会被传递给隐式第一个参数。<br><br>类方法与C++、Java中的静态方法不同。如果你需要，查看[staticmethod()](https://docs.python.org/2.7/library/functions.html#staticmethod)。<br><br>关于类方法的更多信息，查看[The standard type hierarchy](https://docs.python.org/2.7/reference/datamodel.html#types)。|
| cmp(*x*, *y*) | 比较*x*和*y*两个对象，根据比较结果返回一个整数。若x<y返回负数，0若x==y，正数若x>y。|
| compile(*source*, *filename*, *mode*[, *flags*[, *dont_inherit*]]) | 编译*source*成为代码或AST对象。|
| class complex([*real*[, *imag*]]) | 返回一个值为*real*+*imag*\*1j，或者将一个字符串或正数转换为复数。如果第一个参数是字符串，它将会被解析为一个复数，而不能传入第二个参数。第二个参数不能是字符串。两个参数都可以是任意数字，包括复数。如果*imag*被省略了，则默认为0，该函数则作为一个数字转换函数，类似`int()`, `long()`和`float()`。如果两个参数都被省略，则返回0j. |
| delattr(*object*, *name*) | `setattr()`方法的相关函数。参数为一个对象和一个字符串。字符串必须是该对象属性之一的名字。该函数将删除指定的属性，如果该对象允许的话。|
| *class* dict(*\*\*kwarg*) <br>*class* dict(*mapping*, *\*\*kwarg*) *class* dict(*iterable*, *\*\*kwarg*) | 创建一个新词典。这个`dict`对象就是词典类。关于该类，查看`dict` API和[Mapping Types -- dict](https://docs.python.org/2.7/library/stdtypes.html#typesmapping)。 <br><br>关于其他容器，查看内置的`list`、`set`和`tuple`类，以及`collections`模块。 |
|


### 内置数据类型

`int`

### 流程语句

### 表达式

## 函数

### 函数定义

### 嵌套函数与闭包

### 变量作用域域声明周期

### yield与Generator用法

### 内置函数
