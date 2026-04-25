# 32 · PITR、WAL / Redo 与日志复制

## 定位

`PITR` 不是“多做几份备份”那么简单，而是 `基线副本 + 连续日志 + 明确停止点` 的组合能力。真正决定你能不能回到“误删前 3 分钟”的，不是备份窗口本身，而是日志链有没有保住、能不能按目标时点安全回放。

本章把 PostgreSQL WAL、MySQL binary log、Oracle redo / archived redo log 放到同一个恢复语义里理解：日志复制不是 PITR 的全部，停止点选择也不是纯技术命令。

## 学习目标

- 能解释 base backup、WAL/binlog/redo archive、recovery target 三者如何组成 PITR。
- 能区分 PITR、log shipping、replication 和 snapshot rollback。
- 能识别日志链断裂、基线不一致、停止点错误和归档同域失效等风险。
- 能设计一次数据库误删后的停止点定位和恢复验证演练。

## 核心直觉

PITR 的核心是三条链：

```mermaid
flowchart LR
  A[Base backup] --> B[WAL / binlog / redo archive]
  B --> C[Recovery target time / LSN / SCN]
  C --> D[Restore and replay]
  D --> E[Validate application state]
```

关键判断：

1. 没有基线副本，日志不能单独还原完整系统状态。
2. 没有连续日志链，就没有真正 PITR。
3. 没有停止点记录，就只能“恢复到最新”或靠猜。
4. 日志链和基线备份应跨故障域保存。
5. 停止点选择是业务判断，不只是技术命令。

## 机制边界

### 基线链

- 基线链是可作为恢复起点的 `base backup`、全量备份或一致性快照。
- 基线必须和后续日志链匹配；如果基线时间、timeline、server UUID 或数据库版本不清楚，恢复会变成猜测。
- 基线副本也需要校验、保留和跨故障域保存。

### 日志链

- 日志链记录从基线之后发生的变化。
- PostgreSQL 的核心是 `WAL`；MySQL 典型是 `binary log`；Oracle 体系里是 `redo` 与 `archived redo log`。
- 日志链断了，恢复窗口就断了；日志链被污染，恢复结果也会被污染。

### 停止点链

- 日志不是必须一直回放到最新。
- PostgreSQL 文档支持按时间、命名 restore point、事务 ID、LSN 等方式设置 recovery target。
- MySQL PITR 依赖 full backup 之后的 binary log，可以通过 `mysqlbinlog` 按时间或位置回放。
- 停止点应结合业务事件、审计日志、变更记录和应用校验来确定。

### PITR 不直接解决的问题

- 不天然解决勒索传播，因为被污染后的日志也可能被完整归档。
- 不天然等于异地容灾，因为日志归档和备份可能仍在同一故障域里。
- 不天然等于热备库，因为热备强调接近当前，PITR 强调可按历史点恢复。

## 架构/流程

### PostgreSQL: base backup + WAL archive + recovery target

- PostgreSQL 当前文档把这条路称为 `Continuous Archiving and Point-in-Time Recovery (PITR)`。
- 基本模式是保留一个 base backup，再持续归档 WAL。
- 恢复时先还原基线，再通过 `restore_command` 取回 WAL，回放到指定 recovery target。
- 同一份基线和 WAL 也可以用于 warm standby，但 standby 和 PITR 的目标不同。

### MySQL: full backup + binary log + mysqlbinlog

- MySQL 8.4 文档说明，PITR 的信息源是 full backup 之后生成的 binary log files。
- 如果想做时间点恢复，必须开启并保留足够时间的 binary log。
- `mysqlbinlog` 负责把目标时间点或事件位置之前的日志事件重新执行到恢复目标上。

### Oracle / Data Guard: redo transport + apply services

- Oracle Data Guard 的 redo transport services 负责把 redo 从生产库传到归档目标或备用库，apply services 负责落地回放。
- `standby redo log` 可支持 real-time apply；没有时则依赖归档后再应用。
- 这说明日志复制与日志归档恢复处在同一技术栈，但服务目标不同。

最小 PITR runbook：

