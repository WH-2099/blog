---
description: 天下武功，唯快不破
---

# 树莓派实时性优化

## 优化目的

项目需要用 **树莓派 4B** 实现 **毫秒级控制**，并且误差控制在 **5%** 内（即 **0.05ms = 50us**）。   
本身这种精度的控制应该考虑由 C 实现，但考虑到以下原因，最终还是选用了 **Python** 作为主语言。 

1. 用惯了 Python，好久好久没有写过 C 了，真的 **不想写 C** 啊啊啊啊 \(/▽＼\) 
2. 控制过程中 Python 调用的 **包多是封装了底层 C 库实现，实际性能并不差**
3. 整体程序框架中，**需要 Python 实现的顶层控制非常简单**

那么，**为了实现 50 us 的控制精度，咱就需要针对树莓派上运行的 Raspberry Pi OS 及 Python 进程 进行优化**。

## 优化内容

优化的主要思路参考了下面的两篇文章：

* [树莓派提高实时性的几种方式\(使用python测试\)](https://jianshu.com/p/47d5d89c164c)
* [四足机器人高算力、低成本主控第一步：给树莓派打上RT实时补丁](https://zhuanlan.zhihu.com/p/358151393)

### 提升 Python 进程优先级

#### 操作

**1. 首先需要安装 `wiringpi` 包：**

```bash
sudo pip3 install wiringpi
```

**2. 其次需要在程序中加入以下内容：**

```python
import wiringpi
wiringpi.piHiPri(99)
```

#### 原理

`wiringpi` 中的 `piHiPri` 函数通过调用 Linux 内核的 API，改变了当前进程的调度策略及优先级进一步提高了实时性。 这和 Python 中标准库的 `os.setpriority` 不同，`os.setpriority` 并未改变调度策略，实际优化效果很有限。 需要注意的是，此处设置的优先级是用于比较的相对值，严格地说并不是越大就越好。  

{% hint style="info" %}
> The priority parameter works relative to others – so you can make one program priority 1 and another priority 2 and it will have the same effect as setting one to 10 and the other to 90 \(as long as no other programs are running with elevated priorities\)

priority 参数 **相对** 于其他参数有效-因此您可以将一个程序的优先级设置为 1，将另一个程序的优先级设置为 2，其效果与将一个设置为 10，另一个设置为 90 相同（只要没有其他程序以更高的优先级运行）。
{% endhint %}

### 绑定独立 CPU

#### 操作

**1. 首先需要更改内核启动参数：**

```bash
sudo sed -i.bak -e 's/$/ isolcpus=3/' /boot/cmdline.txt
```

{% hint style="warning" %}
**！！！请注意：单引号中的空格为必须！！！**  
其中 `3` 可以更改为你想要指定的 CPU 编号（树莓派 4B 中 CPU 编号为 0-3\)。   
_考虑到未来可能需要还原，此步将原文件添加 .bak 后缀进行了备份。_
{% endhint %}

**2. 接着安装后面要用到的 `psutil` 包**

```bash
sudo pip3 install psutil
```

**3. 最后在程序中加入以下内容：**

```python
import psutil
current_process = psutil.Process()
current_process.cpu_affinity([3])
```

{% hint style="info" %}
_此处的 `3` 为上文提到的 CPU 编号_
{% endhint %}

#### 原理

首先让系统不会主动调度程序到指定 CPU 上运行，也就是 **保证指定 CPU 默认是完全空闲** 的状态。 其次让 **程序主动要求在指定 CPU 上运行**。 这样程序就完成了 **对指定 CPU 的独享**。

### 打内核 RT 补丁

#### 操作

**1. 这里作者选择直接下载编译好的内核：** 

### \*\*\*\*[**rt-kernel.tgz**](https://github.com/lemariva/RT-Tools-RPi/blob/4fb1bb511d1701d5b0975314ac1ec87b792f9530/preempt-rt/kernel_4_19_59-rt23-v7l+/rt-kernel.tgz)\*\*\*\* <a id="blob-path"></a>

链接源自Github，所以需要科学上网，你懂的 \(￣\_￣ \)

{% hint style="info" %}
如果你想要亲手编译内核，可参考以下资料 ：

1. \_\_[Raspberry Pi: Real Time System - Preempt-RT Patching Tutorial for Kernel 4.14.y](https://lemariva.com/blog/2018/07/raspberry-pi-preempt-rt-patching-tutorial-for-kernel-4-14-y)
2. [四足机器人高算力、低成本主控第一步：给树莓派打上RT实时补丁](https://zhuanlan.zhihu.com/p/358151393)\_\_
3. [Raspberry Pi OS Linux Kernel building](https://www.raspberrypi.org/documentation/linux/kernel/building.md)
4. [preemptrt\_setup](https://wiki.linuxfoundation.org/realtime/documentation/howto/applications/preemptrt_setup)

_由于涉及内容相对庞杂，且作者在此方面经验尚浅，故不做展开。_
{% endhint %}

**2. 将下载的压缩包传至树莓派 /tmp 目录下：**

_基础操作，如不理解请善用搜索引擎_

**3. 依次运行以下命令：**

```bash
sudo -i
cd /tmp
mkdir rt-kernel
tar -xzvf rt-kernel.tgz -C rt-kernel
cp -Rdf rt-kernel/boot/* /boot/
cp -Rdf rt-kernel/lib/* /lib/
cp -Rdf rt-kernel/overlays/* /boot/overlays/
cp -Rdf rt-kernel/bcm* /boot/
echo 'kernel=kernel7_rt.img' >> /boot/config.txt
```

**4. 重启系统：**

```bash
sync
reboot
```

**5. 检查内核版本：**

```bash
uanme -r
```

若输出为 `4.19.59-rt23-v7l+`，说明安装成功。

#### 原理

> Linux ****是典型的分时应用系统，对于实时性要求很高的应用，必须对内核本身动手术。而 RTLinux 则采取了一种比较聪明也比较折中的办法：他们实现一个最底层的精简的调度器，用于调度实时线程，原来的内核本身则成为实时调度器的一个优先级最低的任务。这样，当有实时任务时，普通内核已经建立于其上的普通进程被强制中断，实时线程被强制执行；只有当若有实时线程都让出cpu之后，普通内核才被运行，再由普通内核去调度执行普通的应用程序。

## 性能测试

### 测试方法

1. 用 `rt-tests` 软件包中的 `cyclictest` 工具测试系统实时性：

   ```bash
   # 安装 rt-tests 软件包
   sudo apt install rt-tests
   # 实际运行测试的命令，--duration 指定测试时长，此处为 1 分钟
   cyclictest --mlockall --smp --priority=80 --interval=100 --distance=0 --duration=1m
   ```

2. 用以下程序测试程序实时性：

   ```python
   from time import monotonic, sleep
   from array import array
   from statistics import mean

   i = 0
   data = array("f")
   while i < 60000:
       t0 = monotonic()
       sleep(0.001)
       t1 = monotonic()
       data.append((t1 - t0 - 0.001) * 1000000)
       i += 1
   print("AVG: %.2f us" % mean(data))
   print("MAX: %.2f us" % max(data))
   print("MIN: %.2f us" % min(data))
   ```

### 测试数据

#### 优化前

```bash
policy: fifo: loadavg: 0.55 0.66 0.36 1/293 2309

T: 0 ( 2303) P:80 I:100 C: 599954 Min:      6 Act:   18 Avg:   15 Max:     936
T: 1 ( 2304) P:80 I:100 C: 599498 Min:      5 Act:   18 Avg:   15 Max:     382
T: 2 ( 2305) P:80 I:100 C: 599062 Min:      5 Act:   18 Avg:   14 Max:    1482
T: 3 ( 2306) P:80 I:100 C: 598541 Min:      6 Act:   19 Avg:   14 Max:    5696
```

{% hint style="success" %}
**AVG: 82.94 us  
MAX: 6550.83 us  
MIN: 32.67 us**
{% endhint %}

#### 优化后

```bash
policy: fifo: loadavg: 2.38 1.39 0.64 1/308 1491


T: 0 ( 1471) P:80 I:100 C: 599898 Min:      5 Act:   20 Avg:   13 Max:      95
T: 1 ( 1472) P:80 I:100 C: 599418 Min:      6 Act:   18 Avg:   13 Max:     114
T: 2 ( 1473) P:80 I:100 C: 598916 Min:      5 Act:   18 Avg:   13 Max:     102
T: 3 ( 1474) P:80 I:100 C: 598534 Min:      6 Act:   21 Avg:   14 Max:      34
```

{% hint style="info" %}
**AVG: 82.94 us  
MAX: 6550.83 us  
MIN: 32.67 us**
{% endhint %}

#### 网上已有的树莓派 3B+ 上的具体性能测试数据

{% hint style="warning" %}
请注意，此处并非本次优化的数据。
{% endhint %}

![](../.gitbook/assets/image%20%2811%29.png)

![](../.gitbook/assets/image%20%2810%29.png)

### 测试结论

从以上数据可以看出，系统实时性提升主要体现在 **极值** 的降低上。

**在实际程序中，均值与极值都有非常可观的降低。**  
_优化完成后，手头 Python 运行的测试误差为 30 us 左右，考虑到项目可容许的偏差是 50 us，终于——咱不用去写 C 啦！o\(￣▽￣\)ブ_

## 参考源

1. [树莓派提高实时性的几种方式\(使用python测试\)](https://jianshu.com/p/47d5d89c164c)
2. [四足机器人高算力、低成本主控第一步：给树莓派打上RT实时补丁](https://zhuanlan.zhihu.com/p/358151393)
3. [Wiring Pi -- GPIO Interface library for the Raspberry Pi](http://wiringpi.com/reference/priority-interrupts-and-threads)
4. [preemptrt\_setup](https://wiki.linuxfoundation.org/realtime/documentation/howto/applications/preemptrt_setup)
5. [sched\_setscheduler\(\)函数](https://blog.csdn.net/wennuanddianbo/article/details/79304869)
6. [Raspberry Pi OS Linux Kernel building](https://www.raspberrypi.org/documentation/linux/kernel/building.md)
7. [psutil.Process.cpu\_affinity](https://psutil.readthedocs.io/en/latest/#psutil.Process.cpu_affinity)
8. [Cyclictest Wiki](https://wiki.linuxfoundation.org/realtime/documentation/howto/tools/cyclictest/start)
9. [wiringPi/piHiPri.c](https://github.com/WiringPi/WiringPi/blob/5c6bab7d4279e8c0cc890984eaa1a69ff3af1c99/wiringPi/piHiPri.c)
10. [sched.7](https://man7.org/linux/man-pages/man7/sched.7.html)
11. [Raspberry Pi: Real Time System - Preempt-RT Patching Tutorial for Kernel 4.14.y](https://lemariva.com/blog/2018/07/raspberry-pi-preempt-rt-patching-tutorial-for-kernel-4-14-y)
12. [Rapberry Pi: Preempt-RT Kernel Performance on Rasbperry PI 3 Model B+](https://lemariva.com/blog/2018/04/rapberry-pi-preempt-rt-kernel-performance-on-rasbperry-pi-3-model-b)

