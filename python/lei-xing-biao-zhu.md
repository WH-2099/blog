# 类型标注

## 序言

编程语言界最常见到的动态类型语言莫过于 Javascript 和 Python，尽管二者在其设计哲学、应用领域方面有着巨大的差异，但随着它们用户基数的增长，都出现了一个相同的发展趋势——类型静态化。对于 Javascript 来说，Typescript 的出现就是最好的例子；而对于 Python 而言，由 PEP 484 在标准库中引入的 `typing` 模块也可见一斑。

好好的动态类型，为什么都要朝着静态类型发展呢？

一言以概之：“**动态类型写着爽，维护升级火葬场**”。

就以我们的主角 Python 而言，在其发展早期，它只不过是作为一个小小的工具语言，用以在大型项目中的边边角角做一些简单的辅助性处理。那时的 Python，一般也就是个两三千行撑死，莫说是复杂的类型结构，很可能连 `class` 语句都不会出现。在这种基本就是内置类型传来传去的情况下，动态类型让老练的开发者能够快速高效地完成简单的工作，一时之间受到了大家的追捧。

伴随着 Python 的流行，越来越多的项目采用 Python 作为主语言。而当一个项目拥有过万行的代码量以及完整而精巧的类型结构设计时，动态类型的弊端就开始显现。函数签名的自解释性匮乏，开发者不得不“面向文档编程”；错误的类型传递无法避免，隐藏的 BUG 开始积累；类型的含义逐步被淡化，精心构造的类型结构失去价值……这时候，大家又开始怀念起了静态类型的好，写代码时多跳几个 `type error` 总好过 DEBUG 时抓心挠肺。

