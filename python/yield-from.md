---
description: yield from 委派生成器语句解析
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

### 1.1 委派生成器 `yield from`

_在 Python 3.3 版本（2011年）中，经由 PEP 380 引入了这个新的语法，其目的主要是为了解决伴随着生成器在 Python 中的大量使用而出现的生成器间组合的需求。_

首先我们来看一看各种官方资料中有关这一委派的说明好了：

> * **供生成器将其部分操作委托给另一生成器。这允许包含 `yield` 的一段代码被分解并放置在另一个生成器中。此外，允许子生成器返回一个值，并且该值可用于委派生成器。**
> * 当使用 `yield from <expr>` 时，所提供的表达式必须是一个 **可迭代对象** 。**迭代该可迭代对象所产生的值会被直接传递给当前生成器方法的调用者**。任何通过 [`send()`](https://docs.python.org/zh-cn/3/reference/expressions.html#generator.send) 传入的值以及任何通过 [`throw()`](https://docs.python.org/zh-cn/3/reference/expressions.html#generator.throw) 传入的异常 **如果有适当的方法则会被传给下层迭代器** 。 如果不是这种情况，那么 [`send()`](https://docs.python.org/zh-cn/3/reference/expressions.html#generator.send) 将引发 [`AttributeError`](https://docs.python.org/zh-cn/3/library/exceptions.html#AttributeError) 或 [`TypeError`](https://docs.python.org/zh-cn/3/library/exceptions.html#TypeError)，而 [`throw()`](https://docs.python.org/zh-cn/3/reference/expressions.html#generator.throw) 将立即引发所转入的异常。

感觉不说人话啊有莫有。。。。。。没事不要急，咱们先把这死板的概念放一边，来看看这玩意都有什么用：

* 这个语法的提出促进了 **基于生成器的协程** 的发展，如今所使用的协程关键字 `async` `await` 以及标准库中的 `asyncio` 就是最好的例子。如果你也曾为这些复杂的概念头疼过 XD，那你必然在回溯报错的过程中多多少少见到了底层一个又一个的 `yield from` 。

{% hint style="info" %}
协程相关内容超出本文讨论范围，在这里不作展开。
{% endhint %}

还是不懂？那我们用代码来说话：

```python
def no_yield_from_generator():
    iterable = (0, 1, 2, 3)
    for i in iteralbe:
        yield i

def yield_from_generator():
    iterable = (0, 1, 2, 3)
    yield from iterable
```

简单直白，一眼就能懂意思是吧？`yield from` 就是可以让我们少写一层 `for` ？

不要急，这只是它最简单的作用罢了。**你有没有想过，如果这个 `iterable` 本身也是个生成器，那么外面一层生成器的 `send()` `throw()` `close()` 被调用后要怎么办？**

{% hint style="success" %}
**委派生成器 `yield from` 最重要的作用是当一个 中介 ，协调两个嵌套的生成器的工作** 。
{% endhint %}

咱当时理解到这里的时候，心生疑惑了：不就是个委托调用吗？能有多难，咱自己做不行吗？

咱现在可以明确的说，还真的挺难的，原因且听咱细细道来。

生成器用着方便，但其底层的实现还是相对复杂的。更不幸的是，在基于生成器的协程发展的过程中，为生成器引入了 `send()` `throw()` `close()` 等新方法，这让委托调用变得复杂化_（中介哪有那么好当 ╮\(╯▽╰\)╭）_。  
尽管可以自己来手工实现一些简单的，但随着条件的复杂化和调用的多样化，这些仅关注于部分内容的处理都会变成埋下的 **雷** 。  
_相信我，你不会想亲自体验一次的 ╰\(艹皿艹 \)！_  
所以这种 ~~黑魔法研究~~ 的任务还是交给官方开发组来做吧。

~~yield from 中介所，多个中间商赚差价，让你的代码更好看！~~

## 2. 原理解析

### 2.0 完整语义

首先，既然 `yield from` 涉及了两个生成器，为了便于后文叙述，我们需要先区分它们。以代码为例：

```python
def generator():
    for i in range(9):
        yield i

def delegated_generator():
    yield from generator()    
    
```

外层的生成器 **`delegated_generator()` 称作 委派生成器**，内层的这个 **`generator()` 就简单称之为 迭代器** 就好。

{% hint style="info" %}
**生成器本质上是一种特殊的迭代器。**
{% endhint %}

`yield from` 的具体语义如下：

* 作为一个中介，首先 **迭代器产生的任何值都将直接传递给调用方** 。
* **使用** **`send()` 发送给委派生成器的任何值都将直接传递给迭代器：**
  * 如果发送的值为 `None`，则调用迭代器的 `__next __()` 方法。
  * 如果发送的值不为 `None`，则调用迭代器的 `send()` 方法。如果调用引发 `StopIteration` 则恢复委派生成器的执行。任何其他异常都会传播到委派生成器。
* **抛出到委派生成器中的 `GeneratorExit` 以外的异常被传递给迭代器的 `throw()` 方法** 。如果调用引发 `StopIteration`，则恢复委派生成器的执行。任何其他异常都会传播到委派生成器。
* **如果在委派生成器中抛出 `GeneratorExit` 异常，或者委派生成器的 `close()` 方法被调用，则尝试调用（如果有则调用）迭代器的 `close()` 方法如果** 。如果此调用导致异常，则将其传播到委派生成器。否则，在委派生成器中引发 `GeneratorExit`。
* `yield from` **表达式的值** 是迭代器终止时引发的 **`StopIteration` 异常的第一个参数** 。
* 在生成器中，**`return expr`** 会导致在退出生成器时引发 **`StopIteration(expr)`** 。

### 2.1 代码示范

```python
RESULT = yield from EXPR
```

**↑↑↑↑↑↑ 上面这个语句，实际就相当于 ↓↓↓↓↓↓**

```python
iterator = iter(EXPR)
try:
    yield_value = next(iterator)
except StopIteration as exception:
    result_value = exception.value
else:
    while 1:
        try:
            send_value = yield yield_value
        except GeneratorExit as generator_exit:
            try:
                close_method = iterator.close
            except AttributeError:
                pass
            else:
                close_method()
            raise generator_exit
        except BaseException as base_exception:
            exception_info = sys.exc_info()
            try:
                throw_method = iterator.throw
            except AttributeError:
                raise base_exception
            else:
                try:
                    yield_value = throw_method(*exception_info)
                except StopIteration as stop_iteration:
                    result_value = stop_iteration.value
                    break
        else:
            try:
                if send_value is None:
                    yield_value = next(iterator)
                else:
                    yield_value = iterator.send(send_value)
            except StopIteration as stop_iteration:
                result_value = stop_iteration.value
                break
RESULT = result_value
```

## 3. 参考源

[PEP 255 -- Simple Generators](https://www.python.org/dev/peps/pep-0255/)

[PEP 342 -- Coroutines via Enhanced Generators](https://www.python.org/dev/peps/pep-0342/)

[PEP 380 -- Syntax for Delegating to a Subgenerator](https://www.python.org/dev/peps/pep-0380/)





