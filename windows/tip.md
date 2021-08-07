# Windows 使用小技巧

## 快捷键

{% hint style="info" %}
Windows 的前身 **MS-DOS** 原生是不支持鼠标的哦！所以硬要追根溯源的话，**键盘** 才是 Windows **最正统** 的操作方式。
{% endhint %}

![](../.gitbook/assets/image%20%2814%29.png)

### 系统快捷键

{% hint style="info" %}
**Windows 徽标键** 可以与下面图示中的键组合
{% endhint %}

![Powertoys &#x5FEB;&#x6377;&#x952E;&#x6307;&#x5357;](../.gitbook/assets/image%20%2813%29.png)

{% hint style="info" %}
**`ALT + TAB` 切换窗口**
{% endhint %}

{% hint style="info" %}
**`CTRL + SHIFT + ESC 任务管理器`**
{% endhint %}

{% hint style="danger" %}
**`CTRL + SHIFT + DELETE` 应急设置**  
也被称作 “三指敬礼” （Three-finger Salute）【向比尔·盖茨】和 “安全键” （Security Keys\)
{% endhint %}



### 通用快捷键

| 基础键 | 指令键 | 命令 |
| ---: | :---: | :--- |
| **`CTRL`** | **`S`** | **保存** |
| **`CTRL`** | **`C`** | **复制** |
| **`CTRL`** | **`V`** | **粘贴** |
| **`CTRL`** | **`F`** | **查找** |
| **`CTRL`** | **`P`** | **打印** |
| **`CTRL`** | **`TAB`** | **切换** |
| **`ALT`** | **`F4`** | **关闭** |
| **`CTRL + SHIFT`** | **`S`** | **另存为** |

## 控制面板

**`控制面板`** 集成了 Windows 系统相关的绝大多数功能与配置选项。  
尽管自 Windows 8 以来微软就致力于用新的 **`设置`** 页面来替代传统的控制面板，但这一项目推进缓慢，同时新的页面布局并未得到广泛认可（个人就觉得控制面板更直观、页面利用率更高）。哪怕是到了 Windows 11 ，依旧有很多设置项缺失，所以遇到需要调整复杂设置的情况还是得靠 **`控制面板`** 。

{% hint style="info" %}
**`WIN + R`** 打开 **运行** 窗口，输入对应的控制面板文件名，回车，即可快速打开窗口。
{% endhint %}

| 文件名 | 对应窗口 |
| ---: | :--- |
| control | 控制面板 |
| appwiz.cpl | 程序和功能 |
| desk.cpl | 显示设置 |
| inetcpl.cpl | Internet 属性 |
| intl.cpl | 区域设置 |
| joy.cpl | 游戏控制器 |
| main.cpl | 鼠标属性 |
| mmsys.cpl | 声音 |
| ncpa.cpl | 网络连接 |
| powercfg.cpl | 电源选项 |
| sysdm.cpl | 系统属性 |
| tabletpc.cpl | 笔和触控 |
| timedate.cpl | 日期和时间 |

{% hint style="warning" %}
尽管在微软的 [官方文档](https://support.microsoft.com/en-us/topic/description-of-control-panel-cpl-files-4dc809cd-5063-6c6d-3bee-d3f18b2e0176) 中有提及对应的控制面板文件名，但此文档已经长期未经维护，不再适用于 Windows 7 之后的版本。  
以上的内容汇总了官方文档和注册表中的内容，筛选了其中的有效项。

相关的注册表键：  
`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Control Panel\Cpls`
{% endhint %}

