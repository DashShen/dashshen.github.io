---
title: 乱谈python中的函数属性
date: 2016-11-18 21:36:35
tags:
---
在python中一切都是对象，包括函数。今天就来随便聊聊这个函数，聊聊这个函数的属性的问题。

首先，我们来先定义一个函数，然后调用`dir()`方法看看这个函数里面有什么属性和方法。

```python
>>> def func():
...     pass
...
>>> dir(func)
['__call__', '__class__', '__closure__', '__code__', '__defaults__','__delattr__', '__dict__', '__doc__', '__format__', '__get__', '__getattribute__', '__globals__', '__hash__', '__init__', '__module__', '__name__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'func_closure', 'func_code', 'func_defaults', 'func_dict', 'func_doc', 'func_globals', 'func_name']
>>>
```
咦，我怎么发现了两个大大的`__setattr__`和`__getattribute__`魔术方法呢(⊙o⊙)?。也就是说我可以这样子玩，`func.foo = bar`, 然后再这样子 `print func.foo`,打印出`bar`。我的天，感觉我好像在写`js`哇，既然这样子，那我这样子玩玩看´ψ(｀∇´)ψ\`。

```python
import functools


def set_foo(func):
    @functools.wraps(func)
    def decorator(*args, **kwargs):
        func.foo = 'bar'
        func(*args, **kwargs)
    return decorator

@set_foo
def func():
    print func.foo

func()
```

照理来说我们应该会看到`bar`, 但是这是什么鬼，为什么没有`foo`属性呢, 我都已经在装饰器里面给这个`func`加了`foo`的属性了。

```shell
$ python test.py                                                                                                         
Traceback (most recent call last):                                                                                       
  File "test.py", line 16, in <module>                                                                                   
    func()                                                                                                               
  File "test.py", line 14, in func                                                                                       
    print func.foo                                                                                                       
AttributeError: 'function' object has no attribute 'foo'
```

我怀疑这个在`def decorator(*args, **kwargs)`中的`func`，和在`def func()`中的`func`不会不是同一个本体吧。于是，我拿出`print id(func)`大法，看看这个两货本体究竟是谁 (\`皿´)。 

```python
def decorator(*args, **kwargs):
    print id(func)
...
def func():
    print id(func)
```

OMG, 这两货居然真的不是同一个本体。
```shell
$ python test.py
4538443128
4538443248
```

也就是说，我在装饰器中给`func`加的`foo`属性，并不是加在真正本体上了，那么真正的本体在去哪里了。最后我发现了，其实代码应该这么写。

```python
def decorator(*args, **kwargs):
    decorator.foo = 'bar'
...
```

然后我再执行，就发现没有问题了。

```shell
$ python test.py
bar
```
原来如此呀，原来`decorator`才是本体呀( ⊙ o ⊙ )！。至于为什么是这样解决这个问题的，我们可以去看一下python的**装饰器**, **闭包**等这些概念,然后去看一下解释器在解释执行的为什么会生成两个不同的函数对象。最后强烈建议，开发python的开发者，可以出一个方法，用于在函数内部表示函数的自身调用。