---
description: Python Enhancement Proposals
---

# PEP 阅读清单

{% hint style="success" %}
[**https://www.python.org/dev/peps/**](https://www.python.org/dev/peps/)
{% endhint %}

{% hint style="info" %}
**只是一份简单的清单，只看右侧目录即可获得所有信息。**
{% endhint %}

## PEP 0 -- IndePEPx of Python Enhancement Proposals (PEPs)

### Python 增强建议（PEP）的索引

### PEP 索引

### 【浏览】

## PEP 1 -- PEP Purpose and Guidelines

### PEP 目的和指南

### 内容简述

### 【浏览】

## PEP 8 -- Style Guide for Python Code

### Python 代码样式指南

### 代码样式

### 【理解】：美观简洁，可阅读性

## PEP 20 -- The Zen of Python

### Python 之禅

### `import this`

### 【浏览】

## PEP 230 -- Warning Framework

### 警告框架

### `warning.warn(message [，category [，stacklevel]])` `warnings.filterwarnings(message, category, module, lineno, action)`

### 【理解】：多用 `DeprecationWarning` 提醒用户过期的内容

## PEP 249 -- Python Database API Specification v2.0

### Python 数据库API规范 v2.0

### 数据库 API 2.0

### 【浏览】

## PEP 255 -- Simple Generators

### 简单生成器

### 生成器的来源

### 【理解】：世纪大突破啊！

## PEP 318 -- Decorators for Functions and Methods

### 函数和方法的装饰器

### 函数前 `@`

### 【理解】：装饰器就是个返回函数的函数

## PEP 342 -- Coroutines via Enhanced Generators

### 基于拓展生成器的协程

### `send()` `throw()` `close()`

### 【理解】：`yield` 由语句变为表达式，生成器扩展了新方法

## PEP 343 -- The "with" Statement

### “with”声明

### `with` `@contextmanager`

### 【理解】：`@contextmmanager` 装饰的函数中包围yield的try仍线性执行

## PEP 380 -- Syntax for Delegating to a Subgenerator

### 委派子生成器的语法

### `yield from`

### 【理解】：`yield from` 生成的链中异常的处理（可以throw到底层）

## PEP 479 -- Change StopIteration handling inside generators

### 更改生成器内部的 `StopIteration` 处理

### 生成器中主动抛出的 `StopIteration` 被包装为 `RuntimeError`

### 【理解】：防止yield链的隐式意外中断

## PEP 483 -- The Theory of Type Hints

类型提示的理论

### `typing`

### 【浏览】：就是你把鬼子（静态类型标注）引进村的吗？

## PEP 484 -- Type Hints

### 类型提示

### `typing`

### 【浏览】：终究还是躲不开类型

## PEP 487 -- Simpler customisation of class creation

### 简化类创建的自定义

### `__set_name__` `__init_subclass__`

### 【理解】：`__new__` 中先后增添 `__set_name__` `__init_subclass__`

## PEP 492 -- Coroutines with async and await syntax

### 具有异步和等待语法的协程

### `async` `await`

### 【浏览】

## PEP 508 -- Dependency specification for Python Software Packages

### Python 软件包依赖关系规范

### 内容简述

### 【待阅】

## PEP 514 -- Python registration in the Windows registry

### Windows 注册表中的 Python 注册

### 内容简述

### 【待阅】

## PEP 520 -- Preserving Class Attribute Definition Order

### 保留类属性的定义顺序

### `__definition_order__`

### 【理解】：3.6 中 compact dict 已经替代了 **definition\_order**，字典默认保留插入顺序

## PEP 525 -- Asynchronous Generators

### 异步生成器

### 内容简述

### 【浏览】

## PEP 526 -- Syntax for Variable Annotations

### 变量标注的语法

### `:`

### 【理解】：PEP 484 基础上的变量标注

## PEP 544 -- Protocols: Structural subtyping (static duck typing)

### 协议：结构性分型（静态鸭式 typing）

### 内容简述

### 【待阅】

## PEP 557 -- Data Classes

### 数据类

### `@dataclass`

### 【理解】：`collections.namdetuple()` 从此有了大个的傻弟弟

## PEP 560 -- Core support for typing module and generic types

### 键入模块和泛型类型的核心支持

### `typing`

### 【待阅】

## PEP 563 Postponed Evaluation of Annotations

### 延迟的标注求值

### `a: a = ...`

### 【理解】：注解中的前向引用不必再用 `""`

## PEP 614 -- Relaxing Grammar Restrictions On Decorators

### 放宽对装饰器的语法限制

### `@Anything`

### 【理解】：解放装饰器，`@` 可跟任意表达式

## PEP 617 -- New PEG parser for CPython

### 用于CPython的新PEG解析器

### 新的语法解析器

### 【浏览】

## PEP 634 -- Structural Pattern Matching: Specification

### 结构化模式匹配：规范

### `match ... case:`

### 【理解】：`as` `|` literal capture `_` value `()` sequence mapping class(`__match_args`)

## PEP 635 -- Structural Pattern Matching: Motivation and Rationale

### 结构化模式匹配：动机和理由

### `match ... case:` 的设计考虑

### 【理解】：构造与获取（匹配）的对称

## PEP 636 -- Structural Pattern Matching: Tutorial

### 结构化模式匹配：教程

### `match ... case:` 的教程

### 【理解】：Pythonic 的 switch，使用很便利

## PEP 3107 -- Function Annotations

### 函数标注

### `__annotations__`

### 【理解】：`identifier [: expression] [= expression]`

## PEP 3115 -- Metaclasses in Python 3000

### Python 3000 中的元类

### `__prepare__()`

### 【理解】：搭配 3.6 引入的 compact dict 可控制类属性创建顺序

## PEP 3119 -- Introducing Abstract Base Classes

### 介绍抽象基类

### `abc`

### 【理解】：抽象基类相关，便于检查具体接口实现情况

## PEP 3129 -- Class Decorators

### 类装饰器

### 类前 `@`

### 【理解】：PEP 318 `@` 只能用在函数和方法前，这里扩展到类

## PEP 3135 -- New Super

### 新 super

### `super()` `__class__`

### 【理解】：方法内可以直接用 `__class__`，且 `super()` 可省略参数

## PEP 3153 -- Asynchronous IO support

### 异步IO支持

### `asyncio`

### 【待阅】

## PEP 3333 -- Python Web Server Gateway Interface v1.0.1

### Python Web 服务器网关接口 v1.0.1

### WSGI

### 【浏览】

## 模板

### 标题翻译

### 内容简述

### 【待阅】【浏览】【理解】
