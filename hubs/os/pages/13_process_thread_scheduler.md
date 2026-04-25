# 13 · 进程 / 线程 / 调度详解

## 学习目标

- 区分程序文件、进程、线程、task、调度实体。
- 能解释 `fork`、`clone`、`execve` 和 `exit_group` 在任务生命周期中的位置。
- 能用 `ps -T`、`top -H`、`vmstat`、`pidstat`、`perf sched` 观察线程和调度行为。
- 理解 Linux 6.6 后普通任务调度从 CFS 心智模型推进到 EEVDF 的现代锚点。

## 核心直觉

进程定义资源边界，线程定义执行流，调度器决定哪个可运行实体获得 CPU。

进程不是磁盘上的程序文件，而是运行时资源容器：地址空间、文件描述符、信号处理、凭证、一个或多个线程。线程也不是“更小的进程”，它的重要特征是共享进程级资源，但拥有自己的栈、寄存器状态和调度实体。

## 机制拆解

### 从命令到 CPU

```mermaid
sequenceDiagram
  participant Shell
  participant Kernel
  participant Loader
  participant Scheduler
  Shell->>Kernel: fork/clone
  Kernel->>Kernel: 复制或共享资源上下文
  Shell->>Kernel: execve
  Kernel->>Loader: 装载 ELF / 动态链接器
  Loader->>Kernel: mmap 动态库和栈
  Kernel->>Scheduler: task runnable
  Scheduler->>Kernel: pick next task
  Kernel-->>Shell: 任务获得 CPU
```

### 概念对照

| 概念 | 关注点 | 常见误区 |
| --- | --- | --- |
| 程序 | 磁盘上的可执行内容 | 把程序等同于进程 |
| 进程 | 资源边界 | 以为一定只有一个执行流 |
| 线程 | 执行流/调度实体 | 以为线程越多越快 |
| runnable | 已准备好，只等 CPU | 高 load 不一定等于 CPU 利用率高 |
| blocked | 等 IO、锁、事件 | 看起来“卡住”不一定在烧 CPU |
| context switch | CPU 从一个任务切到另一个任务 | 多不一定好，少也不一定坏 |

### EEVDF 的学习直觉

Linux 6.6 起主线调度逻辑转向 EEVDF。它仍追求公平，但不只靠“最小 vruntime”来理解。可以抓四个词：

| 概念 | 通俗理解 | 排障意义 |
| --- | --- | --- |
| runnable | 已准备好，只等 CPU | 高 load 可能是 runnable 积压 |
| lag | 任务是否欠 CPU 或多拿 CPU | 公平性不是简单轮流 |
| virtual deadline | 虚拟截止时间 | 延迟敏感任务可更快被选中 |
| slice | 任务期望运行片段 | 影响响应性和吞吐平衡 |

### 调度问题先分三类

| 现象 | 先看 | 继续深入 |
| --- | --- | --- |
| CPU 利用率高且业务慢 | `top`, `pidstat -u`, `perf top` | 热点函数、算法、锁、线程池 |
| load 高但 CPU 不高 | `vmstat 1`, `ps -eo stat,wchan,comm` | D 状态 IO、futex、网络等待 |
| 周期性卡顿 | cgroup `cpu.stat`, `cpu.pressure` | `CPUQuota=`, `cpu.max`, 父 slice |

调度器决定 runnable task 谁先跑；cgroup 决定一组 task 最多能拿多少 CPU。两者叠在一起时，应用日志通常只看到“慢”，系统侧证据才会显示 throttling 或 runnable 积压。

## 最小实验

### 实验 1：看任务创建链

```bash
strace -f -e trace=%process python3 -c 'print(1)'
strace -f -e execve,clone,exit_group python3 -c 'print(1)'
```

### 实验 2：看线程视图

```bash
ps -eLo pid,tid,psr,stat,comm | head
top -H
```

启动一个多线程程序后，对比进程视图和线程视图。

### 实验 3：观察争用

```bash
vmstat 1
pidstat -w 1
```

有 perf 权限时：

```bash
perf sched record -- sleep 5
perf sched latency
```

### 实验 4：把 CPU 配额接到调度视图

```bash
systemd-run --user --scope -p CPUQuota=50% bash
cg=$(sed -n 's/^0:://p' /proc/self/cgroup)
yes >/dev/null &
yes >/dev/null &
watch -n 1 "cat /sys/fs/cgroup$cg/cpu.stat; cat /sys/fs/cgroup$cg/cpu.pressure"
kill %1 %2
```

如果 `nr_throttled` 和 `throttled_usec` 持续增长，说明这不是普通“线程太多”，而是 cgroup 带宽上限在生效。退出临时 scope 前记得结束 `yes` 进程。

## 排障线索

- 高 load 不等于 CPU 算力不够。load 包含 runnable 和不可中断等待，需要结合 CPU 利用率、IO wait、D 状态任务看。
- 线程更多可能更慢：锁争用、cache 抖动、上下文切换、NUMA 远端访问会抵消并行收益。
- 被 cgroup `CPUQuota=` 限制的服务可能表现为周期性卡顿，应用层看不到“CPU 满”。
- Python/PyTorch 的 dataloader workers 是进程还是线程，会影响内存、IPC、调度、文件描述符和 cgroup 行为。
- 卡在 `futex` 时不要只看 syscall，要结合线程 dump、锁路径和调度延迟。

## 前沿/现代 Linux 连接

- EEVDF 是现代 Linux 调度的重要锚点；旧的 CFS 文档仍有历史价值，但学习时要知道主线已经变化。
- `sched_setattr()` 暴露了更细的调度属性，现代低延迟应用会更关注 slice 和延迟语义。
- 容器和 systemd 的 CPU 配额/权重会改变调度结果，调度问题经常要和 cgroup 一起查。
- AI 训练中 CPU 数据加载、线程池、NUMA、进程绑定会直接影响 GPU 利用率。
- sched_ext 让 BPF 程序定义调度策略，适合作为前沿观测点；普通排障仍应先把默认调度器、cgroup CPU 和应用线程模型解释清楚。

## 延伸阅读

- https://docs.kernel.org/scheduler/sched-eevdf.html
- https://docs.kernel.org/scheduler/sched-design-CFS.html
- https://docs.kernel.org/scheduler/sched-ext.html
- https://docs.kernel.org/admin-guide/cgroup-v2.html
- https://man7.org/linux/man-pages/man2/clone.2.html
- https://man7.org/linux/man-pages/man2/execve.2.html
- https://man7.org/linux/man-pages/man2/sched_setattr.2.html
