---
description: KubeCon + CloudNativeCon + Open Source Summit China 2023 技术分享
---

# 如何在大型集群中加速 Pod 的启动？

本文整理自 Paco Xu（DaoCloud）与 Byron Wang（Birentech）在 KubeCon + CloudNativeCon + Open Source Summit China 2023 的技术分享。

## 一、背景与 Pod 启动耗时的定义

常言道：“良好的开端是成功的一半”（A work well begun is half-ended）。
对 Pod 而言同样如此：Pod 在生命周期中所面临的绝大多数初始化、网络、挂载及配置问题，都集中在启动这一短暂的窗口期。
一旦 Pod 跨过启动阶段并稳定运行，后续出现组件配合问题的概率将大幅降低。
因此，深入研究并优化 Pod 的启动过程，对于维护大规模集群的稳定性具有极高的价值。

在讨论如何加速启动之前，首先需要明确一个基准：**Pod 启动多快才算快？一个正常 Pod 的启动耗时应该在什么范围内？**

根据常见的应用场景，Pod 启动耗时大致可划分为以下几个梯队：

* **轻量级 Pod**（`< 50ms`）：通常为极简的无状态微服务，无外部卷依赖，且运行节点上已缓存镜像。
* **高度优化的 Pod**（`~ 100ms`）：镜像经过精简，容器运行时（Runtime）及网络插件（CNI）经过高度调优。
* **常规 Pod**（`数秒级`）：绝大多数业务应用的正常启动范围。
* **重型 Pod**（`15s ~ 1min`）：镜像体积较大（如 AI 训练镜像、重型 Java 应用），或涉及复杂的网络配置与远程存储卷挂载。
* **极端延迟 Pod**（`> 5min`）：通常由大镜像拉取超时、网络拥堵、存储挂载失败或初始化容器（Init Container）阻塞引起。

优化的核心目标，就是将重型 Pod 的启动时间尽量压缩至秒级，同时确保常规 Pod 稳定保持在毫秒级到亚秒级的水平。

## 二、Pod 创建阶段的加速（API Server）

Pod 的创建通常由 Controller（如 Deployment Controller）或自定义 Operator 发起。
API Server 及底层存储 etcd 的响应速度，直接决定了 Pod 创建的第一步效率。

### 1. API Server 响应变慢的根本原因

* **etcd 性能瓶颈**：etcd 读写延迟高、磁盘 I/O 压力大或 etcd 负载过高。
* **API Server 高负载**：API Server 本身 CPU/内存资源不足，导致请求排队。
* **并发限流（Rate Limit）**：请求频率触发了 API Server 的流控阈值。
* **Webhook 延迟过高**：集群中注册的 Mutating Webhook 或 Validating Webhook 响应缓慢，这是最常见的影响创建速度的“元凶”。

### 2. Pod 创建失败或处于 Pending 状态的常见诱因

* **校验失败（Validation Failure）**：配置格式或权限不合规。
* **安全策略拦截**：如 Pod Security Admissions（PSA）检查未通过。
* **集群资源耗尽**：Pod 处于排队等待状态（如大规模批处理作业中）。
此时需合理配置 **Priority & Preemption**（优先级与抢占机制），确保关键业务 Pod 能够优先创建。
* **资源配额超限**：Namespace 级别的 ResourceQuota 已达上限。

**优化策略**：对 API Server 和 etcd 进行性能调优（如升级 SSD、调大并发处理线程），严格控制并优化自定义 Webhook 的代码逻辑与响应时间，并在大规模并发创建时合理使用优先级抢占机制。

## 三、Pod 调度阶段的加速（Scheduler）

若 Pod Spec 中未明确指定 `nodeName`，则必须通过 `kube-scheduler` 寻找合适的节点。
在大规模集群中，调度延迟会随着节点数量和调度规则复杂度上升。

### 1. 调度阶段的瓶颈分析

