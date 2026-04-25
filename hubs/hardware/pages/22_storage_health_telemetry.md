# 22 · 存储健康、SMART、NVMe Log 与带外管理

## 定位

真正成熟的存储系统，不是“插上盘能跑”就结束，而是能持续回答四件事：`现在健康吗`、`哪里在退化`、`控制器和 OS 看到的是不是同一件事`、`故障发生前能不能提前发现`。这实际上是存储的控制面与遥测面，不只是运维附属项。

## 学习目标

- 能区分设备本体、控制器/enclosure、BMC/Redfish 三张健康视图。
- 能用 SMART、NVMe log page、SES、RAID 工具和 Redfish 交叉定位同一块盘。
- 能建立包含序列号、槽位、固件、温度、寿命、错误计数和事件的健康基线。
- 能理解 NVMe-MI、Swordfish、OCP telemetry 这些管理面能力为什么重要。

## 核心直觉

存储健康不是单个“绿色/红色”状态，而是一组跨层证据。OS 视图最接近 I/O 路径，控制器视图最接近阵列和槽位，BMC/Redfish 视图最适合资产、事件和带外自动化；三者对不上时，差异本身就是故障线索。

## 先抓住六个判断问题

1. 你现在拿到的健康信息来自 `设备本体`、`RAID 控制器`、`enclosure`，还是 `BMC / Redfish`？
2. 当前设备是 `ATA/SATA`、`SAS`，还是 `NVMe`？不同介质的健康模型并不一致。
3. OS 看到的是物理盘、NVMe namespace，还是控制器暴露出的 virtual disk？
4. 健康指标是 `原始计数`、`规范化阈值`，还是控制器 / BMC 再加工后的状态灯？
5. 这条遥测路径能否跨 `本地 CLI -> 带外 API -> 批量监控` 三层复用？
6. 你记录的是“当前值”，还是已经建立了可比较的 baseline？

## 机制拆解

| 视图 | 工具/接口 | 重点字段 | 适合回答的问题 |
| --- | --- | --- | --- |
| OS 视图 | `smartctl`、`nvme-cli`、`lsblk`、`journalctl` | 设备名、序列号、温度、错误、namespace、内核 reset | I/O 路径是否异常 |
| 控制器视图 | `storcli`、`perccli`、OMSA、厂商工具 | Physical/Virtual Disk、cache、BBU、rebuild、patrol read | 阵列和物理槽位是否健康 |
| Enclosure 视图 | SES、expander、slot/LED | presence、fault、locator、温度、电源 | 背板和盘位是否异常 |
| BMC 视图 | iDRAC/Redfish/Swordfish | inventory、health、event、thermal、firmware | 批量资产和带外告警是否一致 |

## 先把存储遥测的对象层级看清

### 设备本体

- `ATA / SATA / SAS` 设备常见入口是 `SMART`。
- `NVMe` 设备常见入口是 `log pages`、`identify` 数据结构和命名空间信息。
- 这些数据最接近设备本体，但不一定完整反映 enclosure、控制器缓存或阵列状态。

### 控制器与 enclosure

- RAID 控制器会看到 `Physical Disk / Virtual Disk / Controller Cache / Patrol Read / Rebuild`。
- Enclosure 会看到 `slot`、`电源`、`温度`、`背板状态`、`盘位 presence / fault`。
- 对 SAS 背板和 JBOD 来说，`SES` 这一层非常关键，因为很多“亮灯”“槽位”“风扇”“扩展器”状态根本不在 SMART 里。

### 带外管理与 API

- BMC / iDRAC / Redfish 面向的是整机视图，适合做 inventory、health、event log 和自动化轮询。
- 如果目标是跨服务器、跨机型统一采集，带外 API 往往比本地手工 SSH 更稳定。

## SMART 不是“一个统一指标”

### SMART 的价值

- smartmontools 官方文档说明 `smartctl` 和 `smartd` 用于控制和监控支持 SMART 的 ATA、SCSI 和 NVMe 设备。
- 这意味着 SMART 不是“只看机械盘”的老工具，而是今天排查本地盘健康时仍然很重要的统一入口。

### 但 SMART 的局限也要看穿

- ATA 属性高度依赖厂商实现，字段语义并不完全统一。
- 在 RAID 控制器之后，`smartctl` 能否直接看到物理盘，要看控制器转发能力与访问方式。
- SMART 更像“设备健康原始窗口”，不是完整的数据平面或控制面观测系统。

## NVMe 的健康模型和传统盘不一样

### NVMe 重点不在传统 SMART 属性编号

