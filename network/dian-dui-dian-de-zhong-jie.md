---
description: PPPoE Relay 原理解析
---

# 点对点的中介

&#x20;某可爱四年内第 N 次提及 PPPoE-Relay 的原理，终于让我的好奇心冲破了发际线。

是时候狠心读一波 RFC 终结这个话题了！

## 了解 PPP 和 Ethernet

### PPP (Point-to-Point Protocol)

1. 用于点对点通信的通用协议。
2. 设计为工作在网络层（常见的 IP 协议）之下。
3. 采用 C/S 架构。
4. 内含 **Link Control Protocol**，提供身份验证功能。
5. 包含网络层配置功能

### **Ethernet**

1. 以太网，常见的链路层协议。
2. 平常说的 MAC 地址就是该协议的一部分。&#x20;

## 了解 PPPoE

### PPPoE (PPP Over Ethernet)

1. 工作在以太网之上的 PPP。
2. 继承了 PPP 的所有功能
3. 在 ADSL 时代就普遍使用，当时被冠以 _拨号连接_ 之名，沿用至今。



PPP 设计上需要客户端和服务器在同一广播域内，故 PPPoE 只能在同一以太网域下工作。

## 参考源

1. [RFC 2516 - A Method for Transmitting PPP Over Ethernet (PPPoE)](https://datatracker.ietf.org/doc/html/rfc2516)
2. [pppoe-relay(8) - Linux man page](https://linux.die.net/man/8/pppoe-relay)
3. [10张图带你搞懂数据链路层PPP点到点协议](https://zhuanlan.zhihu.com/p/153515394)
