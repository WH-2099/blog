---
description: PyCon China 2023 主题演讲
---

# “类”的渐进式剖析

类是面向对象编程中的核心概念，更是 Python 中不可或缺的一部分。 几乎每一个 Python 用户都曾写过 class 语句，但你可曾想过在这个优雅的语句下，到底包装了怎样的底层设计和实现？ 本文将以自顶向下的形式，从最基础的类构建顺序展开，渐进式地深入元类、属性/方法解析顺序、描述器、C3—MRO 算法等多个极其重要的底层概念。 随后，我们将在此基础上展开 typing 类型注解中的类型系统理论，并尝试探讨关于 None 值的开放性问题。

### 0o10 你真的“创建”了类吗？

_类_是 OOP（面向对象编程）的核心概念，更是 Python 中不可或缺的一部分。

一般情况下我们会这样“创建”类：

```python
class A(object):  # Python 3 的类默认继承自 object
    ...
```

然后我们就可以用 `A` 这个名称来引用这个类了，类似于这样：

```python
a = A()
```

如果我们抛开 `A` 作为类的特殊身份，单独审视上面这个语句。 根据基础的变量引用语法，既然能够引用 `A` 这个名字，那么必然发生了针对于 `A` 的赋值。 也许你会觉得这背后是一个类似于 `A = class ...` 语法糖，但要知道 `class` 语句并不能作为合法的右值。

想要了解真相，这一切都还要从 Python 中类的具体概念说起：

> 在 Python 中，类是由 class 语句定义的对象工厂，并由 type(obj) 内置函数返回。 是一个动态的、运行时的概念。 —— PEP 483

准确地说，`class` 语句只是定义了类，但此时语句块里的内容还只是静态的、运行前的声明。 所以我们并没有直接“创建”类，准确地说，我们只是设计了类。 真正创建类的操作是在代码实际运行的过程中，由解释器完成的。

解释器实际创建的类本质上是 `type` 的一个对象，称为 **类对象**，它正是上文中 `A =` 中等号右侧的内容。

_注意这里说的“类对象”不是“类实例”，类对象指的是类型本身作为一个对象，而类实例是指类型的实例化后的生成的、从属于类型的对象。_

那么从 `class` 语句到最终的类对象生成，解释器都做了些什么呢？ 最直观的莫过于标准库模块 `types` 的源码了：

```python
# Provide a PEP 3115 compliant mechanism for class creation
def new_class(name, bases=(), kwds=None, exec_body=None):
    """Create a class object dynamically using the appropriate metaclass."""
    resolved_bases = resolve_bases(bases)
    meta, ns, kwds = prepare_class(name, resolved_bases, kwds)
    if exec_body is not None:
        exec_body(ns)
    if resolved_bases is not bases:
        ns['__orig_bases__'] = bases
    return meta(name, resolved_bases, ns, **kwds)

def resolve_bases(bases):
    """Resolve MRO entries dynamically as specified by PEP 560."""
    new_bases = list(bases)
    updated = False
    shift = 0
    for i, base in enumerate(bases):
        if isinstance(base, type):
            continue
        if not hasattr(base, "__mro_entries__"):
            continue
        new_base = base.__mro_entries__(bases)
        updated = True
        if not isinstance(new_base, tuple):
            raise TypeError("__mro_entries__ must return a tuple")
        else:
            new_bases[i+shift:i+shift+1] = new_base
            shift += len(new_base) - 1
    if not updated:
        return bases
    return tuple(new_bases)

def prepare_class(name, bases=(), kwds=None):
    """Call the __prepare__ method of the appropriate metaclass.

    Returns (metaclass, namespace, kwds) as a 3-tuple

    *metaclass* is the appropriate metaclass
    *namespace* is the prepared class namespace
    *kwds* is an updated copy of the passed in kwds argument with any
    'metaclass' entry removed. If no kwds argument is passed in, this will
    be an empty dict.
    """
    if kwds is None:
        kwds = {}
    else:
        kwds = dict(kwds) # Don't alter the provided mapping
    if 'metaclass' in kwds:
        meta = kwds.pop('metaclass')
    else:
        if bases:
            meta = type(bases[0])
        else:
            meta = type
    if isinstance(meta, type):
        # when meta is a type, we first determine the most-derived metaclass
        # instead of invoking the initial candidate directly
        meta = _calculate_meta(meta, bases)
    if hasattr(meta, '__prepare__'):
        ns = meta.__prepare__(name, bases, **kwds)
    else:
        ns = {}
    return meta, ns, kwds

def _calculate_meta(meta, bases):
    """Calculate the most derived metaclass."""
    winner = meta
    for base in bases:
        base_meta = type(base)
        if issubclass(winner, base_meta):
            continue
        if issubclass(base_meta, winner):
            winner = base_meta
            continue
        # else:
        raise TypeError("metaclass conflict: "
                        "the metaclass of a derived class "
                        "must be a (non-strict) subclass "
                        "of the metaclasses of all its bases")
    return winner
```