- NVMe 设备通常更依赖 `smart-log`、`error-log`、`id-ctrl`、`id-ns` 等结构化数据。
- `percentage used`、`data units read/written`、`media errors`、`critical warning` 这类字段，比“第几个 SMART 属性”更有解释力。

### nvme-cli 是 Linux 里的关键工具

- `linux-nvme/nvme-cli` 是 Linux 上 NVMe 管理命令行工具的事实标准入口。
- 对服务器排障来说，它的意义不仅是“列出设备”，更是把 namespace、controller、firmware、log page 和 sanitize / format 类操作放到同一条工具链里。

## NVMe-MI：为什么健康管理不能只靠 OS 内的 I/O 路径

### NVMe-MI 在解决什么问题

- NVM Express 官方说明，`NVMe-MI` 用于在多种操作环境中发现、监控、配置和更新 NVMe 设备。
- 它覆盖 `in-band` 与 `out-of-band` 管理，这一点很关键。

### 为什么它重要

- 仅靠主机 OS 看到的 block device，你很难优雅地管理 enclosure 里的多盘 NVMe 子系统。
- 在服务器和 JBOF 场景里，设备管理往往需要和 BMC / MCTP / SMBus / PCIe VDM 之类的路径结合。
- 所以 NVMe-MI 是“NVMe 数据面之外的管理面”。

## SES、Redfish、Swordfish：带外和跨平台管理的三层视角

### SES

- T10 的 `SES-4` 继续定义 device enclosure command set。
- 这类标准的价值，是让 enclosure 的槽位、状态灯、扩展器与环境信息有统一管理模型。

### Redfish

- DMTF 的 Redfish 已经把平台管理推进到标准 RESTful API 形态。
- 对单台服务器来说，Redfish 很适合做 `inventory / health / event / update / power / thermal` 自动化。

### Swordfish

- SNIA Swordfish 是 Redfish 在存储管理侧的扩展。
- SNIA 在 `2025-06-27` 公布 `Swordfish v1.2.8` 作为 SNIA Standard 发布，并进入 ISO 发布流程；它特别强调对 NVMe 设备管理的扩展。
- 这意味着如果你的目标不是只管理“服务器”，而是逐步统一“服务器 + 存储设备 + 存储服务”，Swordfish 比单纯 Redfish 更贴近存储语义。

## 在 Dell 服务器上，为什么要同时保留三张视图

### 本地 OS 视图

- 用 `lsblk`、`smartctl`、`nvme-cli` 看设备节点和健康计数。

### 控制器视图

- 用 PERC / OMSA / iDRAC 看 `Physical Disk / Virtual Disk / Hot Spare / Rebuild / Patrol Read`。

### BMC / Redfish 视图

- 用 Redfish 拉 inventory、状态和事件。
- 这层特别适合做资产管理、周期巡检和异常趋势发现。

### 真正常见的问题

- OS 看到一个 `Virtual Disk`，控制器看到六块 SAS 盘。
- `smartctl` 拿到的只是抽象状态，PERC 才知道哪块盘在 predictive failure。
- iDRAC 又会给出 slot 级和事件日志级信息。

## 建一个真正可用的存储健康基线

### 第一步：盘点对象

- 列出 controller、virtual disk、physical disk、namespace、enclosure。

### 第二步：记录基线

- 记录 firmware version、介质错误、温度、已写入量、percentage used、reallocated / pending 之类关键字段。

### 第三步：确定采样入口

- 本地：`smartctl`、`nvme-cli`
- 带外：`Redfish`
- OEM：`iDRAC / OMSA / PERC`

### 第四步：做差分而不是只看瞬时状态

- 真正有价值的通常不是某一个绝对值，而是 `增长速度`、`是否突然跳变`、`不同视图是否互相矛盾`。

## 服务器落地时最该问的十个问题

1. 当前健康状态来自哪一层，设备、控制器还是带外？
2. 有没有同时保存 OS、控制器和 BMC 三张清单？
3. 能否直读物理盘健康，还是只能看到逻辑卷 / virtual disk？
4. NVMe 盘有没有通过 `nvme-cli` 定期抓 `smart-log`？
5. SAS / SATA 盘是否能通过 `smartctl` 或 OEM 工具可靠透传？
6. 有没有 enclosure / slot / fault LED 的统一视图？
7. Redfish 是否能稳定输出 inventory、health 和 event logs？
8. 是否为关键盘型建立了 baseline，而不是只在出事时临时抓数？
9. 健康指标是否能关联到采购批次、盘位和业务负载？
10. 当前监控系统能否区分“设备错误”“控制器错误”“背板错误”“热问题”？

