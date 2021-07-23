---
description: 平时遇到的各种小问题及解决方式
---

# 坑

## Windows

### RDP

{% hint style="info" %}
查看 RDP 端口

```text
Get-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "PortNumber"
```
{% endhint %}

{% hint style="info" %}
更换 RDP 端口为 `XXXX`

```text
Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "PortNumber" -Value XXXX
New-NetFirewallRule -DisplayName 'RDPPORTLatest' -Profile 'Public' -Direction Inbound -Action Allow -Protocol TCP -LocalPort XXXX
```
{% endhint %}

{% hint style="danger" %}
RDP 服务器需要至少 **使用账号密码**_**（非 PIN 码）**_**完成一次系统登陆** 后才可用。
{% endhint %}

{% hint style="danger" %}
RDP 服务器需要 **关闭仅使用 Windows Hello 登录** 。
{% endhint %}

### WSL

{% hint style="warning" %}
> 参考的对象类型不支持尝试的操作。
>
> \[已退出进程，代码为 4294967295\]

```text
netsh winsock reset
```
{% endhint %}



## Linux

### Config

{% hint style="info" %}
配置文件最好以换行符结尾，方便后续 `echo "key=value">>config.conf` 追加配置操作。
{% endhint %}

## GitHub

### LICENSE

![](../.gitbook/assets/image.png)

## Hardware

### RJ45 水晶头

![](../.gitbook/assets/image-3-.png)

### 螺丝标准

![](../.gitbook/assets/image%20%281%29.png)