* **调度失败与重试**：调度器内部拥有重试队列（Retry Queue）。
当调度变慢时，不一定是调度器性能问题，往往是由于调度条件不满足导致 Pod 被频繁退回到重试队列，从而消耗了调度器的计算资源。
* **集群规模过大**：节点基数庞大导致调度器在进行评分（Scoring）和过滤（Filtering）时计算量激增。
可通过调低 `percentageOfNodesToScore` 参数，限制参与评分的节点比例。
* **拓扑与亲和性规则过于复杂**：大量使用深度的 Node Affinity、Pod Affinity 或 Pod Anti-Affinity 规则，会显著增加调度器的算法复杂度。

### 2. 调度侧加速方案

* **负载感知调度（Load-Aware Scheduling）**：原生的调度算法主要基于资源的 Request 进行静态计算。
通过引入社区的 Trimaran 插件，调度器可以感知集群节点的真实运行负载（Real Load），优先将 Pod 调度到实际负载较低的节点上，避免 Pod 在启动瞬间因物理节点 CPU 争抢而卡顿。
* **打散策略（Spread Strategy）**：尽量避免将大批量同时启动的 Pod 调度到同一个物理节点上。
通过在不同可用区（Zone）或 Node 间进行打散调度，可以有效分摊镜像拉取、网络 I/O 及 CPU 消耗压力，防止单节点瞬时过载拖慢整体启动进度。

## 四、GPU / AI 场景下的资源调度与分配机制

在 AI 和大模型训练场景下，Pod 往往需要独占或共享 GPU 资源。
异构资源的调度和分配机制与常规 CPU/内存有着本质的区别。

```text
                    +--------------------+
                    |    API Server      |
                    +---------+----------+
                              |
                     (Request GPU Resource)
                              |
                              v
                    +---------+----------+
                    |   Kube-Scheduler   |
                    +---------+----------+
                              |
               (Bind Pod to Node via GPUBinding)
                              |
                              v
+-----------------------------+-----------------------------+
| Kubernetes Node                                           |
|                                                           |
|    +------------------+             +------------------+  |
|    |     Kubelet      |             |    Node Agent    |  |
|    +--------+---------+             +--------+---------+  |
|             |                                |            |
|     (Allocate Request)               (Read GPUBinding)    |
|             |                                |            |
|             v                                v            |
|    +--------+---------+             +--------+---------+  |
|    |  Device Plugin   +<------------+   Symlink File   |  |
|    |  (Fake Device)   |             | (Actual Physical |  |
|    +------------------+             |     GPU Card)    |  |
|                                     +------------------+  |
+-----------------------------------------------------------+
```

### 1. 传统的 Device Plugin 机制

Kubernetes 原生通过 Device Plugin 扩展异构资源的支持。
设备插件向 Kubelet 注册 Unix Socket，上报本地可用设备（如 GPU、RDMA），Pod 通过 `resources.limits` 进行显式申请。

**原生 Device Plugin 机制的局限性**：

1. **无法感知动态属性变化**：例如 FPGA 的动态重编程或设备硬件发生热插拔。
2. **型号区分模糊**：同一厂商（如 NVIDIA）的不同硬件型号（如 RTX 3090、A100）在原生机制下会被统一识别为 `nvidia.com/gpu`，调度器无法做到型号级精细化匹配。
3. **异构混部复杂**：同一集群内如果同时存在 NVIDIA、AMD 及国产算力芯片，需要部署和维护多套不同的 Device Plugin。
4. **不支持多物理卡精细化分配与拓扑感知**：Kubelet 在收到调度结果后，会在本地“随机”挑选卡，调度器无法在全局层面上干预“具体分配 0 号卡还是 7 号卡”，从而无法优化 NVLink 或 PCIe 的拓扑通信结构。

### 2. GPU 场景下的核心调度算法

为支撑 AI 并行计算，调度器层面需要实现以下高级算法：

