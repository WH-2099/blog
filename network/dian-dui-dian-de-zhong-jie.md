---
description: PPPoE Relay 原理解析
---

# 点对点的中介

某可爱四年内第 N 次提及 PPPoE-Relay 的原理，终于让我的好奇心冲破了发际线。

是时候狠心读一波 RFC 终结这个话题了！

## 了解 PPP 和 Ethernet

### PPP (Point-to-Point Protocol)

1. 用于点对点通信的通用协议。
2. 设计为工作在网络层（常见的 IP 协议）之下。
3. 采用 C/S 架构。
4. 内含 **Link Control Protocol**，提供身份验证功能。
5. 内含 **Network Control Protocol**，可完成对网络层（IP）的基础配置。

### **Ethernet**

1. 以太网，常见的链路层协议。
2. 平常说的 MAC 地址就是该协议的一部分。&#x20;

## 了解 PPPoE

### PPPoE (PPP Over Ethernet)

1. 工作在以太网之上的 PPP。
2. 继承了 PPP 的所有功能
3. 在 ADSL 时代就普遍使用，当时被冠以 _拨号连接_ 之名，沿用至今。

> 以太网数据包【PPPoE 数据包（IP 数据包）】

### 为何 ISP 独爱 PPPoE

1. 点对点连接一定程度上提升了用户网络的安全性（例如不使用 ARP，从根本上防御了 ARP 攻击）。
2. 便于计时计量，结合身份验证可完成计费。
3. 可以完成网络层配置的任务（不再需要 DHCP）。

### PPP 如何 oE ？

PPP 协议建立在客户端与服务器能够稳定双工通信的前提下。

就以太网而言，这需要客户端与服务端互相知晓对方的 MAC 地址，所以在 PPP 协议之前额外增加了 **Discovery Stage** 过程，在此过程中有以下 5 种数据包：

1. The PPPoE Active Discovery Initiation (PADI) packet\
   客户端向本地以太网域广播该包。
2. The PPPoE Active Discovery Offer (PADO) packet\
   服务器向客户端单播回应，可能存在多个服务器，故此时客户端可能收到多个包。
3. The PPPoE Active Discovery Request (PADR) packet\
   客户端选择一个服务器，单播发送该包，请求会话。
4. The PPPoE Active Discovery Session-confirmation (PADS) packet\
   服务器向客户端单播回应，确认会话开始。
5. The PPPoE Active Discovery Terminate (PADT) packet\
   客户端和服务器均可单播发送该包，终止会话。

## PPPoE Relay

### 功能

让处于不同以太网域（如被路由器隔离的 WAN 侧和 LAN 侧）中的客户端和服务器也能够建立 PPPoE 会话。

### 场景

运营商提供给你一个以太网接入方式（光猫桥接），你首先把路由器 WAN 口与运营商网络连接，用路由器拨号，而后又将电脑连接到路由器的 LAN 口。

没有其他设置时，电脑（客户端）启动 PPPoE 后，进入 Discovery Stage，发送的 PADI 数据包只能在 LAN 域中广播，无法到达运营商位于 WAN 域中的服务器。

{% hint style="info" %}
路由器作为网络层设备，它将划分 WAN 和 LAN 两个链路层域（以太网域）。
{% endhint %}

在路由器启用 PPPoE Relay 后，它会探测并识别 Discovery Stage 产生的数据包，并进行特定的标记处理和转发，保证客户端和服务器之间能够正常收发数据。此时电脑（客户端）可以与服务端完成 Discovery Stage 并开始下一阶段的 PPP 协议。

{% hint style="warning" %}
PPP 协议比较复杂，要经历 参数协商 -> 身份认证 -> 网络层配置 三个主要阶段。\
PPPoE Relay 只保证 PPP 协议能够运行，但若是出现身份认证不通过、网络配置冲突等问题，最终还是会导致整个 PPPoE 失败。
{% endhint %}

### 原理

侦测并处理 Discovery Stage 阶段的 5 类数据包，具体处理如下。

后文将 PPPoE Relay 称为 **中继**。

#### PADI

中继在 PPPoE 数据包中添加 `Relay-Session-ID` 标记用于追踪对话。这个标记是 RFC 中规定的，客户端和服务器若收到了带有这个标记的数据包，它们必须在响应中也包含相同的标记：

> 0x0110 Relay-Session-Id
>
> This TAG MAY be added to any discovery packet by an intermediate agent that is relaying traffic. The TAG\_VALUE is opaque to both the Host and the Access Concentrator. If either the Host or Access Concentrator receives this TAG they MUST include it unmodified in any discovery packet they send as a response. All PADI packets MUST guarantee sufficient room for the addition of a Relay-Session-Id TAG with a TAG\_VALUE length of 12 octets.
>
> A Relay-Session-Id TAG MUST NOT be added if the discovery packet already contains one. In that case the intermediate agent SHOULD use the existing Relay-Session-Id TAG. If it can not use the existing TAG or there is insufficient room to add a Relay- Session-Id TAG, then it SHOULD return a Generic-Error TAG to the sender.

在添加标记后，中继会将 PADI 数据包广播到服务器所在的以太网域（一般是 WAN）。

#### PADO

\#TODO

#### PADR

\#TODO

#### PADS

\#TODO

#### PADT

\#TODO

## 参考源

1. [RFC 2516 - A Method for Transmitting PPP Over Ethernet (PPPoE)](https://datatracker.ietf.org/doc/html/rfc2516)
2. [pppoe-relay(8) - Linux man page](https://linux.die.net/man/8/pppoe-relay)
3. [10张图带你搞懂数据链路层PPP点到点协议](https://zhuanlan.zhihu.com/p/153515394)
