---
description: yield from 语句解析
---

# 如何做一个好中介

## 0. 声明

{% hint style="warning" %}
**不从 `0` 开始难不成从 `1` 开始？你知道吗，世上只有 `10` 种人，程序员和非程序员;** 
{% endhint %}

{% hint style="info" %}
本文假设您已经掌握 **生成器（**Generators**）** 相关的知识
{% endhint %}

## 1. 基础回顾

### 1.0 生成器（Generators） `yield`

{% hint style="info" %}
Python 生成器本质是一种 **迭代器** ，但功能更强大。
{% endhint %}

在 Python 2.2 版本（2001 年）中，经由 [PEP 255](https://www.python.org/dev/peps/pep-0255/) 引入了 **生成器** 的概念。其最初是为了解决开发者经常遇到的一个小问题：

> 当生产者函数完成一项艰苦的工作以至于 **需要维持所生成的值之间的状态** 时，大多数编程语言都无法提供一种愉悦而有效的解决方案，只能将回调函数添加到生产者的参数列表中，以对每个生成的值进行调用。

自此以后，Python 的关键字大家庭里又多了一员 **`yield`** 。而生成器这个概念，也伴随着 Python 走过了  20 年 的风风雨雨，至今，它已经是 Python 中不可分离的重要特性。

{% hint style="warning" %}
探讨生成器的重要性及其延伸开来的广阔特性（如基于生成器的协程）足以另起一篇万字长文，故在此不多做涉猎，但可以肯定的是：

**不用生成器，何谈 Pythonic？**
{% endhint %}

### 1.1 生成器委托 `yield from`

&gt; \*\*\`供生成器将其部分操作委托给另一生成器\`\*\*。这允许包含 \`yield\` 的一段代码被分解并放置在另一个生成器中。此外，允许子生成器返回一个值，并且该值可用于委派生成器。



在 Python 3.3 版本（2011年）中，经由 PEP 380 \[^0\] 引入了这个新的语法，其目的主要是为了解决伴随着生成器在 Python 中的大量使用而出现的生成器间组合的需求。



\`\`\`

yield from &lt;expr&gt;

\`\`\`

其中，\`&lt;expr&gt;\` 是计算为可迭代的表达式，可从中提取迭代器。迭代器运行直到终止状态，在此期间，迭代器产生并直接向生成器的调用者传送值，或直接从生成器的调用者接收值，其中包含 \`yield from\` 表达式的生成器（“委托生成器”）。



这个语法的提出促进了\*基于生成器\*的协程的发展，如今所使用的协程关键字 \`async\` \`await\` 以及标准库中的 \`asyncio\` 就是最好的例子，如果你也曾为这些复杂的概念头疼过 XD，那你必然在回溯报错的过程中多多少少见到了底层一个又一个的 \`yield from\` 。



\*\*表面上看来，\`yield from\` 的工作很简单，就是当一个==中介==，串联两个生成器。可惜，生成器用着方便，但其底层的实现还是相对复杂的，更不幸的是，在基于生成器的协程发展的过程中，为生成器引入了 \`send\(\)\` \`throw\(\)\` \`close\(\)\` 等新方法，这让串联的任务变得十分艰难（中介也不好当啊）。\*\*



\*尽管可以自己来实现一些简单的串联，但在特定的条件下，这些仅关注于表面的处理都会变成埋下的 雷 ，所以这种~~黑魔法改进~~的工作还是得交给官方开发组。\*

\*\[雷\]: 相信我，你不会想自己体验一次的 ╰\(艹皿艹 \)



## 2. 原理解析

### 2.0 完整语义

作为一个中介，首先

迭代器产生的任何值都将直接传递给调用方。

使用send（）发送给委派生成器的任何值都将直接传递给迭代器。如果发送的值为None，则调用迭代器的\_\_next \_\_（）方法。如果发送的值不为None，则调用迭代器的send（）方法。如果调用引发StopIteration，则将委派生成器。任何其他异常都会传播到委托生成器。

委派给生成器的GeneratorExit之外的其他异常将传递给迭代器的throw（）方法。如果调用引发StopIteration，则将委派生成器。任何其他异常都会传播到委托生成器。

如果GeneratorExit异常扔进委派发生器，或关闭（）的委托发生器的方法被调用，则关闭（） ，如果它具有一个迭代器的方法被调用。如果此调用导致异常，则将其传播到委托生成器。否则，将在委派的生成器中引发GeneratorExit。

表达式的yield值是迭代器终止时引发的StopIteration异常的第一个参数。

生成器中的return expr导致从生成器退出时StopIteration（expr）升高。

### 2.1 代码示范

```python
RESULT = yield from EXPR

_i = iter(EXPR)
try:
    _y = next(_i)
except StopIteration as _e:
    _r = _e.value
else:
    while 1:
        try:
            _s = yield _y
        except GeneratorExit as _e:
            try:
                _m = _i.close
            except AttributeError:
                pass
            else:
                _m()
            raise _e
        except BaseException as _e:
            _x = sys.exc_info()
            try:
                _m = _i.throw
            except AttributeError:
                raise _e
            else:
                try:
                    _y = _m(*_x)
                except StopIteration as _e:
                    _r = _e.value
                    break
        else:
            try:
                if _s is None:
                    _y = next(_i)
                else:
                    _y = _i.send(_s)
            except StopIteration as _e:
                _r = _e.value
                break
RESULT = _r
```

## 3. 参考源

[PEP 255 -- Simple Generators](https://www.python.org/dev/peps/pep-0255/)

[PEP 342 -- Coroutines via Enhanced Generators](https://www.python.org/dev/peps/pep-0342/)

[PEP 380 -- Syntax for Delegating to a Subgenerator](https://www.python.org/dev/peps/pep-0380/)





