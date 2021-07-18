---
description: 描述器实战应用
---

# property 装饰器的描述器实现

## 引言

在 Python 官方文档中的 [描述器使用指南](https://docs.python.org/zh-cn/3.9/howto/descriptor.html) 一篇中，给出了常用装饰器 `property` 的 Python 代码模拟实现，且这个实现是基于描述器原理的。 在本文中，作者将利用这段代码，向你展示 **描述器的实际运行原理** 。

## 装饰器基本内容

首先我们回忆一下装饰器的有关内容。

> **装饰器本质是个返回函数的函数，表现为数学概念中的复合函数** $$\big(g \circ f\big)(x) \Rightarrow g\big(f(x)\big) $$

下面我们用伪代码再补充一些容易产生疑问的情况。

### **多层装饰器的含义**

```python
@dec2
@dec1
def func(arg1, arg2, ...):
    pass
```

↑↑↑ 等价于 ↓↓↓

```python
func = dec2(dec1(func))
```

### **带参装饰器的含义**

```python
@decomaker(argA, argB, ...)
def func(arg1, arg2, ...):
    pass
```

↑↑↑ 等价于 ↓↓↓

```python
func = decomaker(argA, argB, ...)(func)
```



类装饰器和函数装饰器是一致的，只相当于将 func 代表的函数换做 cls 代表的类罢了，故这里不再多做涉及。

如果对于装饰器还有问题的话建议阅读一下官方 PEP ，可以直接使用 Google 翻译成中文，可读性还是可以接受的。 

[PEP 318 -- Decorators for Functions and Methods](https://www.python.org/dev/peps/pep-0318/) [PEP 3129 – Class Decorators](https://www.python.org/dev/peps/pep-3129/)

## 描述器基本内容

这里推荐我针对描述器撰写的上一篇博文

[描述器学习指南](descriptor.md)

## 官方模拟的源码

我们首先看一下用描述器模拟实现装饰器 `property` 的源代码。

_（这里大概看一下就好，我们后文再详细分析）_

```python
class Property:
    "Emulate PyProperty_Type() in Objects/descrobject.c"

    def __init__(self, fget=None, fset=None, fdel=None, doc=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel
        if doc is None and fget is not None:
            doc = fget.__doc__
        self.__doc__ = doc

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        if self.fget is None:
            raise AttributeError("unreadable attribute")
        return self.fget(obj)

    def __set__(self, obj, value):
        if self.fset is None:
            raise AttributeError("can't set attribute")
        self.fset(obj, value)

    def __delete__(self, obj):
        if self.fdel is None:
            raise AttributeError("can't delete attribute")
        self.fdel(obj)

    def getter(self, fget):
        return type(self)(fget, self.fset, self.fdel, self.__doc__)

    def setter(self, fset):
        return type(self)(self.fget, fset, self.fdel, self.__doc__)

    def deleter(self, fdel):
        return type(self)(self.fget, self.fset, fdel, self.__doc__)
```

## 代码分析

### 改写测试用代码

其实作者比较喜欢的方式是利用 VS Code 在代码段顶部就加上断点，然后逐句执行查看代码的具体运行位置和变量值。   
但这种方式难以通过文字向大家展示，所以我换了个变通的方式，在每个关键环节都加一个 `print()` 输出相关内容，这样我们看代码的命令行反馈就可以了。

 更改后的测试代码如下：

```python
class Property(object):
    "Emulate PyProperty_Type() in Objects/descrobject.c"

    print('body of Property')

    def __init__(self, fget=None, fset=None, fdel=None, doc=None):
        print(f'Property.__init__() {self=}')
        self.fget = fget
        self.fset = fset
        self.fdel = fdel
        if doc is None and fget is not None:
            doc = fget.__doc__
        self.__doc__ = doc

    def __get__(self, obj, objtype=None):
        print(f'Property.__get__() {self=} {obj=} {objtype=}')
        if obj is None:
            return self
        if self.fget is None:
            raise AttributeError("unreadable attribute")
        return self.fget(obj)

    def __set__(self, obj, value):
        print(f'Property.__set__() {self=} {obj=}')
        if self.fset is None:
            raise AttributeError("can't set attribute")
        self.fset(obj, value)

    def __delete__(self, obj):
        print(f'Property.__delete__() {self=} {obj=}')
        if self.fdel is None:
            raise AttributeError("can't delete attribute")
        self.fdel(obj)

    def getter(self, fget):
        print(f'Property.getter() {self=}')
        return type(self)(fget, self.fset, self.fdel, self.__doc__)

    def setter(self, fset):
        print(f'Property.setter() {self=}')
        return type(self)(self.fget, fset, self.fdel, self.__doc__)

    def deleter(self, fdel):
        print(f'Property.deleter() {self=}')
        return type(self)(self.fget, self.fset, fdel, self.__doc__)


class A(object):

    print('body of A')

    def __init__(self):
        print(f'A.__init__() {self=}')
        self._x=0
        pass 

    @Property       # 事实上这里就是getter哦
    def x(self):
        print(f'A.x.fget() {self=}')
        return self._x

    @x.setter
    def x(self, value):
        print(f'A.x.fset() {self=}')
        self._x=value
    @x.deleter
    def x(self):
        print(f'A.x.fdelete() {self=}')
        del self._x

    # setter和deleter的函数命名并不会有具体含义，所以按照官方的示例，直接与getter同名即可

print('\n----- before a1 = A() -----')
a1 = A()
print('\n----- before   a1.x   -----')
a1.x
print('\n-----before  a1.x = 1 -----')
a1.x = 1
print('\n-----before del a1.x  -----')
del a1.x

print()

print('\n----- before a2 = A() -----')
a2 = A()
print('\n----- before   a2.x   -----')
a2.x
print('\n-----before  a2.x = 1 -----')
a2.x = 1
print('\n-----before del a2.x  -----')
del a2.x
```

### 全部输出

运行以上代码得到的输出是：

```text
body of Property
body of A
Property.__init__() self=<__main__.Property object at 0x000002A7BBA310A0>
Property.setter() self=<__main__.Property object at 0x000002A7BBA310A0>
Property.__init__() self=<__main__.Property object at 0x000002A7BBA31100>
Property.deleter() self=<__main__.Property object at 0x000002A7BBA31100>
Property.__init__() self=<__main__.Property object at 0x000002A7BBA310A0>

----- before a1 = A() -----
A.__init__() self=<__main__.A object at 0x000002A7BBA31100>

----- before   a1.x   -----
Property.__get__() self=<__main__.Property object at 0x000002A7BBA310A0> obj=<__main__.A object at 0x000002A7BBA31100> objtype=<class '__main__.A'>
A.x.fget() self=<__main__.A object at 0x000002A7BBA31100>

-----before  a1.x = 1 -----
Property.__set__() self=<__main__.Property object at 0x000002A7BBA310A0> obj=<__main__.A object at 0x000002A7BBA31100>
A.x.fset() self=<__main__.A object at 0x000002A7BBA31100>

-----before del a1.x  -----
Property.__delete__() self=<__main__.Property object at 0x000002A7BBA310A0> obj=<__main__.A object at 0x000002A7BBA31100>
A.x.fdelete() self=<__main__.A object at 0x000002A7BBA31100>


----- before a2 = A() -----
A.__init__() self=<__main__.A object at 0x000002A7BBA31160>

----- before   a2.x   -----
Property.__get__() self=<__main__.Property object at 0x000002A7BBA310A0> obj=<__main__.A object at 0x000002A7BBA31160> objtype=<class '__main__.A'>
A.x.fget() self=<__main__.A object at 0x000002A7BBA31160>

-----before  a2.x = 1 -----
Property.__set__() self=<__main__.Property object at 0x000002A7BBA310A0> obj=<__main__.A object at 0x000002A7BBA31160>
A.x.fset() self=<__main__.A object at 0x000002A7BBA31160>

-----before del a2.x  -----
Property.__delete__() self=<__main__.Property object at 0x000002A7BBA310A0> obj=<__main__.A object at 0x000002A7BBA31160>
A.x.fdelete() self=<__main__.A object at 0x000002A7BBA31160>
```

**由于这其中涉及的内容较多，我们拆开来做分析。**

### 初始化部分

```text
body of Property        # Python 解释器在发现类创建操作后，对类进行初始化
body of A               # 类初始化首先执行的就是类体中的代码，而类体中的 @Property def x(self): .... 相当于被转化为了 x=Property(x)  注意后面这个 x 是原本的函数
                        # 如果不理解的话建议回头看一下最上面的 装饰器基本内容
Property.__init__() self=<__main__.Property object at 0x000002A7BBA310A0>        # 继而 Property 对象初始化
Property.setter() self=<__main__.Property object at 0x000002A7BBA310A0>          # 这里对应的是 @x.setter
Property.__init__() self=<__main__.Property object at 0x000002A7BBA31100>        # 注意这里是在原本 Property 的对象基础上，又创造了一个新的Property对象（具体参见Property的setter方法）
Property.deleter() self=<__main__.Property object at 0x000002A7BBA31100>         # 对应 @x.deleter
Property.__init__() self=<__main__.Property object at 0x000002A7BBA310A0>        # 和前面的 setter 一样，这里也是返回了新的 Property 对象
                                                                                 # 自这之后， A 类中和 x 挂勾的 Property 就确定下来了
```

### 对象 a1 创建及属性操作部分

```text
----- before a1 = A() -----
A.__init__() self=<__main__.A object at 0x000002A7BBA31100>        # a1 实例对象创建的初始化

----- before   a1.x   -----
Property.__get__() self=<__main__.Property object at 0x000002A7BBA310A0> obj=<__main__.A object at 0x000002A7BBA31100> objtype=<class '__main__.A'>
  # 上面这行是实例对象 a1 想要获取属性 x 的值，Property 作为一个描述器，它的 __get__ 方法被调用
A.x.fget() self=<__main__.A object at 0x000002A7BBA31100>        # 在 Property 的 __get__ 方法中，转调用了 通过 @Property 设置的方法

## setter 和 deleter 的调用方式和 getter 一样，没有本质差别

-----before  a1.x = 1 -----
Property.__set__() self=<__main__.Property object at 0x000002A7BBA310A0> obj=<__main__.A object at 0x000002A7BBA31100>
A.x.fset() self=<__main__.A object at 0x000002A7BBA31100>

-----before del a1.x  -----
Property.__delete__() self=<__main__.Property object at 0x000002A7BBA310A0> obj=<__main__.A object at 0x000002A7BBA31100>
A.x.fdelete() self=<__main__.A object at 0x000002A7BBA31100>
```

### 对象 a2 创建及属性操作部分

```text
----- before a2 = A() -----
A.__init__() self=<__main__.A object at 0x000002A7BBA31160>        # a2 实例对象初始化，a2 和 a1 不是相同的对象

                            # 以下的调用整体上与前面 a1 的调用方式是一样的
                            # 但是请注意：尽管 a2 和 a1 不是相同的对象，但 Property 对象还是同一个
                            # 这说明类对应的描述器对象其实是唯一的
----- before   a2.x   -----
Property.__get__() self=<__main__.Property object at 0x000002A7BBA310A0> obj=<__main__.A object at 0x000002A7BBA31160> objtype=<class '__main__.A'>
A.x.fget() self=<__main__.A object at 0x000002A7BBA31160>

-----before  a2.x = 1 -----
Property.__set__() self=<__main__.Property object at 0x000002A7BBA310A0> obj=<__main__.A object at 0x000002A7BBA31160>
A.x.fset() self=<__main__.A object at 0x000002A7BBA31160>

-----before del a2.x  -----
Property.__delete__() self=<__main__.Property object at 0x000002A7BBA310A0> obj=<__main__.A object at 0x000002A7BBA31160>
A.x.fdelete() self=<__main__.A object at 0x000002A7BBA31160>
```

## @property中的小陷阱

{% hint style="danger" %}
**@property 装饰后生成的描述器实际上成为了数据描述器！**
{% endhint %}

## 扩展内容

其实在同一篇中官方也给出了 `staticmethod` 和 `classmethod` 的类似模拟实现，有兴趣的读者也不妨一览：

```python
class StaticMethod(object):
    "Emulate PyStaticMethod_Type() in Objects/funcobject.c"

    def __init__(self, f):
        self.f = f

    def __get__(self, obj, objtype=None):
        return self.f

class ClassMethod(object):
    "Emulate PyClassMethod_Type() in Objects/funcobject.c"

    def __init__(self, f):
        self.f = f

    def __get__(self, obj, klass=None):
        if klass is None:
            klass = type(obj)
        def newfunc(*args):
            return self.f(klass, *args)
        return newfunc
```

## 参考

1. [描述器使用指南](https://docs.python.org/zh-cn/3.9/howto/descriptor.html) 
2. [property 装饰器的描述器实现](property.md)
3. [PEP 318 -- Decorators for Functions and Methods](https://www.python.org/dev/peps/pep-0318/)
4. [PEP 3129 – Class Decorators](https://www.python.org/dev/peps/pep-3129/)

