# 18 · 系统排障方法论

## 学习目标

- 建立从现象到证据再到根因的稳定排障流程。
- 能把问题先定位到正确层级，而不是到处试参数。
- 能把一次排障沉淀成可复用 casebook 条目。
- 能把 OS Hub 前面章节串成“现象 -> 机制 -> 证据 -> 修复”的工作流。

## 核心直觉

排障不是找一个神奇命令，而是把模糊现象稳定收敛成证据链。先定义问题，再缩小边界；先看低成本证据，再进入高分辨率 trace；一次只改一个变量，并用同一个指标验证。

根因要写成可验证句子。例如：

- 不写“程序有 bug”，写“service 在 systemd unit 下运行，`PrivateTmp=` 改变了 `/tmp` 可见性，导致应用找不到 socket 文件”。
- 不写“NCCL 不稳定”，写“容器默认 `/dev/shm` 太小，多进程初始化创建共享内存失败”。

## 机制拆解

### 五层问题树

| 层 | 问题 | 产物 |
| --- | --- | --- |
| 现象层 | 用户感受到什么 | 时间、范围、影响、错误信息 |
| 边界层 | 问题在哪个边界内 | 单进程/单机/容器/网络/存储/GPU |
| 系统层 | 涉及哪些机制 | 调度、内存、IO、隔离、权限、cgroup |
| 证据层 | 哪些输出支持判断 | 命令输出、日志、trace、指标 |
| 根因层 | 为什么会发生 | 可复现解释与修复 |

### 标准 runbook

```text
1. Freeze: 记录时间、命令、版本、负载、错误信息
2. Scope: 判断是否单用户、单进程、单机、单集群
3. Baseline: 采集 CPU/内存/IO/网络/GPU/日志
4. Trace: 对最可疑路径使用 strace/perf/eBPF
5. Change: 只改一个变量
6. Verify: 用同一个观测指标验证
7. Write: 沉淀到 casebook
```

### 问题类型到入口工具

| 现象 | 第一入口 |
| --- | --- |
| 启动失败 | `systemctl status`, `journalctl -u`, `strace` |
| 权限/路径异常 | `strace -e trace=%file`, `id`, mount/env 对比 |
| OOM/被杀 | `journalctl -k`, cgroup `memory.events` |
| CPU 慢/抖动 | `top`, `pidstat`, `perf`, cgroup `cpu.stat` |
| IO 慢 | `iostat`, `strace`, mount/VFS/cgroup IO |
| 容器才失败 | namespace/cgroup/mount/user mapping 对比 |
| 多卡通信异常 | `/dev/shm`, `NCCL_DEBUG`, topology, network |

### 总决策树

```mermaid
flowchart TD
  A[用户现象] --> B{能否稳定复现?}
  B -- 否 --> C[先补时间窗、日志、指标、采样]
  B -- 是 --> D{只在 service/container/某用户下出现?}
  D -- 是 --> E[对比 env/cwd/user/namespace/cgroup/mount]
  D -- 否 --> F{资源是否异常?}
  F -- CPU --> G[cpu.stat, cpu.pressure, perf, 调度]
  F -- 内存 --> H[memory.events, memory.stat, OOM 日志]
  F -- IO --> I[strace, iostat, io.stat, VFS/mount]
  F -- 网络 --> J[ss, route, DNS, connect trace]
  F -- GPU --> K[/dev/shm, NCCL_DEBUG, topology, runtime]
  E --> L[最小变更验证]
  G --> L
  H --> L
  I --> L
  J --> L
  K --> L
  L --> M[casebook: 证据、根因、修复、预防]
```

### 跨章节证据矩阵

| 机制 | 最小证据 | 回看章节 |
| --- | --- | --- |
| syscall 行为 | `strace -f -ttT`, errno, fd/path | 08 |
| cgroup 资源 | `/proc/<pid>/cgroup`, `cpu.stat`, `memory.events` | 09, 11 |
| 隔离视图 | `lsns`, `/proc/<pid>/ns`, mountinfo | 10, 17 |
| systemd 执行环境 | `systemctl show`, `journalctl -u` | 11 |
| 共享内存/GPU 通信 | `/dev/shm`, NCCL 日志, topology | 12 |
| 调度/CPU | `pidstat`, `perf`, `cpu.pressure` | 13, 16 |
| 内存/page cache | `memory.stat`, `/proc/<pid>/status` | 14 |
| VFS/IO | `findmnt`, `stat -f`, `iostat`, `io.stat` | 15 |

## 最小实验

### 实验 1：写一份基线

```bash
date
uname -a
uptime
free -h
df -h
systemctl --failed --no-pager
journalctl -k -n 50 --no-pager
```

### 实验 2：对一个失败命令生成证据

```bash
strace -f -ttT -o /tmp/failure.trace your_command_here
grep -E 'ENOENT|EACCES|EPERM|ETIMEDOUT|ECONNREFUSED|OOM|killed' /tmp/failure.trace
```

### 实验 3：最小 casebook 模板

```markdown
## 案例：一句话标题

- 时间：
- 影响范围：
- 用户现象：
- 环境版本：
- 初始假设：
- 关键证据：
- 根因：
- 修复：
- 验证：
- 以后如何预防：
```

### 实验 4：15 分钟 runbook drill

选一个低风险命令，故意制造一个失败，再按固定模板收敛：

```bash
systemd-run --user --unit=os-drill -p WorkingDirectory=/no/such/dir /bin/true
systemctl --user status os-drill --no-pager
journalctl --user -u os-drill -n 50 --no-pager
systemctl --user show os-drill -p Result -p ExecMainStatus -p WorkingDirectory
```

目标不是记住这个错误，而是练习“现象 -> systemd 证据 -> 根因句子 -> 修复验证”。把 `/no/such/dir` 换成权限、cgroup、容器 mount 或 `/dev/shm` 案例，就是同一套方法。

## 排障线索

- 没有现象定义就不要改配置。
- 一次改多个变量会破坏证据链。
- “恢复了”不等于“找到根因”，要写清楚为什么恢复。
- 先确认边界：同一命令在 shell、systemd service、容器、不同用户下的差异通常很关键。
- 参数修改如果只是扩大资源、关闭隔离、绕过缓存，需要标记为临时缓解，不要当作根因。

## 前沿/现代 Linux 连接

- 现代 Linux 排障要把 systemd、cgroup v2、namespace、eBPF、perf、GPU runtime 作为同一套证据系统。
- Casebook 的价值是压缩经验：保留可复现实验、关键输出、根因句子和预防动作。
- AI/GPU 系统问题常跨层：Python 日志只是表层，根因可能在共享内存、拓扑、NCCL、网络、cgroup 或设备权限。
- 对生产系统，优先使用低扰动观察；高开销 profiling、清 cache、重启服务、扩大隔离边界都需要明确风险。

## 延伸阅读

- https://docs.kernel.org/admin-guide/
- https://docs.kernel.org/trace/index.html
- https://docs.kernel.org/admin-guide/perf/index.html
- https://man7.org/linux/man-pages/man1/strace.1.html
- https://www.brendangregg.com/linuxperf.html
- https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/
- https://kubernetes.io/docs/tasks/debug/
