# 智能家居入门指南

- [智能家居入门指南](#智能家居入门指南)
  - [概念定义](#概念定义)
    - [国际观点](#国际观点)
    - [国内观点](#国内观点)
  - [行业概况](#行业概况)
    - [供需现状](#供需现状)
    - [全球龙头](#全球龙头)
    - [国内市场](#国内市场)
      - [产业链](#产业链)
      - [龙头企业](#龙头企业)
      - [市场前景](#市场前景)
  - [组网架构](#组网架构)
  - [产品实操](#产品实操)
    - [涂鸦简介](#涂鸦简介)
    - [APP 安装与登录](#app-安装与登录)
    - [有线网关](#有线网关)
      - [网关配网](#网关配网)
    - [八键开关](#八键开关)
      - [开关配网](#开关配网)
      - [多控关联](#多控关联)
      - [普通开关触发场景](#普通开关触发场景)
    - [八键语音开关](#八键语音开关)
      - [更改唤醒词](#更改唤醒词)
      - [普通按键语音名称](#普通按键语音名称)
      - [场景按键语音名称](#场景按键语音名称)
      - [继电器锁定设置](#继电器锁定设置)
      - [继电器解锁设置](#继电器解锁设置)
    - [雷达传感器](#雷达传感器)
      - [设置项说明](#设置项说明)
    - [背景音乐主机-P6](#背景音乐主机-p6)
      - [P6-主机绑定](#p6-主机绑定)
      - [P6-同步设备和场景](#p6-同步设备和场景)
      - [P6-歌单设置](#p6-歌单设置)
      - [P6-智能场景设置](#p6-智能场景设置)
    - [背景音乐主机-M10](#背景音乐主机-m10)
      - [M10-网关绑定](#m10-网关绑定)
      - [M10-同步设备和场景](#m10-同步设备和场景)
  - [参考源](#参考源)

## 概念定义

### 国际观点

> 智能家庭，或称智能家庭、**智能家居（英语：smart home）**，指建筑自动化的家庭。
>
> 而家庭自动化（英语：home automation 或 domotics）则指实现智能家庭的过程。
>
> 家庭自动化系统可以监控和/或控制灯光、窗户、温湿度等家庭设置，还可能可以控制家庭出入和触发警报器以保护家庭。

### 国内观点

![定义特征](image/%E5%AE%9A%E4%B9%89%E7%89%B9%E5%BE%81.png)

智能家居的定义经历了从 1.0 至 3.0 的变化：

- 1.0 时期智能家居通过感知技术与家居设备相连，帮助用户减少劳务量；
- 2.0 时期智能家居综合利用多项技术，通过家庭设施创造高效、安全、便捷的居住环境；
- 3.0 时期智能家居以人的需求为核心，随时提供个性化服务。

![分类明细](image/%E5%88%86%E7%B1%BB%E6%98%8E%E7%BB%86.png)

根据功能不同，智能家居可以分为以下十类：

- 智能照明
- 智能安防
- 智能控制
- 智能影音
- 智能传感
- 智能家电
- 智能设备
- 智能网络
- 智能遮晾
- 环境控制

## 行业概况

### 供需现状

![家庭数量](image/%E5%AE%B6%E5%BA%AD%E6%95%B0%E9%87%8F.png)

近年来，消费者对于智能家居产品的接受度不断提高，Statista 和 Strategy Analytics 公布数据显示，**2016-2021 年，全球拥有智能家居设备的家庭数量不断增长，2021 年达 2.6 亿户**。

![细分市场](image/%E7%BB%86%E5%88%86%E5%B8%82%E5%9C%BA.png)

分细分市场看，截至 2021 年，全球智能照明、智能安防及智能家居控制和链接用户数量均已超过 1 亿，能源管理、家庭娱乐和智能家电用户数量也超过 0.8 亿。

### 全球龙头

![alexa](image/alexa.jpg)
![google home](image/google-home.png)
![homekit](image/homekit.png)

目前世界上最多用户的平台分别为 amazon 的 **alexa**，google 的 **google home** 与 apple 的 **homekit**。为解决智能家庭空间碎片化的问题，google 与 amazon、apple 和 zigbee 共同推出了 *“project connected home over ip”* 计划。

### 国内市场

#### 产业链

![产业链图](image/%E4%BA%A7%E4%B8%9A%E9%93%BE%E5%9B%BE.png)

智能家居产业链上游为技术层和基础层，主要参与企业包括芯片、传感器、智能控制器等硬件供应商以及AI技术、云服务等软件供应商；中游为智能家居系统及设备的设计制造；下游为消费市场，主要参与者有房地产企业、家装企业、零售企业等。

#### 龙头企业

![国内龙头](image/%e5%9b%bd%e5%86%85%e9%be%99%e5%a4%b4.jpg)

目前，我国智能家居设备行业的主要企业包括 **阿里巴巴**、**小米科技**、**华为**、**美的集团**、**海尔智家**、**欧瑞博** 和 **绿米联创** 等等。

#### 市场前景

其中沪深股上市企业有 **美的集团（000333）**、**海尔智能（600690）**、**格力电器（000651）**、**长虹美菱（000521）**，**小米集团（1810.HK）** 和 **阿里巴巴（9988.HK）** 为港股上市公司，非上市企业主要有 **欧瑞博** 和 **绿米联创**。

![市场预测](image/%E5%B8%82%E5%9C%BA%E9%A2%84%E6%B5%8B.png)

我国智能家居行业正处于快速发展期，随着人工智能、5G 等技术的应用的愈发成熟、中国智能家居硬件设备的不断发展和消费者需求的提升，预计未来 5 年我国智能家居行业仍将处于快速发展阶段，到 2027 年，市场规模有望超过 1.1 万亿元。

![发展趋势](image/%E5%8F%91%E5%B1%95%E8%B6%8B%E5%8A%BF.png)

## 组网架构

根据设备间通信所采用技术的不同，常见的智能家居可以分为以下三类：

1. 电力载波通信
   - X-10
   - PLCBUS
2. 有线通信
   - RS-232
   - RS-485
   - CAN
   - LonWorks
   - KNX
3. 无线通信
   - 蓝牙
   - WiFi
   - ZigBee
   - Z-Wave
   - Thread
   - RF 射频技术
   - 移动通信网络（4G/5G）

当然，这三种通信技术也可简单的看做无线技术和有线技术，有线智能家居成本较低，并且系统研发简单，在实际使用中，相比起无线技术，它的数据传输速度高，并且不受外部环境的影响，抗干扰能力远远大于无线系统。但是，有线智能家居也有一些无可避免的缺陷，施工周期长，并且由于其布线较为繁琐，只适用于新装修的房屋，其次终端设备的位置是固定的，之后要挪动或者拓展的成本较高。相比之下，无线智能家居更为灵巧简便，可自动组网、可灵活地拓展和变更终端，并且随着无线通信技术的更新，在功耗、传输距离、传输速率、抗干扰方面具有很大提升。业界基本达成一致的观点是：无线技术比有线技术更具潜力，但就目前而言，无线技术仍有很长的一段发展路程要走。

![无线特点](image/%E6%97%A0%E7%BA%BF%E7%89%B9%E7%82%B9.png)

## 产品实操

这里选用**涂鸦**作为主要的互联平台，采用 **zigbee** 无线连接组网。

### 涂鸦简介

> **涂鸦智能（NYSE：TUYA，HKEX：2391）** 是一家致力于让生活更智能的领先技术公司，涂鸦提供能够智连万物的云平台，打造互联互通的开发标准，连接品牌、OEM 厂商、开发者、零售商和各行业的智能化需求，涂鸦的解决方案赋能并提升合作伙伴和客户的产品价值，同时通过技术应用使消费者的生活更加便利，涂鸦智能的智慧商业 SaaS 为丰富的垂直行业提供智能解决方案。涂鸦智能领先业界的技术，符合严格的数据保护标准和安全性。并且与多家世界 500 强公司合作，让生活更智能，包括飞利浦、施耐德电气、联想等。

### APP 安装与登录

![涂鸦智能](image/%E6%B6%82%E9%B8%A6%E6%99%BA%E8%83%BD.jpg)

涂鸦平台产品的配置和使用主要依赖的 APP 为 **涂鸦智能**，在常见的手机应用市场搜索下载即可。

官网下载地址：<https://smartapp.tuya.com/tuyasmart/>

*涂鸦作为开放平台，会向厂商提供全套的定制解决方案，其中就包括 APP 定制服务，因而部分品牌提供的独立 APP 可能为涂鸦换壳。只要是标注为兼容涂鸦平台，都可以采用涂鸦官方 APP 进行操控。*

首次使用 APP 需要注册并登录账号，根据页面提示操作即可。

### 有线网关

![有线网关](image/%E6%9C%89%E7%BA%BF%E7%BD%91%E5%85%B3.png)

网关，是 Zigbee 网络的核心设备，Zigbee 设备通过网关添加到网络中，实现与其他 Zigbee 设备的通信，网关通过网线连接至路由器实现与云端和手机 App 的通信，通过 App 可查看和控制 Zigbee 设备。

常见的网关根据其与互联网连接的方式可分为 **有线网关** 和 **无线网关**。有线网关通过插入网线水晶头（RJ-45 接口）的方式连接互联网，而无线网关则多采用 2.4G WiFi 的方式连接互联网。由于 ZigBee 协议同样工作在 2.4G 频段，考虑到信号干扰及稳定性问题，在情况允许的情况下一般建议优先选用有线网关。

![网关原理](image/%E7%BD%91%E5%85%B3%E5%8E%9F%E7%90%86.png)

#### 网关配网

1. 将网关与电源连接，并通过网线与家庭 2.4GHz 频段路由器相连；
2. 确认配网指示灯（绿灯）常亮（如果指示灯处于其他状态，长按“复位键”，至绿灯常亮)；
3. 确保手机连接家庭 2.4GHz 频段路由器，此时手机、网关处于同一个局域网；
4. 打开涂鸦智能 App “我的家”页，点击页面右上角添加按钮“+”；
5. 依照提示操作设备入网；
6. 添加成功后，即可在“我的家”列表中找到设备。

### 八键开关

![八键开关](image/%E5%85%AB%E9%94%AE%E5%BC%80%E5%85%B3.jpg)

八键开关需要接入常零火，可接入 4 路控制线，通过继电器控制其与火线的通断。

面板按键有背光显示功能，可通过红外感应（传感器在上下两块面板的槽缝中）周围有人后自动开启。默认背光色为橘黄色，在开关状态变为开启后，对应位置的背光将改为白色。

开关正面共设有八个按键，其编号如下：
|  1  |     |     |     |     |     |  2  |
| --- | --- | --- | --- | --- | --- | --- |
|  3  |     |     |     |     |     |  4  |
|  5  |     |     |     |     |     |  6  |
|  7  |     |     |     |     |     |  8  |

上半部分，1-4 号键为普通按键，直接操纵 4 路继电器的状态。

下半部分的 5-8 号键为场景按键，按下后将触发在 APP 中设置的对应场景。

#### 开关配网

1. 确保已经有一个入网完成的网关；
2. 将八键开关接入 220V 交流零火线；
3. 长按八键开关的 1 号键至背光灯闪烁，进入配对模式；
4. 打开涂鸦智能 App “我的家”页，点击页面右上角添加按钮“+”；
5. 依照提示操作设备入网；
6. 添加成功后，即可在“我的家”列表中找到设备。

#### 多控关联

![多控关联](image/%E5%A4%9A%E6%8E%A7%E5%85%B3%E8%81%94.jpg)

1. APP 内点击对应的开关图标，进入开关设置界面；
2. 点击右上角设置（铅笔图标）-> “多控关联”；
3. 在页面上部灰色区中左右滑动选择要同步的开关按键；
4. 点击虚线框中的“+ 关联开关”；
5. 选择需要同步控制的开关（继电器）；
6. 可以重复第 3-5 步，设置多个关联；
7. 若需要删除关联，左滑目标后点击“删除”即可。

#### 普通开关触发场景

![触发场景](image/%E8%A7%A6%E5%8F%91%E5%9C%BA%E6%99%AF.jpg)

1. APP 内 场景 -> 自动化；
2. 点击页面右上角添加按钮“+” -> 设备状态变化时；
3. 选中对应的开关按键 -> 开启；
4. 点击页面下半执行部分的添加按钮“+”；
5. 添加需要执行的动作或场景；
6. 点击页面底部的保存。

### 八键语音开关

八键语音开关整体外观与八键开关一致，功能也基本相似。但是面板中部缝槽中置入的是语音传感器而非红外传感器，故而语音开关的背光灯不支持感应亮起。

默认唤醒词为 **“小智管家”**。

#### 更改唤醒词

1. 确保当前环境相对安静；
2. 说出以下语音指令：“小智管家，学习唤醒词”；
3. 根据语音提示，重复说出想要设置的四字唤醒词。

#### 普通按键语音名称

1. 按住想要设置名称的开关按键，不要松手；
2. 说出以下语音指令：“小智管家，打开 XX”。

XX 为支持的按键名称：

- 主灯
- 筒灯
- 射灯
- 壁灯
- 吊灯
- 灯带

#### 场景按键语音名称

1. 按住想要设置名称的开关按键，不要松手；
2. 说出以下语音指令：“小智管家，XXXX”。

XXXX 支持的按键名称：

- 打开窗帘
- 关闭窗帘
- 明亮模式
- 温馨模式
- 睡眠模式
- 用餐模式
- 回家模式
- 离家模式

#### 继电器锁定设置

1. 按住想要锁定的继电器对应的开关按键，不要松手；
2. 说出以下语音指令：“小智管家，继电器锁定”。

#### 继电器解锁设置

1. 按住想要解锁的继电器对应的开关按键，不要松手；
2. 说出以下语音指令：“小智管家，继电器解锁”。

### 雷达传感器

![雷达传感](image/%E9%9B%B7%E8%BE%BE%E4%BC%A0%E6%84%9F.png)

雷达传感器本质上是一个毫米波雷达，发射电磁波探知周围空间内的物体运动，得益于其较高的精度，可以通过人体呼吸时的微小动作确定空间内人体的存在与具体行为模式。

#### 设置项说明

![雷达说明](image/%E9%9B%B7%E8%BE%BE%E8%AF%B4%E6%98%8E.png)

### 背景音乐主机-P6

![背景音乐-P6](image/%E8%83%8C%E6%99%AF%E9%9F%B3%E4%B9%90-P6.jpg)

#### P6-主机绑定

1. 确保主机已经正常连接到互联网；
2. 从首页下滑，点击齿轮按钮 -> 智能家居；
3. 打开 智能家居模式 开关；
4. 智能主机管理 -> 输入管理密码（默认为 admin）；
5. 根据提示打开 APP 扫码完成绑定。

#### P6-同步设备和场景

1. 确保主机已经正常连接到互联网；
2. 从首页下滑，点击齿轮按钮 -> 智能家居；
3. 打开 智能家居模式 开关；
4. 智能主机管理 -> 输入管理密码（默认为 admin）；
5. 点击右上 重新导入。

#### P6-歌单设置

1. 首页 -> 音乐 -> 我的；
2. 点击新建歌单；
3. 设置歌单名称；
4. 搜索想要的歌曲名称；
5. 点击歌曲右侧的心形按钮，将歌曲收藏至歌单。

#### P6-智能场景设置

1. 从首页下滑，点击设置（齿轮）按钮；
2. 智能家居 -> 智能场景编辑；
3. 选择要编辑的智能场景；
4. 在弹出的页面中可以设置场景的：
   - 图片
   - 名称
   - 别名
   - 音乐（必须通过歌单设置）
   - 音量
   - 定时

### 背景音乐主机-M10

![背景音乐-M10](image/%E8%83%8C%E6%99%AF%E9%9F%B3%E4%B9%90-M10.jpg)

#### M10-网关绑定

1. 确保主机已经正常连接到互联网；
2. 从首页下滑，点击齿轮按钮 -> 网关设置 -> 网关绑定；
3. 根据提示打开 APP 扫码完成绑定。

#### M10-同步设备和场景

1. 确保主机已经正常连接到互联网；
2. 从首页下滑，点击齿轮按钮 -> 网关设置 -> 同步设备。

## 参考源

- [智慧家庭 - 维基百科](https://zh.m.wikipedia.org/wiki/%E6%99%BA%E6%85%A7%E5%AE%B6%E5%BA%AD)
- [2021 年中国智能家居设备行业竞争格局及市场份额分析](https://bg.qianzhan.com/trends/detail/506/220408-3a1aa8e7.html)
- [2022 年全球智能家居行业发展现状及市场规模分析](https://www.qianzhan.com/analyst/detail/220/220920-4719f16d.html)
- [2023 年中国智能家居行业全景图谱](https://www.qianzhan.com/analyst/detail/220/221103-d1d553ba.html)
- [智能家居行业发展前景及趋势预测分析](https://bg.qianzhan.com/report/detail/459/221103-7dce7a7a.html)
- [智能家居通讯协议有哪些？](https://zhuanlan.zhihu.com/p/32941912)
- [智能家居通信协议之争，谁会是最终赢家？](https://www.sdnlab.com/24244.html)
- [5 种常见的智能家居协议优缺点分析](https://new.qq.com/rain/a/20201230A0522300)
- [涂鸦智能](https://www.tuya.com/cn/)