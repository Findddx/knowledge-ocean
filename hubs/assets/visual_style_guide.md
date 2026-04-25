# OS Hub / Hardware Hub Visual Style Guide

本指南用于 OS Hub 与 Hardware Hub 的封面图、章节解释图、架构图和真实硬件插图。目标是形成克制、可信、耐看的技术视觉系统：有艺术感，但不靠廉价赛博感和过度装饰制造“AI 味”。

## 1. 总体视觉原则

### 1.1 气质

- **可信、冷静、结构清楚**：优先让读者理解系统边界、数据路径、硬件形态和关键权衡。
- **抽象但不俗气**：封面可以抽象化，但必须暗示真实主题，例如调度队列、内核边界、缓存层级、PCIe 拓扑、机箱背板、GPU 互连。
- **少即是多**：保留足够留白，避免堆叠无意义电路线、发光粒子、随机晶格、抽象芯片纹理。
- **工程而非营销**：画面应像高质量技术出版物或研究手册，而不是消费电子广告页。

### 1.2 明确避免

- 避免高饱和霓虹色、廉价蓝紫渐变、荧光粉/电光青大面积铺底。
- 避免“泛 AI 风”：发光大脑、随机神经网络线球、无来源芯片幻想图、过度镜面反射。
- 避免在信息图中使用复杂背景、阴影、厚重 3D、装饰性透视。
- 避免真实硬件图片被过度裁切、模糊、暗化到无法识别接口、封装、插槽或结构。

## 2. 图像类型与使用边界

### 2.1 Mermaid：用于精确结构

适用：

- OS 内核路径、进程/线程状态机、I/O 栈、调度器、网络栈。
- Hardware Hub 的 PCIe/CXL 拓扑、NUMA 拓扑、存储路径、RAID/JBOD 数据流。
- 需要可维护、可 diff、可复查的架构图。

要求：

- 节点命名短而具体，优先使用真实层级名。
- 控制节点数量；超过 15 个节点时拆成多图。
- 图中只表达结构，不承担装饰职能。

### 2.2 生成图：用于封面与视觉解释

适用：

- 章节封面、专题页首图、概念隐喻图。
- 难以用真实照片覆盖的抽象主题，例如页缓存、内存一致性、调度延迟、故障域、恢复窗口。

要求：

- 构图清楚，有一个主要视觉焦点。
- 色彩以中性深灰、石墨、暖白、低饱和青绿、钢蓝、铜色、少量琥珀色为主。
- 不生成伪品牌、伪芯片型号、伪接口标识。
- 不生成可被误认为真实产品照片的硬件细节；真实硬件外观必须使用真实图片来源。

建议提示词方向：

```text
restrained editorial technical illustration, abstract but grounded, matte materials, precise layered structure, soft studio lighting, neutral graphite and warm white palette, subtle accent color, no neon, no glowing AI brain, no fake logos, no text
```

### 2.3 真实图片：用于硬件外观

适用：

- CPU package / die / wafer、DIMM、PCIe/CXL 卡、GPU accelerator、server motherboard、backplane、NIC/RDMA、NVMe SSD、HDD、JBOD/RAID enclosure。
- 读者需要识别物理形态、接口、插槽、散热器、托架、背板、连接器时。

要求：

- 优先使用官方 press/media kit、Wikimedia Commons、厂商产品页中明确可引用的媒体，或授权清楚的摄影来源。
- 每张图片插入前必须在 `image_attribution.md` 记录来源、许可、使用限制、下载日期和目标章节。
- 对官方 press assets 使用“按条款引用”的保守策略：保留版权方、产品名、原始 URL；不裁掉商标和关键上下文；不暗示厂商背书。
- 对 Wikimedia Commons 必须记录具体文件页、作者、许可证版本、是否要求署名、是否 ShareAlike。

## 3. 色彩与构图

### 3.1 推荐色彩

- 主背景：`#111315`、`#191C1F`、`#F4F1EA`、`#E8E3D8`
- 结构线：`#8A9299`、`#B7C0C7`
- 强调色：`#2F8F83`、`#B56B45`、`#D0A24A`、`#4D7EA8`
- 警示/故障：低饱和红棕或琥珀色，不使用刺眼纯红。

### 3.2 构图

- 封面图：一个主结构 + 少量环境线索；不要满屏塞满元件。
- 信息图：白底或浅灰底优先；深色底只用于封面或高层概念图。
- 硬件照片：优先展示可识别的整体外观，再在正文中用局部特写解释接口。

## 4. 章节插图策略

| 内容类型 | 首选图像 | 备用图像 | 注意 |
| --- | --- | --- | --- |
| OS 架构/调用路径 | Mermaid | 简洁矢量图 | 保持可维护性 |
| 调度、内存、I/O 概念 | 生成图 | Mermaid + 注释图 | 不伪造硬件 |
| CPU、DIMM、SSD、NIC 等外观 | 真实图片 | 官方产品渲染图 | 必须记录授权 |
| PCIe/CXL 拓扑 | Mermaid | 官方示意图 | 避免未经授权复制标准图 |
| GPU 服务器平台 | 官方/厂商图片 | Wikimedia Commons | 注意品牌和出口管制语境 |
| RAID/JBOD/backplane | 真实图片 + Mermaid 数据流 | 厂商手册图 | 手册截图需额外确认可用性 |

## 5. 文件命名与归档

后续如果下载并纳入仓库，建议命名：

```text
hubs/assets/images/{hub}/{chapter}/{source}-{subject}-{variant}.{ext}
```

示例：

```text
hubs/assets/images/hardware/09_cpu_architecture/intel-xeon6-package-press.jpg
hubs/assets/images/hardware/18_storage_hardware/wikimedia-nvme-m2-ssd.jpg
```

每次插图进入正文前，先在 `image_attribution.md` 添加一条记录。未确认授权的图片只能列为候选，不进入正文。