* **Gang Scheduling（组调度 / All-or-Nothing）**：大模型分布式训练需要多个 Worker 协同。
调度器必须保证同一个 Job 下的所有 Pod 要么全部调度成功，要么一个都不调度，避免部分 Pod 抢占了资源却因人数不足而无限挂起，造成整批 GPU 资源死锁。
* **SKU / 等比例分配**：物理节点的 CPU、内存、GPU 与 RDMA 通常存在最佳绑定配比。
调度时需进行多维度等比例计算，防止 CPU 被非 AI 任务单方面占满而导致 GPU 空置。
* **Binpack（装箱算法）与 DRF（主导资源公平算法）**：Binpack 负责尽量堆满单个节点以留出完整的空闲节点；DRF 则在多租户场景下，保证各队列能够公平、不饿死地获取其主导资源。

## 五、节点侧的 Pod 启动加速（Node / Kubelet）

当 Pod 绑定到节点后，Kubelet 接管其生命周期。
此阶段是 Pod 启动耗时的重灾区，主要包含四大步骤：**拉取镜像 -> 创建 Sandbox（CNI）-> 挂载卷（Volume）-> 启动主进程**。

### 1. 定位启动瓶颈的“控制变量法”

当节点上的 Pod 启动缓慢时，可通过以下手段进行快速排查定位：

1. **提前手动拉取镜像**：若启动速度大幅提升，说明瓶颈在于**网络带宽或镜像拉取**。
2. **配置 `hostNetwork: true` 启动**：若启动速度大幅提升，说明瓶颈在于 **CNI 网络插件分配 IP 或 Sandbox 创建**。
3. **移除所有 Volume 挂载**：若启动速度大幅提升，说明瓶颈在于**存储挂载或配置卷预热**。
4. **去除 CPU/Memory Limit 限制**：若启动速度大幅提升，说明容器在初始化时遇到了严重的 **CPU Throttling（CPU 限制流控）**。

### 2. 镜像相关优化手段

* **精简镜像体积**：
  * 在构建时合理设置 `.dockerignore` 排除非必要文件。
  * 选用极简基础镜像（如 Alpine 或 scratch）。
  * 采用多阶段构建（Multi-stage builds），剔除编译依赖。
  * 精简并压缩镜像层数（Squash Layers）。
  * **严禁在生产镜像中打包 Debug 工具**：由于 `kubectl debug` 临时容器（Ephemeral Containers）技术已非常成熟，生产镜像应保持绝对纯净。
推荐使用 wagoodman 的开源工具 `dive` 逐层分析镜像体积与效率。
* **大镜像加速分发技术**：
  * **P2P 分发**：面对动辄数十 GB 的大模型镜像，部署 Dragonfly 或 Kraken 等 P2P 镜像分发系统，可以极大缓解镜像仓库的单点带宽压力。
  * **按需加载（Lazy Pulling）**：结合 Nydus 或 stargz-snapshotter 等按需加载技术，容器无需等待整个镜像下载并解压完成即可瞬时启动，在运行过程中按需从网络拉取数据块。
* **Kubelet 并行拉取优化**：Kubelet 默认采用串行镜像拉取策略（`serializeImagePulls: true`）以保护磁盘 I/O。
在高性能 SSD 节点上，建议将 `serializeImagePulls` 设为 `false`，并合理配置 `maxParallelImagePulls` 提升拉取并发度。
目前 Kubelet 对于单个 Pod 内的多个容器镜像拉取仍为串行，社区后续将持续优化该细节。

### 3. 初始化容器（Init Containers）优化

* 仅在 Init Container 中编写绝对必要的配置动作。
* 如果存在多个 Init Container，评估是否能将其合并或进行并发处理。
* 如果可能，优先使用主容器的 `postStart` 钩子或前置 DaemonSet 预先完成节点环境准备，避免阻塞 Pod 主生命周期的演进。

### 4. CPU Limits 优化（避免启动态 CPU Throttling）

* **背景**：Java 等重型应用在启动阶段会进行大量的类加载和 JIT 编译，瞬时 CPU 需求极高（如需 2 核），但启动后日常运行仅需 0.5 核。
若在 Limit 中限制为 0.5 核，会导致启动过程极其缓慢，甚至触发健康检查超时而无限重启。
* **解决方案**：自 Kubernetes v1.27 起，原地动态调整机制（In-Place Pod Vertical Scaling）正式引入。
可以结合 VPA（Vertical Pod Autoscaler），在 Pod 启动时为其分配充足的 CPU 额度，待其 Ready 并进入稳定运行态后，再无感知地调低其 Limit，在不造成资源浪费的前提下实现启动加速。