#### 0o11 类的创建流程

1. **执行到类定义代码**
2. **解析 MRO 条目**
3. **确定适当的元类**
4. **准备类命名空间**
5. **执行类主体**
6. **创建类对象**
7. **准备执行后续代码**

#### 0o12 元类存在的必要性

在 OOP 中，动态的类创建是偏向于底层的高级概念。\
OOP 的核心是 **“万物皆对象”** ，而类作为程序员设计并使用的基础工具，也应该被视为对象。\
那么自然，**类对象** 需要有他的类，而这个类就是 **元类**。\
元类是留给类设计者的最底层概念，在其之下的都属于具体语言的实现，不再是 OOP 应该关注的内容。

而相较于 Java 等语言的反射机制，Python 选择直接将一部分类的创建流程公开给程序员，无疑是秉承了 **“显示优于隐式”** 的哲学。\
尽管在平常的使用中普通程序员很少会直接去影响类的创建，但在很多复杂而神奇的包开发中这却是不可或缺的工具。\
而事实上这的确成为了 Python 的 **重要优势** 之一，至今元类的应用非常广泛，包括且不限于 \_枚举、日志、接口检查、自动委托、自动特征属性创建、代理、框架以及自动资源锁定/同步 \_等等。

#### 0o13 元类使用中的问题及优化方式

首先，**元类是一个相当复杂的概念**。即便你已经对此有所了解，但实际应用时总还是会出现各种各样的问题，而且这些个问题往往会是由于底层概念上的递归造成的。

其次，**没有自动方法可以组合元类**。如果要为一个类使用两个元类，则通常需要手动创建一个将这两个类合并在一起的新元类。\
这种需求常常使用户感到意外：从两个不同的库继承的两个基类继承突然增加了手动创建组合元类的必要性，通常情况下，人们对这些库的那些细节完全不感兴趣。\
如果一个库开始使用以前从未使用过的元类，这将变得更加糟糕。当库本身继续正常工作时，将这些类与另一个库中的类组合在一起的每个代码突然都失败了。

尽管有多种使用元类的方法，但绝大多数用例可分为三类：

1. 在类创建后运行的一些初始化代码
2. 描述器的初始化
3. 保持类属性定义的顺序

事实上为了满足这些需求，我们 **未必就非用元类不可**。

通过对类的创建进行简单的 **挂钩** 就可以轻松实现前两个类别：

一个 `__init_subclass__` 挂钩初始化一个给定类的子类。\
创建类时，对类中定义的所有属性（描述符）调用 `__set_name__` 挂钩。

而第三个类别自从 Python 3.6 进行了 compact dict 的优化使 `dict` 类型默认保留插入顺序后，通过 `__prepare__` 也完全可以满足。

{% hint style="success" %}
**在当前的版本中，元类不再是我们深度定制类创建的唯一选择，活用以下这三个魔法方法，你就可以和元类说再见了：**

* `__init_subclass__`
* `__set_name__`
* `__prepare__`
{% endhint %}

### 0o20 基于过程如何面向对象？