## 设计 / 采购判断

- 采购前要求样盘证明 `serial number -> slot -> OS device -> controller physical disk -> Redfish drive` 能闭环。
- 对 NVMe SSD，至少要求能采集 `smart-log`、`error-log`、firmware、温度、`percentage used` 和 namespace 信息。
- 对 SAS/SATA 阵列，确认 RAID 控制器是否支持 SMART 透传、predictive failure、patrol read、rebuild 事件和 fault LED。
- 对规模化环境，优先考虑 Redfish/Swordfish/OCP telemetry 可自动化的产品线，减少只能人工进 GUI 的设备。

## 常见误区

### 误区 1：SMART 正常就等于整条存储路径正常

- 错。控制器缓存、背板、slot、电源和 enclosure 问题都可能不体现在单块盘 SMART 里。

### 误区 2：有 Redfish 就不需要本地工具

- 错。Redfish 更适合带外管理与自动化，本地工具仍然更适合深挖具体设备字段。

### 误区 3：NVMe 健康就是看温度

- 错。`percentage used`、介质错误、错误日志、firmware 和 namespace 信息同样重要。

### 误区 4：控制器里绿色就说明 OS 一定无感知问题

- 错。文件系统、队列层和应用抖动仍可能先在 OS 层暴露出来。

## 故障模式

- 视图矛盾：Redfish 标记健康，但 OS 日志持续出现 NVMe reset 或 I/O timeout。
- 槽位错配：监控只记录 `/dev/sdX`，换盘时无法映射到物理 slot。
- 计数误读：把厂商归一化 SMART 值当成统一含义，忽略 raw value 和设备类型差异。
- 控制器遮蔽：PERC 后面的物理盘错误不能被普通 `smartctl` 命令直接看见。
- 没有基线：出事时才抓日志，无法判断温度、介质错误或寿命消耗是否异常增长。

## Linux / 硬件观察命令

### 观察 1：采一份本机 SMART / NVMe 基线

```bash
sudo smartctl -x /dev/sdX
sudo nvme list
sudo nvme smart-log /dev/nvme0
sudo nvme id-ctrl /dev/nvme0
sudo nvme error-log /dev/nvme0
lsblk -o NAME,SIZE,MODEL,SERIAL,WWN,HCTL,TYPE,MOUNTPOINT
```

目标：建立设备本体级健康快照。

### 观察 2：做一张三视图对照表

- 列 1：OS 看到的设备
- 列 2：控制器看到的 Physical / Virtual Disk
- 列 3：Redfish / iDRAC 看到的 slot 与状态

目标：把 “同一个对象在不同层怎么出现” 说清楚。

### 观察 3：抓一次 Redfish 存储清单

- 获取 storage、drives、event log、thermal / power 相关资源。

目标：验证带外 API 是否可用于批量巡检。

### 观察 4：抓内核侧存储错误时间线

```bash
journalctl -k --since "24 hours ago" | grep -Ei 'smart|nvme|scsi|sas|ata|raid|blk|reset|timeout|media|error'
```

目标：把设备健康数据和内核 I/O 异常放在同一条时间线上比较。

## 前沿趋势

- NVMe-MI 让 NVMe 设备发现、监控、配置和更新可以走 in-band 与 out-of-band 管理路径，适合大规模 NVMe/JBOF 环境。
- SNIA Swordfish 在 Redfish 之上扩展存储语义，适合把服务器、阵列、NVMe 设备和存储服务统一到 API 管理模型。
- OCP Datacenter NVMe SSD 方向把遥测、LED、故障注入、日志、安全和管理一致性变成数据中心级采购指标。
- 遥测会从“事后排障”转向“寿命消耗、温度、错误增长率、固件批次和业务负载”的趋势分析。

## 本页要配套记住的概念卡

- SMART / health telemetry
- NVMe log pages
- NVMe-MI
- SES / enclosure services
- Swordfish

## 延伸阅读

- smartmontools Documentation: https://www.smartmontools.org/static/doxygen/
- nvme-cli: https://github.com/linux-nvme/nvme-cli
- NVM Express Management Interface Specification: https://nvmexpress.org/specification/nvme-mi-specification/
- Redfish Telemetry White Paper: https://www.dmtf.org/documents/redfish-spmf/redfish-telemetry-white-paper-0
- SNIA Swordfish: https://www.snia.org/swordfish-property-guide/release/latest
- SNIA Swordfish v1.2.8 Release Note: https://www.snia.org/blog/2025/swordfishr-v128-released-standard-entering-iso-publication-process
- T10 SES Working Drafts: https://www.t10.org/drafts.htm