### 5. 静态 CPU 策略绑核（Static CPU Policy）

* 在高负载或低延迟场景下，通过 Kubelet 的 Static CPU Manager 实现物理核绑定（Core Pinning），避免容器进程在物理核之间频繁切换带来的上下文开销，提升启动稳定性。
* 启用条件：Pod 的 QoS Class 必须为 `Guaranteed`（即 CPU 的 Request 等于 Limit）。
* **配置变更注意点**：将 CPU Manager 策略从 `none` 变更为 `static` 时，需要先 Drain 节点、停止 Kubelet、手动清理 CPU 状态文件（默认位于 `/var/lib/kubelet/cpu_manager_state`）、修改配置后重新拉起 Kubelet。

### 6. 健康检查探针（Probes）优化

* 合理设置 StartupProbe、ReadinessProbe 和 LivenessProbe 的阈值。
优先通过 StartupProbe 接管长周期的初始化，防止在应用启动成功前被 LivenessProbe 误杀。
* 社区目前正在推进高精度（亚秒级 / 毫秒级）探针的落地应用，未来可实现更灵敏的 Ready 状态感知。

### 7. 容器快照技术（Forensic Container Checkpointing）

自 v1.25 起，Kubernetes 引入了运行时容器快照功能。
对于启动初始化动作（如加载海量配置、JVM 内存预热）极其漫长的应用，可以在其完全 Ready 后通过快照技术（基于 CRIU 机制）导出当前的运行状态。
后续扩容或重启时，直接通过快照进行内存还原，跳过全部启动初始化，实现秒级冷启动。

### 8. SELinux 与 API QPS 调优

* **SELinux Relabeling 优化**：如果挂载的持久卷中包含海量小文件，SELinux 会在启动时递归扫描并重新标记所有文件，导致严重的挂载超时。
自 v1.27 起，通过 Mount Options 优化（Beta），可以直接在挂载参数中指定 SELinux 标签，免去递归扫描。
* **调优 Kubelet QPS 限制**：当单节点上高并发启动大量 Pod 时，Kubelet 会因为默认的 API 请求限制（先前默认 `QPS=5`，`Burst=10`）而向 API Server 发起排队。
在 v1.27 中，Kubelet 的默认并发限制被放大了 10 倍，有效消除了高密度部署场景下的流控瓶颈。

## 六、异构计算（GPU）管理与大模型数据加载加速

随着大模型时代的到来，异构计算的管理和海量数据集的加载速度成为了制约集群启动效率的最新瓶颈。

### 1. 异构资源管理演进：从 Device Plugin 到 DRA

为了彻底解决传统设备插件的缺陷，Kubernetes 社区推出了 **DRA（Dynamic Resource Allocation，动态资源分配，KEP-3063）**。

* **DRA 的优势**：将设备管理上升到一等公民，提供类似于 PV/PVC 的 ResourceClaim 声明模型，支持多设备参数灵活定义、网络附加 GPU（Network-Attached GPUs）、复杂网络拓扑感知以及灵活的设备共享策略。
* **当前的挑战**：由于调度需要高频读写 API Server 获取 Claim 状态，在大规模高并发调度时会略微增加调度器的时延，且其 YAML 配置复杂度显著增加。

### 2. 壁仞科技的自定义实践（虚拟设备 + 自定义调度器）

在社区 DRA 尚未完全成熟和普及前，壁仞科技通过“Fake Device + 拓扑感知调度器”方案实现了极致的异构计算管理：

