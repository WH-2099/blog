---
description: 动态类型写着爽，维护升级火葬场
---

# 类型标注

## 序——大势所趋的类型静态化

编程语言界最常见到的动态类型语言莫过于 Javascript 和 Python，尽管二者在其设计哲学、应用领域方面有着巨大的差异，但随着它们用户基数的增长，都出现了一个相同的发展趋势——类型静态化。对于 Javascript 来说，Typescript 的出现就是最好的例子；而对于 Python 而言，由 PEP 484 在标准库中引入的 `typing` 模块也可见一斑。

好好的动态类型，为什么都要朝着静态类型发展呢？

一言以概之：“**动态类型写着爽，维护升级火葬场**”。

就以我们的主角 Python 而言，在其发展早期，它只不过是作为一个小小的工具语言，用以在大型项目中的边边角角做一些简单的辅助性处理。那时的 Python ，一般也就是个两三千行撑死，莫说是复杂的类型结构，很可能连 `class` 语句都不会出现。在这种基本就是内置类型传来传去的情况下，动态类型让老练的开发者能够快速高效地完成简单的工作，一时之间受到了大家的追捧。

伴随着 Python 的流行，越来越多的项目采用 Python 作为主语言。而当一个项目拥有过万行的代码量以及完整而精巧的类型结构设计时，动态类型的弊端就开始显现。函数签名的自解释性匮乏，开发者不得不“面向文档编程”；错误的类型传递无法避免，隐藏的 BUG 开始积累；类型的含义逐步被淡化，精心构造的类型结构失去价值……这时候，大家又开始怀念起了静态类型的好，写代码时多跳几个 `type error` 总好过 DEBUG 时抓心挠肺。

在这段时期，一个名叫 mypy 的第三方包的兴起令 Python 核心开发者意识到了真正的问题所在：**码农们对于静态类型的偏好其实源于对类型检查器的依赖**。mypy 就是一个静态类型检查器，通过解析 Python 代码及其中包含的特定的注释，在 Python 代码实际运行前进行静态的类型检查。而这是一个兼容性非常棒的解决方案：类型声明的内容均包含在注释中，不会对原有的代码含义产生任何影响；类型检查独立工作在代码运行前，不会对代码的实际运行产生任何影响。2014 年 9 月 29 日，Python 之父 Guido van Rossum 、 mypy 之父 Jukka Lehtosalo 及 Python 核心开发者 Łukasz Langa 联手发布了 [PEP 484 -- Type Hints](https://www.python.org/dev/peps/pep-0484/) ，在 Python 标准库中引入了全新的 `typing` 模块，提供对类型标注的官方支持**。**&#x20;





##

###



## 参考

1. [PEP 483 -- The Theory of Type Hints](https://www.python.org/dev/peps/pep-0483/)
2. [PEP 484 -- Type Hints](https://www.python.org/dev/peps/pep-0484/)
3. [PEP 526 -- Syntax for Variable Annotations](https://www.python.org/dev/peps/pep-0526/)
4. [PEP 544 -- Protocols: Structural subtyping (static duck typing)](https://www.python.org/dev/peps/pep-0544/)
5. PEP 585
6. PEP 586
7. PEP 589
8. PEP 591
9. PEP 593
10. PEP 604
11. [typing --- 类型提示支持](https://docs.python.org/zh-cn/3/library/typing.html)