> 面向对象设计有三个基本特征：封装、继承和多态。 ——《设计模式：可复用面向对象软件的基础》

Python 是一门基于解释器的语言，也就是我们常说的“脚本语言”。 作为脚本语言，Python 的解释器需要逐行读取代码并顺序执行，所以 Python 的底层语言设计是基于过程的。 _请注意这里提到的是“底层语言设计”，而非“底层实现”。因为无论语言本身多么花哨，CPU 最终执行的机器码仍是面向操作的，而面向操作可以说是面向过程的一个子集，所以从本质上来说，所有语言的底层实现都是基于过程的。_ 同时，xx

Linus 所强调的“Nothing better than C” 不严谨地讲，对象是过程上的再封装

#### 0o21 `struct` 与 `class` 的对比

封装是共有的，因为“继承、多态”其实就是封装的一种实现方式。

对比 “.” 语法关联的 struct 和 class

1. 表面的 self 传入
2. 继承与多态

#### 0o22 描述器

描述器不仅是个简单的 hook，更多的要涉及到属性查找顺序，进而到mro c3-mro算法 面向对象，但是又把类当作聚合隔离的工具，所以就有了 类对象

#### 0o23 属性访问顺序

面向对象与面向过程的重要区别之一就是`something.attribute`形式的调用，这被称为属性调用，这里的属性是指广义上的，**包括基础属性和方法**。

**对象属性访问顺序**

1. 依照 **MRO** 顺序的类的类属性中的 **数据描述器属性**
2. **实例对象的属性**`object.__dict__`
3. 依照 MRO 顺序的类的类属性中的 **非数据描述器属性**
4. 依照 MRO 顺序的类的类属性中的 **普通（非描述器）属性**`cls.__dict__`

**类属性访问顺序**

1. 依照元类的 MRO 顺序的类的类属性中的 **元类数据描述器属性**
2. 依照 MRO 顺序的类的类属性中的 **数据描述器属性**
3. 依照 MRO 顺序的类的类属性中的 **普通（非描述器）属性**`cls.__dict__`
4. 依照 MRO 顺序的类的类属性中的 **非数据描述器属性**
5. 依照元类的 MRO 顺序的类的类属性中的 **元类非数据描述器属性**

{% hint style="info" %}
Python 中 **类的本质是元类创建的对象**，所以相当于是外面套了一层不包含普通属性（亦即非描述器属性）的对象属性访问顺序。
{% endhint %}

#### 0o24 C3-MRO

{% hint style="info" %}
**方法解析顺序（Method Resolution Order, MRO）** 是在 **面向对象编程** 中，当某个实例对象应用了继承，进而引发 **多态** 特性时，编译/解释器 **查找并决定具体实例方法的顺序** 。
{% endhint %}

一般情况下所提到的 MRO 基本都是指复杂多继承中的 MRO，其本质是 **一个顺序**，**可用具体编程语言中的序列来表示（Python 中就是 `collections.abc.Sequence`）**，本文同一般情况。

MRO 的作用：

* 实现方法重载
* 构建 OOP 多态
* 保证继承有效

### 0o30 路在何方？

在前文的基础上，我们可以展开一些开放性的探讨。

#### 0o31 类型标注 `typing` 何以成为大势所趋？

起源于大规模代码维护 一言以概之：“**动态类型写着爽，维护升级火葬场**”。 就以我们的主角 Python 而言，在其发展早期，它只不过是作为一个小小的工具语言，用以在大型项目中的边边角角做一些简单的辅助性处理。那时的 Python，一般也就是个两三千行撑死，莫说是复杂的类型结构，很可能连 `class` 语句都不会出现。在这种基本就是内置类型传来传去的情况下，动态类型让老练的开发者能够快速高效地完成简单的工作，一时之间受到了大家的追捧。

