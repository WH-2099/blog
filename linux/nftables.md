# nftables

## 简介

> Nftables 是现代 Linux 内核数据包分类框架。新的代码应该使用它来代替遗留的 `{ip, ip6, arp, eb} _ tables (xtables)` 基础设施。对于尚未转换的现有代码库，遗留的 xtables 基础设施将维护到到 2021 年。

在 Linux 服务器的日常维护中，网络防火墙是很重要的一环。对于一般用途而言，其实需要的防火墙策略都不是很复杂，没有必要为此而购买独立的软硬件防火墙，使用 Linux 内嵌的基础网络包处理就完全足够应对。

iptables 作为这方面的老大哥，在 Linux 系统的基础网络包处理一线坚挺了多年。但随着时代的发展，老大哥也到了该退休的岁数，目前公认的继任者就是 nftables 。

{% hint style="info" %}
关于 nftables 的具体简介可以参照这里 [Waht is nftables?](https://wiki.nftables.org/wiki-nftables/index.php/What_is_nftables%3F)
{% endhint %}

## 快速入门

[Quick reference-nftables in 10 minutes](https://wiki.nftables.org/wiki-nftables/index.php/Quick_reference-nftables_in_10_minutes)

### 常用命令

#### 基础命令

* `nft list` 查看
* `nft create` 创建
* `nft add` 添加
* `nft insert` 插入
* `nft replace` 替换
* `nft delete` 删除
* `nft flush` 清空

{% hint style="info" %}
`add` 和 `create` 的唯一区别是，如果指定的内容已经存在，`add` 不会返回错误，而 `create` 会返回错误。
{% endhint %}

#### 命令选项

* `-n, --numeric` 打印完整的数字输出
* `-c, --check` 检查命令的有效性，而不实际应用更改
* `-a, --handle` 在输出中显示对象句柄
* `-e, --echo` 当使用 `add`，`insert` 或 `replace` 命令将项目插入到规则集时，就像 `nft monitor` 一样打印通知
* `-I, --includepath directory` 在要搜索包含的文件目录列表中添加 `directory`。这个选项可以被多次指定
* `-f, --file filename` 从 `filename` 读取输入。如果 `filename` 是 `-`，则从 stdin 读取
* `-i, --interactive` 从一个交互式的 readline CLI 中读取输入。你可以使用 `quit` 来退出，或者使用 EOF 标记，通常是 `CTRL-D`



#### family

* `ip`
* `arp`
* `ip6`
* `bridge`
* `inet`
* `netdev`

## 参考图表

### Hook 触发流程

![](../.gitbook/assets/nf-hooks.png)



### Hook 可用性

{% hint style="warning" %}
没有明确标为 Yes 的都不可用。
{% endhint %}

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>Chain type</b>
      </th>
      <th style="text-align:center">ingress</th>
      <th style="text-align:center">prerouting</th>
      <th style="text-align:center">forward</th>
      <th style="text-align:center">input</th>
      <th style="text-align:center">output</th>
      <th style="text-align:center">postrouting</th>
      <th style="text-align:center">egress</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <p></p>
        <p><b>inet family</b>
        </p>
      </td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
    </tr>
    <tr>
      <td style="text-align:left">filter</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center"></td>
    </tr>
    <tr>
      <td style="text-align:left">nat</td>
      <td style="text-align:center"></td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center"></td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center"></td>
    </tr>
    <tr>
      <td style="text-align:left">route</td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
    </tr>
    <tr>
      <td style="text-align:left">
        <br /><b>ip6 family</b>
      </td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
    </tr>
    <tr>
      <td style="text-align:left">filter</td>
      <td style="text-align:center"></td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center"></td>
    </tr>
    <tr>
      <td style="text-align:left">nat</td>
      <td style="text-align:center"></td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center"></td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center"></td>
    </tr>
    <tr>
      <td style="text-align:left">route</td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
    </tr>
    <tr>
      <td style="text-align:left">
        <br /><b>ip family</b>
      </td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
    </tr>
    <tr>
      <td style="text-align:left">filter</td>
      <td style="text-align:center"></td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center"></td>
    </tr>
    <tr>
      <td style="text-align:left">nat</td>
      <td style="text-align:center"></td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center"></td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center"></td>
    </tr>
    <tr>
      <td style="text-align:left">route</td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
    </tr>
    <tr>
      <td style="text-align:left">
        <br /><b>arp family</b>
      </td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
    </tr>
    <tr>
      <td style="text-align:left">filter</td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
    </tr>
    <tr>
      <td style="text-align:left">
        <br /><b>bridge family</b>
      </td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
    </tr>
    <tr>
      <td style="text-align:left">filter</td>
      <td style="text-align:center"></td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center"></td>
    </tr>
    <tr>
      <td style="text-align:left">
        <br /><b>netdev family</b>
      </td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
    </tr>
    <tr>
      <td style="text-align:left">filter</td>
      <td style="text-align:center">Yes</td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center"></td>
      <td style="text-align:center">Yes(5.7)</td>
    </tr>
  </tbody>
</table>









| nftables [Families](https://wiki.nftables.org/wiki-nftables/index.php/Nftables_families) | Typical hooks | nft Keyword | Value | Netfilter Internal Priority | Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
|  | prerouting |  | -450 | NF\_IP\_PRI\_RAW\_BEFORE\_DEFRAG |  |
| inet, ip, ip6 | prerouting |  | -400 | NF\_IP\_PRI\_CONNTRACK\_DEFRAG | Packet defragmentation / datagram reassembly |
| inet, ip, ip6 | all | **raw** | -300 | NF\_IP\_PRI\_RAW | Traditional priority of the raw table placed before connection tracking operation |
|  |  |  | -225 | NF\_IP\_PRI\_SELINUX\_FIRST | SELinux operations |
| inet, ip, ip6 | prerouting, output |  | -200 | NF\_IP\_PRI\_CONNTRACK | [Connection tracking](https://wiki.nftables.org/wiki-nftables/index.php/Connection_Tracking_System) processes run early in prerouting and output hooks to associate packets with tracked connections. |
| inet, ip, ip6 | all | **mangle** | -150 | NF\_IP\_PRI\_MANGLE | Mangle operation |
| inet, ip, ip6 | prerouting | **dstnat** | -100 | NF\_IP\_PRI\_NAT\_DST | Destination NAT |
| inet, ip, ip6, arp, netdev | all | **filter** | 0 | NF\_IP\_PRI\_FILTER | Filtering operation, the filter table |
| inet, ip, ip6 | all | **security** | 50 | NF\_IP\_PRI\_SECURITY | Place of security table, where secmark can be set for example |
| inet, ip, ip6 | postrouting | **srcnat** | 100 | NF\_IP\_PRI\_NAT\_SRC | Source NAT |
|  | postrouting |  | 225 | NF\_IP\_PRI\_SELINUX\_LAST | SELinux at packet exit |
| inet, ip, ip6 | postrouting |  | 300 | NF\_IP\_PRI\_CONNTRACK\_HELPER | Connection tracking helpers, which identify expected and related packets. |
| inet, ip, ip6 | input, postrouting |  | INT\_MAX | NF\_IP\_PRI\_CONNTRACK\_CONFIRM | Connection tracking adds new tracked connections at final step in input & postrouting hooks. |
|  |  |  |  |  |  |
| bridge | prerouting | **dstnat** | -300 | NF\_BR\_PRI\_NAT\_DST\_BRIDGED |  |
| bridge | all | **filter** | -200 | NF\_BR\_PRI\_FILTER\_BRIDGED |  |
| bridge |  |  | 0 | NF\_BR\_PRI\_BRNF |  |
| bridge | output | **out** | 100 | NF\_BR\_PRI\_NAT\_DST\_OTHER |  |
| bridge |  |  | 200 | NF\_BR\_PRI\_FILTER\_OTHER |  |
| bridge | postrouting | **srcnat** | 300 | NF\_BR\_PRI\_NAT\_SRC |  |

## 参考源

1. [nftables wiki](https://wiki.nftables.org/wiki-nftables/index.php/Main_Page)
2. [nftables manpage](https://www.netfilter.org/projects/nftables/manpage.html)