在这段时期，一个名叫 mypy 的第三方包的兴起令 Python 核心开发者意识到了真正的问题所在：**码农们对于静态类型的偏好其实源于对类型检查器的依赖**。mypy 就是一个静态类型检查器，通过解析 Python 代码及其中包含的特定的注释，在 Python 代码实际运行前进行静态的类型检查。而这是一个兼容性非常棒的解决方案：类型声明的内容均包含在注释中，不会对原有的代码含义产生任何影响；类型检查独立工作在代码运行前，不会对代码的实际运行产生任何影响。2014 年 9 月 29 日，Python 之父 Guido van Rossum、mypy 之父 Jukka Lehtosalo 及 Python 核心开发者 Łukasz Langa 联手发布了 [PEP 484 -- Type Hints](https://www.python.org/dev/peps/pep-0484/) ，在 Python 标准库中引入了全新的 `typing` 模块，提供对类型标注的官方支持**。**&#x20;

时至今日，类型标注已经成为 Python 的核心语言特性之一，但相关的中文资料多是只言片语，缺乏系统性的介绍，故开此篇以作补充。

{% hint style="info" %}
本文内容基于 Python 3.10.2
{% endhint %}

## 引入类型标注的动机

Python 项目中需要引入类型检查以维持项目的可持续发展，但是不应为此改变 Python 自诞生以来的动态语言特征。而社区中已有被广泛接受的解决方案：在源代码中使用注释补充变量信息，检查器在代码运行前进行（静态）类型检查。

静态类型检查的高度实用性与优雅性赢得了整个 Python 社区的青睐，故而 Python 核心开发者们决定在核心语法特性中引入官方支持。

但需要强调的是，**类型标注** 与静态类型语言中的 _变量声明_ 不尽相同，且这不会动摇 Python 一直以来作为动态类型语言的本质与设计哲学。

> **Python 将仍然是一种动态类型的语言，作者不希望强制类型提示，即使按照惯例也是如此。**
>
> &#x20;                                                                                                                                                                      **——PEP 484**

> **类型注释不应与静态类型语言中的变量声明相混淆。注释语法的目标是提供一种简单的方法来为第三方工具指定结构化类型元数据。**
>
> &#x20;                                                                                                                                                                      **——PEP 526**

## 类型标注的理论基础

Python 所采用的类型标注运用了 **渐进式类型（Gradual Typing）** 理论。

常见编程语言对类型检查的实现，存在两个派别——动态类型检查与静态类型检查。

动态类型检查的代表性语言包括 Perl、Python、Javascript、Ruby 和 PHP，“动态”的含义是类型检查发生在程序代码实际执行期间，即对内存中已经存在的数据进行操作时才对其类型进行检查。

静态类型检查的代表性语言包括 Java、C#、C 和 C++，“静态”的含义是类型检查发生在程序代码实际执行之前，此时内存中并没有实际生成对应数据。

这两派的支持者间关于谁优谁劣的争论已经持续了数十年，至今仍为得出一致的结论。

{% hint style="info" %}
个人认为这场争论的核心是围绕 **开发效率、运行效率、维护效率** 的取舍：动态类型检查以损失运行效率和维护效率为代价，大大提升了开发效率；静态类型检查以较大的开发效率损失换来了更高的运行效率和维护效率。尽管偏好相异，但二者都对开发效率做出了相对极端的取舍。
{% endhint %}

[Jeremy Siek](https://wphomes.soic.indiana.edu/jsiek/) 与 Walid Taha 联手在 2006 年提出了渐进式类型理论，该理论结合了动态类型检查和静态类型检查







区分类与类型

类型继承与子类型







## 语法详解

在介绍语法之前，我们需要先区分两个易混淆的概念—— **类** 和 **类型。**

在 Python 中，类是由 `class` 语句定义的对象工厂，并由 `type(obj)` 内置函数返回。类是一个动态的、运行时的概念。具体可参见作者另一篇博文 [你真的“创建”了类吗？](create-class.md)

类型是用于描述对象特征的抽象概念，拥有一系列相同的功能、值的对象可以归为一个类型。类型是一个静态的、逻辑性的概念。

{% hint style="danger" %}
“类型标注”这个名称已经明确表明了其标注的内容是类型而非类，所以一定要注意，**应将类型标注中使用的名称作为类型而非简单的类来看待**。
{% endhint %}

### 变量

### 函数

### 方法



## 高级特性

### 泛型

### TypeGuard

###



## 参考

1. [typing --- 类型提示支持](https://docs.python.org/zh-cn/3/library/typing.html)
2. [PEP 483 -- The Theory of Type Hints](https://www.python.org/dev/peps/pep-0483/)
3. [PEP 484 -- Type Hints](https://www.python.org/dev/peps/pep-0484/)
4. [PEP 526 -- Syntax for Variable Annotations](https://www.python.org/dev/peps/pep-0526/)
5. [PEP 544 -- Protocols: Structural subtyping (static duck typing)](https://www.python.org/dev/peps/pep-0544/)
6. [PEP 585 -- Type Hinting Generics In Standard Collections](https://www.python.org/dev/peps/pep-0585/)
7. [PEP 586 -- Literal Types](https://www.python.org/dev/peps/pep-0586/)
8. [PEP 589 -- TypedDict: Type Hints for Dictionaries with a Fixed Set of Keys](https://www.python.org/dev/peps/pep-0589/)
9. [PEP 591 -- Adding a final qualifier to typing](https://www.python.org/dev/peps/pep-0591/)
10. [PEP 593 -- Flexible function and variable annotations](https://www.python.org/dev/peps/pep-0593/)
11. [PEP 604 -- Allow writing union types as X | Y](https://www.python.org/dev/peps/pep-0604/)
12. [PEP 612 -- Parameter Specification Variables](https://www.python.org/dev/peps/pep-0612/)
13. [PEP 613 -- Explicit Type Aliases](https://www.python.org/dev/peps/pep-0613/)
14. [PEP 647 -- User-Defined Type Guards](https://www.python.org/dev/peps/pep-0647/)
15. [What is Gradual Typing](https://wphomes.soic.indiana.edu/jsiek/what-is-gradual-typing/)