伴随着 Python 的流行，越来越多的项目采用 Python 作为主语言。而当一个项目拥有过万行的代码量以及完整而精巧的类型结构设计时，动态类型的弊端就开始显现。函数签名的自解释性匮乏，开发者不得不“面向文档编程”；错误的类型传递无法避免，隐藏的 BUG 开始积累；类型的含义逐步被淡化，精心构造的类型结构失去价值……这时候，大家又开始怀念起了静态类型的好，写代码时多跳几个 `type error` 总好过 DEBUG 时抓心挠肺。

在这段时期，一个名叫 mypy 的第三方包的兴起令 Python 核心开发者意识到了真正的问题所在：**码农们对于静态类型的偏好其实源于对类型检查器的依赖**。mypy 就是一个静态类型检查器，通过解析 Python 代码及其中包含的特定的注释，在 Python 代码实际运行前进行静态的类型检查。而这是一个兼容性非常棒的解决方案：类型声明的内容均包含在注释中，不会对原有的代码含义产生任何影响；类型检查独立工作在代码运行前，不会对代码的实际运行产生任何影响。2014 年 9 月 29 日，Python 之父 Guido van Rossum、mypy 之父 Jukka Lehtosalo 及 Python 核心开发者 Łukasz Langa 联手发布了 [PEP 484 -- Type Hints](https://www.python.org/dev/peps/pep-0484/) ，在 Python 标准库中引入了全新的 `typing` 模块，提供对类型标注的官方支持。

> Type concept is described above, types appear in variable and function type annotations, can be constructed from building blocks described below, and are used by static type checkers. 类型概念如上所述，类型出现在变量和函数类型注释中，可以从下面描述的构建块构造，并由静态类型检查器使用。

> In Python, classes are object factories defined by the `class` statement, and returned by the `type(obj)` built-in function. Class is a dynamic, runtime concept. 在 Python 中，类是由 class 语句定义的对象工厂，并由 type(obj) 内置函数返回。 是一个动态的、运行时的概念。 —— PEP 483

1. Python 中由 Type Hints 及其标志性的 `typing` 模块代表的类型化是一种自顶向下的过程，从某种角度上来讲，这算是一种动态类型到静态类型的逆向工程
2. 常见的强类型语言中 则是 自底向上
3. 渐进式类型标注 Gradual typing allows one to annotate only part of a program, thus leverage desirable aspects of both dynamic and static typing. 渐进式类型化允许仅注解程序的一部分，从而充分利用动态和静态类型化的优点。 本文题目也正是自此而来。

除此之外还有更多的用途：

1. 被用于以 dataclass，Pydantic 为代表的 Model 式定义
2. 被用于以 LPython 为代表的性能（编译）优化
3. 或将被用于文档字符注释 [PEP 727 – Documentation in Annotated Metadata](https://peps.python.org/pep-0727/)

#### 0o32 None 为何如此棘手？

如果说 `NULL` 价值十亿美元，那么 Python 中的 `None` 少说也得价值百万美元。 不同于 C 中的指针与内存安全问题，Python 中 `None` 的问题主要出现在类型方面。

子类型关系

> every value from second\_type is also in the set of values of first\_type; second\_type 中的每个值也在 first\_type 的值集中； every function from first\_type is also in the set of functions of second\_type. first\_type 中的每个函数也在 second\_type 的函数集中。

推论

> 1. Every type is a subtype of itself.
> 2. 每种类型都是其自身的子类型。
> 3. The set of values becomes smaller in the process of subtyping, while the set of functions becomes larger.
> 4. 在子类型化的过程中，值的集合变得更小，而函数的集合变得更大。 pep483

> When used in a type hint, the expression None is considered equivalent to type(None). 当在类型提示中使用时，表达式 None 被视为等同于 type(None) 。 pep484

客观问题 None 的存在违反了值和方法集合的变化规律，打破了原有的类型系统.

主观问题 很多时候其实它的作用只是一个 sentinel `if` 的简便写法忽略了类型问题 `if None` `if []` 同为假

### 0o40 参考资料

#### 0o41 PEP

* [PEP 253 -- Subtyping Built-in Types](https://www.python.org/dev/peps/pep-0253/)
* [PEP 318 -- Decorators for Functions and Methods](https://www.python.org/dev/peps/pep-0318/)
* [PEP 483 -- The Theory of Type Hints](https://www.python.org/dev/peps/pep-0483/)
* [PEP 487 -- Simpler customisation of class creation](https://www.python.org/dev/peps/pep-0487/)
* [PEP 520 – Preserving Class Attribute Definition Order](https://peps.python.org/pep-0520/)
* [PEP 560 – Core support for typing module and generic types](https://peps.python.org/pep-0560/)
* [PEP 727 – Documentation in Annotated Metadata](https://peps.python.org/pep-0727/)
* [PEP 3115 -- Metaclasses in Python 3000](https://www.python.org/dev/peps/pep-3115/)
* [PEP 3129 – Class Decorators](https://www.python.org/dev/peps/pep-3129/)

#### 0o42 官方文档

* [数据模型](https://docs.python.org/zh-cn/3/reference/datamodel.html)
* [描述器使用指南](https://docs.python.org/zh-cn/3/howto/descriptor.html)
* [The Python 2.3 Method Resolution Order](https://www.python.org/download/releases/2.3/mro/)

#### 0o43 三方文档

* [The paper A Monotonic Superclass Linearization for Dylan](https://doi.org/10.1145/236337.236343)
* [C3 线性化算法与 MRO——理解Python中的多继承](http://kaiyuan.me/2016/04/27/C3\_linearization/)
* [C3 linearization](https://en.wikipedia.org/wiki/C3\_linearization)
* [The thread on python-dev started by Samuele Pedroni](https://mail.python.org/pipermail/python-dev/2002-October/029035.html)
* [Python 方法解析顺序MRO-C3算法](https://xubiubiu.com/2019/06/10/python-%E6%96%B9%E6%B3%95%E8%A7%A3%E6%9E%90%E9%A1%BA%E5%BA%8Fmro-c3%E7%AE%97%E6%B3%95/)
* [python高级编程——描述符Descriptor详解（上篇）——python对象的属性访问优先级与属性的控制与访问）](https://blog.csdn.net/qq\_27825451/article/details/84848341)
* [python高级编程——描述符Descriptor详解（中篇）——python对象的属性访问优先级与属性的控制与访问）](https://blog.csdn.net/qq\_27825451/article/details/84767061)
* [python高级编程——描述符Descriptor详解（下篇）——python描述符三剑客详解](https://blog.csdn.net/qq\_27825451/article/details/84848341)
* [python高级编程——描述符Descriptor详解（补充篇）——python描述符实现一些底层高级功能](https://blog.csdn.net/qq\_27825451/article/details/84848341)
* [描述器](https://www.dazhuanlan.com/2020/02/29/5e5965cea4d60/)
* [Python中的类与描述器(Descriptors)](https://blog.csdn.net/u013008795/article/details/90646667)
* [Python 3.11 更快的 CPython](https://www.white-winds.com/post/python%203.11%20fast%20cpython/)
* [PyConChina2022-深圳-大规模生产环境下的Faster CPython-王文洋](https://github.com/PyConChina/2022-slides/blob/main/Shenzhen/PyConChina2022-%E6%B7%B1%E5%9C%B3-%E5%A4%A7%E8%A7%84%E6%A8%A1%E7%94%9F%E4%BA%A7%E7%8E%AF%E5%A2%83%E4%B8%8B%E7%9A%84Faster%20CPython-%E7%8E%8B%E6%96%87%E6%B4%8B.pdf)
* [NULL: The Billion Dollar Mistake](https://hackernoon.com/null-the-billion-dollar-mistake-8t5z32d6)

#### 0o44 个人博客

* [你真的“创建”了类吗？](https://blog.wh2099.com/python/create-class)
* [MRO 三定律](https://blog.wh2099.com/python/c3-mro)
* [描述器学习指南](https://blog.wh2099.com/python/descriptor)
* [描述器实现 property](https://blog.wh2099.com/python/property)
* [类型标注](https://blog.wh2099.com/python/typing)
