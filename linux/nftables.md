---
description: 功能确实很强大，但这也太复杂了
---

# nftables

## 简介

> Nftables 是现代 Linux 内核数据包分类框架。新的代码应该使用它来代替遗留的 `{ip, ip6, arp, eb}_tables (xtables)` 基础设施。对于尚未转换的现有代码库，遗留的 xtables 基础设施将维护到到 2021 年。

在 Linux 服务器的日常维护中，网络防火墙是很重要的一环。对于一般用途而言，其实需要的防火墙策略都不是很复杂，没有必要为此而购买独立的软硬件防火墙，使用 Linux 内嵌的基础网络包处理就完全足够应对。

iptables 作为这方面的老大哥，在 Linux 系统的基础网络包处理一线坚挺了多年。但随着时代的发展，老大哥也到了该退休的岁数，目前公认的继任者就是 nftables。

相较于 iptables，nftables 的框架设计更为合理，功能更为全面，命令行的语法也更为清晰。但从网络上关于 nftables 的资料并不多这一点就不难看出，nftables 目前的应用仍不是非常广泛。（和 IPv6 同病相怜）这主要是因为 nftables 丰富的功能和特性立足于陡峭的学习曲线之上，令用户望而却步。

{% hint style="info" %}
关于 nftables 的具体简介可以参照这里 [Waht is nftables?](https://wiki.nftables.org/wiki-nftables/index.php/What\_is\_nftables%3F)
{% endhint %}

## 命令

```
nft 选项 动作 对象
```

### 动作

```
基础动作 动作目标
```

#### 选项

* `-n, --numeric` 打印完整的数字输出
* `-c, --check` 检查命令的有效性，而不实际应用更改
* `-a, --handle` 在输出中显示对象句柄
* `-e, --echo` 当使用 `add`，`insert` 或 `replace` 命令将项目插入到规则集时，就像 `nft monitor` 一样打印通知
* `-I, --includepath directory` 在要搜索包含的文件目录列表中添加 _directory_。这个选项可以被多次指定
* `-f, --file filename` 从 _filename_ 读取输入。如果 _filename_ 是 `-`，则从 stdin 读取
* `-i, --interactive` 从一个交互式的 readline CLI 中读取输入。你可以使用 `quit` 来退出，或者使用 EOF 标记，通常是 `CTRL-D`

#### 基础动作

* **`list`** 查看
* **`create`** 创建
* **`add`** 添加
* **`insert`** 插入
* **`replace`** 替换
* **`delete`** 删除
* **`flush`** 清空

{% hint style="info" %}
`add` 和 `create` 的唯一区别是，如果指定的内容已经存在，`add` 不会返回错误，而 `create` 会返回错误。
{% endhint %}

#### 动作目标

* **`ruleset`**
* **`table`**
* **`chain`**
* **`rule`**
* **`set`**
* **`map`**
* **`element`**

{% hint style="info" %}
动作目标与对象是对应关系。
{% endhint %}

## 对象

### Address Family 地址族

地址族决定处理的数据包的类型。对于每个地址族，内核在数据包处理路径的特定阶段包含所谓的钩子，如果存在这些钩子的规则，这些钩子将调用 nftables。

* **`ip`** IPv4，**默认**
* **`ip6`** IPv6
* **`inet`** Internet （IPv4/IPv6）
* **`arp`** 处理 IPv4 ARP 数据包
* **`bridge`** 处理通过网桥设备的数据包
* **`netdev`** 处理来自入口的数据包

###

### Ruleset 规则集

`ruleset` 关键字用于标识内核中当前到位的整个表、链等集合。

```
{list | flush} ruleset [family]
```

### Tables 表

表是链、集和有状态对象的容器，通过地址族家庭和名称来识别。

```
{add | create} table [family] table [{ flags flags ; }]
{delete | list | flush} table [family] table
list tables [family]
delete table [family] handle handle
```

### Chain 链

链条是规则的容器，分为基链和规则链两种。

```
{add | create} chain [family] table chain [{ type type hook hook [device device] priority priority ; [policy policy ;] }]
{delete | list | flush} chain [family] table chain
list chains [family]
delete chain [family] table handle handle
rename chain [family] table chain newname
```

#### **基本链**

基本链是网络堆栈中数据包的入口点，必须包含类型、钩子和优先级参数。

**类型**

* `filter` 过滤时使用的标准类型。
* `nat` 基于连接跟踪条目执行地址转换。只有连接的第一个数据包实际穿越该链——其规则通常定义所创建的连接跟踪条目的细节（例如 NAT 语句）。
* `route` 如果一个数据包已经穿越了这种类型的链，并且即将被接受，如果 IP 头的相关部分发生了变化，就会进行新的路由查询。这允许在 nftables 中实现策略路由选择器。

**钩子**

* `prerouting` 刚到达并未被 nftables 的其他部分所路由或处理的数据包。
* `input` 已经被接收并且已经经过 `prerouting` 钩子的传入数据包。
* `forward` 如果数据报将被发送到另一个设备，它将会通过 `forward` 钩子。
* `output` 从本地传出的数据包。
* `postrouting` 仅仅在离开系统之前，可以对数据包进行进一步处理。

**优先级**

优先级参数接受一个带符号的整数值或一个标准的优先级名称，该名称指定具有相同钩子值的链的遍历顺序。排序是升序的，也就是说低优先级的值优先于高优先级的值。标准的优先级值可以用容易记忆的名称替换。并不是所有的名字在每个钩子族中都有意义 (参见 [钩子可用性](nftables.md#gou-zi-ke-yong-xing)) ，但是它们的数值仍然可以用来排列链的优先级。

* `raw`
* `mangle`
* `dstnat`
* `filter`
* `security`
* `srcnat`

#### 规则链

规则链可以作为跳转目标，用于更好地组织规则。

### Rule 规则

规则是根据一套语法规则由两类成分构成的：表达式和声明。

```
{add | insert} rule [family] table chain [handle handle | index index] statement ... [comment comment]
replace rule [family] table chain handle handle statement ... [comment comment]
delete rule [family] table chain handle handle
```

{% hint style="warning" %}
规则内容过于冗杂，建议直接阅读 [Quick reference-nftables in 10 minutes](https://wiki.nftables.org/wiki-nftables/index.php/Quick\_reference-nftables\_in\_10\_minutes)
{% endhint %}

## 输入文件

* `\` 如果上一行的最后一个字符是非引号反斜杠，则下一行被视为延续
* `;` 将同一行中的多个命令分开
* `#` 后面的同一行字符都被忽略
* `include filename` 包含 _filename_ 文件的内容
* `define variable = expr` 定义值为 _expr_ 的变量 _variable_
* `$variable` 引用变量 _variable_ 的值

{% hint style="info" %}
标识符以字母字符 `[a-zA-Z]` 开头，后面跟着零个或多个字母数字字符 `[a-zA-Z0-9]` 和字符斜杠 `[/]` 、反斜杠 `[\\]`、下划线 `[_]` 和点 `[\.]` 。使用不同字符或与关键字冲突的标识符需要用双引号 `["]` 括起来。
{% endhint %}

## 通用容器

### Set 集合

nftables 提供了两种集合概念：匿名集和命名集。

#### 匿名集

匿名集是没有特定名称的集合。集合成员用花括号括起来，在创建集合使用的规则时，用逗号分隔元素。一旦该规则被删除，集合也会被删除。它们不能被更新，也就是说，一旦一个匿名集被声明，它就不能再被更改，除非删除/更改使用匿名集的规则。

#### 命名集

```
add set [family] table set { type type | typeof expression ; [flags flags ;] [timeout timeout ;] [gc-interval gc-interval ;] [elements = { element[, ...] } ;] [size size ;] [policy policy ;] [auto-merge ;] }
{delete | list | flush} set [family] table set
list sets [family]
delete set [family] table handle handle
{add | delete} element [family] table set { element[, ...] }
```

| **Keyword** | **Description**                         | **Type**                                                                      |
| ----------- | --------------------------------------- | ----------------------------------------------------------------------------- |
| type        | 集合中元素的数据类型                              | string: ipv4\_addr, ipv6\_addr, ether\_addr, inet\_proto, inet\_service, mark |
| typeof      | 集合中元素的数据类型                              | 要用于获取类型的表达式                                                                   |
| flags       | 设置标志                                    | string: constant, dynamic, interval, timeout                                  |
| timeout     | 一个元素停留在集合中的时间，如果集合被添加到数据包路径（规则集），则是强制性的 | string, decimal followed by unit. Units are: d, h, m, s                       |
| gc-interval |                                         |                                                                               |
|             |                                         |                                                                               |
|             | 垃圾收集时间间隔，只有在超时或标志超时激活时才可用               | string, decimal followed by unit. Units are: d, h, m, s                       |
| elements    | 集合包含的元素                                 | 集合的数据类型                                                                       |
| size        | 集合中的最大元素数，如果集合是由数据包路径（规则集）添加的，则是强制性的    | unsigned integer (64 bit)                                                     |
| policy      | 设置策略                                    | string: performance \[default], memory                                        |
| auto-merge  | 自动合并相邻/重叠的集合元素（仅适用于区间集合）                |                                                                               |

### Map 映射

映射根据用作输入的某个特定键存储数据。它们由用户定义的名称唯一标识并附加到表中。

```
add map [family] table map { type type | typeof expression [flags flags ;] [elements = { element[, ...] } ;] [size size ;] [policy policy ;] }
{delete | list | flush} map [family] table map
list maps [family]
```

| Keyword  | Description | Type                                                                                                        |
| -------- | ----------- | ----------------------------------------------------------------------------------------------------------- |
| type     | 映射中元素的数据类型  | string: ipv4\_addr, ipv6\_addr, ether\_addr, inet\_proto, inet\_service, mark, counter, quota 计数器和配额不能作为键使用 |
| typeof   | 映射中元素的数据类型  | 要用于获取类型的表达式                                                                                                 |
| flags    | 设置标志        | string: constant, interval                                                                                  |
| elements | 映射包含的元素     | 映射的数据类型                                                                                                     |
| size     | 映射中元素的最大数量  | string: performance \[default], memory                                                                      |
| policy   | 映射策略        | string: performance \[default], memory                                                                      |

### Element 元素

与元素相关的命令允许更改命名集和映射的内容。

```
{add | create | delete | get } element [family] table set { ELEMENT[, ...] }

ELEMENT := key_expression OPTIONS [: value_expression]
OPTIONS := [timeout TIMESPEC] [expires TIMESPEC] [comment string]
TIMESPEC := [numd][numh][numm][num[s]]
```

| **Option** | **Description**     |
| ---------- | ------------------- |
| timeout    | 带有超时标志的集合/映射的超时值    |
| expires    | 给定元素过期的时间，仅对规则集复制有用 |
| comment    | 每个元素的注释字段           |

## 参考图表

#### 钩子触发流程

![](../.gitbook/assets/nf-hooks.png)

#### 钩子可用性

{% hint style="warning" %}
白名单列举，没有明确标为 Yes 的都不可用。
{% endhint %}

| Chain type                                | ingress | prerouting | forward | input | output | postrouting |  egress  |
| ----------------------------------------- | :-----: | :--------: | :-----: | :---: | :----: | :---------: | :------: |
| **inet family**                           |         |            |         |       |        |             |          |
| filter                                    |   Yes   |     Yes    |   Yes   |  Yes  |   Yes  |     Yes     |          |
| nat                                       |         |     Yes    |         |  Yes  |   Yes  |     Yes     |          |
| route                                     |         |            |         |       |   Yes  |             |          |
| <p><br><strong>ip6 family</strong></p>    |         |            |         |       |        |             |          |
| filter                                    |         |     Yes    |   Yes   |  Yes  |   Yes  |     Yes     |          |
| nat                                       |         |     Yes    |         |  Yes  |   Yes  |     Yes     |          |
| route                                     |         |            |         |       |   Yes  |             |          |
| <p><br><strong>ip family</strong></p>     |         |            |         |       |        |             |          |
| filter                                    |         |     Yes    |   Yes   |  Yes  |   Yes  |     Yes     |          |
| nat                                       |         |     Yes    |         |  Yes  |   Yes  |     Yes     |          |
| route                                     |         |            |         |       |   Yes  |             |          |
| <p><br><strong>arp family</strong></p>    |         |            |         |       |        |             |          |
| filter                                    |         |            |         |  Yes  |   Yes  |             |          |
| <p><br><strong>bridge family</strong></p> |         |            |         |       |        |             |          |
| filter                                    |         |     Yes    |   Yes   |  Yes  |   Yes  |     Yes     |          |
| <p><br><strong>netdev family</strong></p> |         |            |         |       |        |             |          |
| filter                                    |   Yes   |            |         |       |        |             | Yes(5.7) |

#### 钩内优先级

{% hint style="info" %}
在给定的钩子中，会按照数字递增的顺序执行操作，所以 **数字越小优先级越高** 。
{% endhint %}

| nftables [Families](https://wiki.nftables.org/wiki-nftables/index.php/Nftables\_families) | Typical hooks      | nft Keyword  | Value    | Netfilter Internal Priority      | Description                                                                                                                                                                                             |
| ----------------------------------------------------------------------------------------- | ------------------ | ------------ | -------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|                                                                                           | prerouting         |              | -450     | NF\_IP\_PRI\_RAW\_BEFORE\_DEFRAG |                                                                                                                                                                                                         |
| inet, ip, ip6                                                                             | prerouting         |              | -400     | NF\_IP\_PRI\_CONNTRACK\_DEFRAG   | Packet defragmentation / datagram reassembly                                                                                                                                                            |
| inet, ip, ip6                                                                             | all                | **raw**      | -300     | NF\_IP\_PRI\_RAW                 | Traditional priority of the raw table placed before connection tracking operation                                                                                                                       |
|                                                                                           |                    |              | -225     | NF\_IP\_PRI\_SELINUX\_FIRST      | SELinux operations                                                                                                                                                                                      |
| inet, ip, ip6                                                                             | prerouting, output |              | -200     | NF\_IP\_PRI\_CONNTRACK           | [Connection tracking](https://wiki.nftables.org/wiki-nftables/index.php/Connection\_Tracking\_System) processes run early in prerouting and output hooks to associate packets with tracked connections. |
| inet, ip, ip6                                                                             | all                | **mangle**   | -150     | NF\_IP\_PRI\_MANGLE              | Mangle operation                                                                                                                                                                                        |
| inet, ip, ip6                                                                             | prerouting         | **dstnat**   | -100     | NF\_IP\_PRI\_NAT\_DST            | Destination NAT                                                                                                                                                                                         |
| inet, ip, ip6, arp, netdev                                                                | all                | **filter**   | 0        | NF\_IP\_PRI\_FILTER              | Filtering operation, the filter table                                                                                                                                                                   |
| inet, ip, ip6                                                                             | all                | **security** | 50       | NF\_IP\_PRI\_SECURITY            | Place of security table, where secmark can be set for example                                                                                                                                           |
| inet, ip, ip6                                                                             | postrouting        | **srcnat**   | 100      | NF\_IP\_PRI\_NAT\_SRC            | Source NAT                                                                                                                                                                                              |
|                                                                                           | postrouting        |              | 225      | NF\_IP\_PRI\_SELINUX\_LAST       | SELinux at packet exit                                                                                                                                                                                  |
| inet, ip, ip6                                                                             | postrouting        |              | 300      | NF\_IP\_PRI\_CONNTRACK\_HELPER   | Connection tracking helpers, which identify expected and related packets.                                                                                                                               |
| inet, ip, ip6                                                                             | input, postrouting |              | INT\_MAX | NF\_IP\_PRI\_CONNTRACK\_CONFIRM  | Connection tracking adds new tracked connections at final step in input & postrouting hooks.                                                                                                            |
|                                                                                           |                    |              |          |                                  |                                                                                                                                                                                                         |
| bridge                                                                                    | prerouting         | **dstnat**   | -300     | NF\_BR\_PRI\_NAT\_DST\_BRIDGED   |                                                                                                                                                                                                         |
| bridge                                                                                    | all                | **filter**   | -200     | NF\_BR\_PRI\_FILTER\_BRIDGED     |                                                                                                                                                                                                         |
| bridge                                                                                    |                    |              | 0        | NF\_BR\_PRI\_BRNF                |                                                                                                                                                                                                         |
| bridge                                                                                    | output             | **out**      | 100      | NF\_BR\_PRI\_NAT\_DST\_OTHER     |                                                                                                                                                                                                         |
| bridge                                                                                    |                    |              | 200      | NF\_BR\_PRI\_FILTER\_OTHER       |                                                                                                                                                                                                         |
| bridge                                                                                    | postrouting        | **srcnat**   | 300      | NF\_BR\_PRI\_NAT\_SRC            |                                                                                                                                                                                                         |

## 参考源

1. [nftables wiki](https://wiki.nftables.org/wiki-nftables/index.php/Main\_Page)
2. [nftables manpage](https://www.netfilter.org/projects/nftables/manpage.html)
3. [Quick reference-nftables in 10 minutes](https://wiki.nftables.org/wiki-nftables/index.php/Quick\_reference-nftables\_in\_10\_minutes)
