---
layout: post
title: Python函数中访问非local变量
category: python
tags: [python, scope]
---
{% include JB/setup %}

Python的作用域解析遵循LEGB原则，在查找变量name的时候按照local -> parent local -> ... -> global -> built-in的方式逐层查找。

- L, Local — Names assigned in any way within a function (def or lambda)), and not declared global in that function.
- E, Enclosing-function locals — Name in the local scope of any and all statically enclosing functions (def or lambda), from inner to outer.
- G, Global (module) — Names assigned at the top-level of a module file, or by executing a global statement in a def within the file.
- B, Built-in (Python) — Names preassigned in the built-in names module : open,range,SyntaxError,...

[ref](http://stackoverflow.com/questions/291978/short-description-of-python-scoping-rules)

## 函数中访问上层scope变量

在函数中，即local namespace，访问其上层的local namespace或global变量时，通过LEGB规则查找即可访问。如下面例子所示。

```py
def foo():
    print 'in foo method'
    a = 1
    s = 'abc'
    l = [1, 2]

    def bar():
        print 'in bar method'
        print a
        print s
        print l

    bar()	       

if __name__ == '__main__':
    foo()
```

其结果为:

```
in foo method
in bar method
1
abc
[1, 2]
```

## 函数中对上层scope变量修改

但是，想要在local namespace内对非local的变量修改，却不是这么简单。如下面例子所示，试图对上层scope的变量赋值时，却弹出了UnboundLocalError的一场。

```py
def foo():
    print 'in foo method'
    a = 1
    s = 'abc'
    l = [1, 2]

    def bar():
        print 'in bar method'
        print a
        print s
        print l

        a = 2
        s = 'xyz'
        l += 3

    bar()	       

if __name__ == '__main__':
    foo()
```

运行结果为：

```
in foo methoTraceback (most recent call last):d
in bar method

  File "C:\Users\shln5899\PycharmProjects\python exercise\modify nonlocal variables.py", line 20, in <module>
    foo()
  File "C:\Users\shln5899\PycharmProjects\python exercise\modify nonlocal variables.py", line 17, in foo
    bar()          
  File "C:\Users\shln5899\PycharmProjects\python exercise\modify nonlocal variables.py", line 9, in bar
    print a
UnboundLocalError: local variable 'a' referenced before assignment
```

结果显示变量a在使用前未定义。变量s、l均是相同的情况，说明与变量的类型无关。出现这种情况的原因是，Python中**对非local变量的赋值（assignment）会生成一个新的局部变量**，而不是对该变量进行值修改。

Python这么做的原因，大概是因为在函数内访问上层scope或global的变量是一种不好的方式，修改变量的值就更糟了。这种直接访问、修改的方式会造成代码逻辑的不清晰，这种隐藏在代码中的操作会将程序的逻辑打乱，使得其可读性变差。

## 非赋值修改

值得注意的是，虽然Python会为具有**赋值**操作的上层scope变量创建新的local变量，但是却没有禁止所有的修改操作。

```py
def foo():
    print 'in foo method'
    a = 1
    s = 'abc'
    l = [1, 2]

    def bar():
        print 'in bar method'
        # print a
        # print s
        print l

        # a = 2
        # s = 'xyz'
        l[1] = 3

    bar()         
    print l 

if __name__ == '__main__':
    foo()
```

这个例子中，通过下标的方式对list中的一个元素进行修改（不可添加新元素），其运行结果如下。可见，bar函数成功的对上层scope的l变量的第二个元素进行了修改。这也说明，Python的这一局部变量访问限制，仅仅限制了赋值操作，仍旧可以通过一些方式修改变量。当然，这种访问方式也是不推荐。

```
in foo method
in bar method
[1, 2]
[1, 3]
```

## import相关

运行下面的示例程序。

```py
import math

def foo():
    a = math.PI
    import math
    print math.sin(a)

if __name__ == '__main__':
    foo()
```

其结果如下。可见，import操作对math这一模块的加载导致了对模块变量math的修改操作，导致其在local中创建了一个新的math变量。

```
Traceback (most recent call last):
  File "C:\Users\shln5899\PycharmProjects\python exercise\modify nonlocal variables - import.py", line 9, in <module>
    foo()
  File "C:\Users\shln5899\PycharmProjects\python exercise\modify nonlocal variables - import.py", line 4, in foo
    a = math.PI
UnboundLocalError: local variable 'math' referenced before assignment
```