1. **节点 Agent 伪装**：在节点本地运行 Agent，向 Kubelet 的 Device Plugin 注册一组“虚假的 GPU 设备”（Fake Device）。
这些 Fake Device 在底层其实只是无实质意义的软链接（Symlink）或空心设备文件。
Kubelet 误以为本地拥有充足的异构设备，从而直接放行 Pod 的调度申请。
2. **自定义调度器（Custom Scheduler）拓扑决策**：调度器拥有集群全局的物理 GPU 拓扑视图。
在调度阶段，它会根据卡间 NVLink 带宽、PCIe 拓扑及共享状态做出最优的分配决策，并将具体的绑定信息（如将几号物理卡分配给该 Pod）写入自定义的 `GPUBinding` 资源中。
3. **设备映射动态链接**：当 Kubelet 本地准备拉起容器并调用 `PreStartContainer` 时，节点 Agent 拦截该调用，读取 `GPUBinding`，在底层将真正的物理 GPU 设备文件建立软链接，直接替换掉容器内部的 Fake Device。
4. 这套方案在不修改 Kubernetes 核心代码的前提下，完美实现了 GPU 的拓扑感知分配、跨 Pod 共享、切分以及精细化卡指定。

### 3. AI 训练数据集加载加速

大模型训练在启动时需要加载数 TB 甚至数十 TB 的训练数据集。
若直接通过网络从远程 HDFS 或 Ceph 中读取，会导致启动初期的 I/O 吞吐极低。

**加速数据加载的优化闭环**：

* **一阶段（Prepare）**：在 Pod 调度成功后，前置的 Data Agent 开始异步预热，提前将所需的分块数据集拉取并缓存在节点本地的 NVMe SSD（Disk）中。
* **二阶段（Memory Copy）**：将高频使用的热点训练数据，通过节点大内存直接载入 Page Cache（Memory）。
* **三阶段（GPU Copy）**：GPU 运行时直接从本地高带宽内存或本地 SSD 中读取数据，实现高吞吐训练，避开分布式文件系统的网络带宽限制。
* **四阶段（LRU Cache Eviction）**：本地 Data Agent 实时监控节点的磁盘空间，当训练结束或空间不足时，自动基于 LRU 淘汰算法清理陈旧的数据集缓存，确保本地磁盘健康。

## 七、可观测性（Observability）

在进行任何性能调优前，完备的数据指标是唯一的事实依据。

### 1. 内核级深链追踪

在大规模、极端延迟调优场景下，可利用 Linux BCC / BPF 调试工具（如 `execsnoop` 监听进程创建，`biosnoop` 监听底层磁盘 I/O 延迟）定位是由于系统调用卡顿还是宿主机硬件性能瓶颈导致。

### 2. Kubelet 结构化启动日志

自 v1.26 起，Kubelet 提供了极其详尽的结构化 Pod 启动追踪日志。
当 Pod 启动完成后，Kubelet 会在日志中输出以下关键时间轴：

```json
{
  "observed_pod_startup_duration": "pod_start_sli_duration_seconds",
  "podCreationTimestamp": "2022-12-30 15:33:00",
  "firstObservedPulling": "2022-12-30 15:33:05",
  "lastObservedFinishedPulling": "2022-12-30 15:33:10",
  "observedRunningTime": "2022-12-30 15:33:15"
}
```

通过这些日志，你可以清晰地得出以下结论：

* **镜像拉取耗时** = `lastObservedFinishedPulling` - `firstObservedPulling` = 5 秒
* **Sandbox 及容器配置耗时** = `observedRunningTime` - `lastObservedFinishedPulling` = 5 秒

### 3. Prometheus 指标监控

Kubelet 会将上述统计上报为标准的直方图指标：`pod_start_sli_duration_seconds`。

```prometheus
# Prometheus 直方图部分数据示例：
pod_start_sli_duration_seconds_bucket{le="10"} 120
pod_start_sli_duration_seconds_bucket{le="30"} 145
pod_start_sli_duration_seconds_bucket{le="45"} 146
```

通过观察分位数（如 P99、P95），运维团队可以敏锐地发觉是否有某些特定类型的 Pod（或运行在特定节点上的 Pod）出现了异常的长尾延迟，并基于该指标在 Grafana 上组建监控大盘与告警通路，从而保障集群的整体性能。