1. 确认事故时间、影响对象和目标停止点。
2. 选择事故前可用基线副本。
3. 拉取从基线起点到停止点所需的连续日志。
4. 在隔离环境恢复基线并回放日志。
5. 在目标停止点暂停，避免把误删或污染事件也回放进去。
6. 做数据库一致性、行数、关键业务查询和应用启动验证。
7. 决定是导出修复数据、替换生产库，还是继续做差异合并。

## 常见故障

### 只做全量备份，不保连续日志

- 故障表现：只能回到备份时刻，无法回到误操作发生前的分钟级时间点。
- 判断方法：检查从 base backup start 到故障时间是否有完整 WAL/binlog/redo。
- 修正方向：启用归档并设置保留窗口、监控归档失败、定期演练回放。

### 日志和基线放在同一故障域

- 故障表现：生产存储或账号失效时，备份和日志一起不可用。
- 判断方法：检查归档桶、备份仓库、KMS 和权限是否与生产共域。
- 修正方向：跨账号、跨区域或隔离仓库保存基线和日志。

### 停止点选错

- 故障表现：恢复后误删仍存在，或丢掉了事故前后的有效业务数据。
- 判断方法：把业务审计、数据库日志和应用日志对齐到统一时间线。
- 修正方向：使用命名 restore point、变更窗口记录和隔离环境预验证。

### 日志回放从未演练

- 故障表现：恢复命令、timeline、binlog 位置、权限或版本兼容性在事故时才暴露。
- 判断方法：定期抽样从旧基线恢复到指定时间点。
- 修正方向：把 PITR 演练纳入备份验收，而不是只做最新恢复。

## 演练方法

### 演练 1：画一张 `base backup -> log archive -> recovery target` 恢复链

- 分别为 PostgreSQL、MySQL、Oracle 画三种最小闭环。
- 目标：把抽象概念变成恢复路径图。

### 演练 2：给一次误删事件设计停止点

- 设定业务在 `14:32` 误删关键表。
- 设计如何定位 `14:31:59`、某个命名 restore point、LSN/SCN 或 binlog 位置。
- 目标：训练“停在哪”而不是“只知道往前回放”。

### 演练 3：做一张 `PITR vs Replication vs Snapshot Rollback` 对照表

- 对比粒度、时延、复杂度、故障域和适用场景。
- 目标：避免把三者写进同一个采购需求里。

### 演练 4：设计一次日志链断裂应急演练

- 假设一段归档日志缺失。
- 说明恢复窗口会怎样截断，如何向业务方解释。
- 目标：让团队接受“可恢复窗口是工程约束，不是口号”。

## 治理/合规判断

- PITR 窗口应与业务 RPO、审计保留和变更管理记录一致。
- 归档日志包含敏感数据变化，应与备份一样纳入加密、访问控制和删除治理。
- 关键变更前应建立命名 restore point 或明确变更时间线，降低恢复停止点争议。
- 恢复证明应包括基线标识、日志范围、停止点、校验查询和业务验收。

## 前沿趋势

- 云数据库把 PITR 做成托管能力，但底层仍是基线加日志链；团队仍要理解恢复窗口、停止点和克隆验证。
- PostgreSQL 生态持续强化 WAL archive、timeline 和自动化恢复工具链，适合把 PITR 标准化到平台层。
- MySQL 8.4 LTS 让 binary log 成为长期支持版本里的关键恢复与复制基础。
- Cyber recovery 场景下，日志链也需要恶意行为时间线和 clean room 验证，不能盲目回放到最新。

## 本页要配套记住的概念卡

- Point-in-Time Recovery (PITR)
- Write-Ahead Log (WAL)
- Binary Log
- Redo Log
- Log Shipping
- Recovery Target
- Relay Log

## 延伸阅读

- PostgreSQL Continuous Archiving and Point-in-Time Recovery (PITR): https://www.postgresql.org/docs/current/continuous-archiving.html
- MySQL 8.4 Point-in-Time Recovery Using Binary Log: https://dev.mysql.com/doc/refman/8.4/en/point-in-time-recovery-binlog.html
- MySQL 8.4 The Relay Log: https://dev.mysql.com/doc/refman/8.4/en/replica-logs-relaylog.html
- Oracle Data Guard Concepts and Administration: https://docs.oracle.com/en/database/oracle/oracle-database/26/sbydb/introduction-to-oracle-data-guard-concepts.html
