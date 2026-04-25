# Agents、Multi-Agent 与 Benchmark 地图

> 这页不是术语表，而是一张地形图。读完它，你应该能回答四个问题：什么才算 agent；什么时候多智能体真有价值；不同 benchmark 到底在测什么；看到排行榜时应该先怀疑什么。

## 1. 先把“agent”这个词拆开

2026 年谈 agent，最容易犯的错误是把所有“会调工具的模型”混成一个概念。Anthropic 在《Building Effective AI Agents》里给了一个非常实用的分界：`workflow` 是 LLM 和工具沿预定义代码路径协同；`agent` 则是模型自己决定过程和工具使用方式。这个区分不是字面游戏，而是工程边界。前者的核心问题是流程设计是否清晰，后者的核心问题则是模型在不确定环境里是否还能作出稳定判断。

OpenAI 的 evaluation best practices 又补上了第二层视角：系统复杂度通常会沿着“单次调用 -> workflow -> single-agent -> multi-agent”上升，而不确定性会在每一层增加。也就是说，所谓“更 agentic”，并不等于“更先进”；它常常只是“把更多控制权交给模型，因此也把更多责任交给评估系统”。

真正有用的判断方式不是问“这个系统里有几个 agent”，而是问：

1. 模型拥有多少自主权？
2. 它面对的是一个静态 API，还是一个会变化、会出错、会反过来影响它的环境？
3. 一旦失败，你能否知道它究竟错在理解、规划、执行，还是环境摩擦？

如果这三件事没想清楚，多智能体往往只会把一个难调的系统变成四个更难调的系统。

## 2. 地图的三条轴：任务、环境、评估

理解 agent，不应该从“框架名”开始，而应该从三条坐标轴开始。

### 2.1 任务轴：从固定流程到开放任务

固定流程问题的特征，是你能提前写出大部分步骤。比如“抽取结构化字段 -> 调数据库 -> 生成回复”，更像 workflow。开放任务则不同，你只能给出目标，不能预先写出完整路径，例如“去网页上完成预订”“在代码库里修复一个真实 bug”“在桌面环境里完成跨应用任务”。

任务越开放，系统越依赖模型在中途修正路线；而一旦允许修路，你就必须接受绕路、迷路和走错路。

### 2.2 环境轴：从干净接口到脏世界

API 型工具环境最干净，输入输出结构化、错误边界清晰、状态也相对可控。网页环境开始变脏，因为 DOM、视觉布局、异步加载、登录态、广告、弹窗、反爬机制都会引入摩擦。桌面环境更脏，因为任务不仅跨页面，还跨应用、跨文件、跨 OS 状态。代码库环境则把“环境”扩展为仓库历史、测试、依赖、补丁、副作用与回归风险。

环境越真实，benchmark 的含金量通常越高；但与此同时，分数越难解释，因为失败不再只是“模型不会”，还可能是“环境太嘈杂”“工具契约太差”“脚手架有偏差”。

### 2.3 评估轴：从结果对错到过程可解释

黑箱式 benchmark 只告诉你成没成功，无法告诉你为什么成功或失败。OpenAI 在 `Trace grading` 和 `Evaluate agent workflows` 里强调，trace 是端到端的决策、工具调用、handoff 与 guardrail 记录；只有把这些过程也纳入打分，系统才真正进入可迭代阶段。换句话说，agent 的评估不能只看“答案对不对”，还要看：

- 选错了工具没有？
- 参数传错了没有？
- 明明该 handoff 却没 handoff？
- 不该读到的隐私数据是否被带进了上下文？
- 失败后是否有恢复路径？

一旦进入 multi-agent，这些问题会成倍增长，因为每一次 triage 和 handoff 都会新增不确定性。

## 3. 一张更有解释力的 agent 地形图

下面这张表，不是按“热门程度”分类，而是按“系统真正面对的世界”分类。

| 层级 | 典型代表 | 真正在测什么 | 最容易被误读的地方 |
| --- | --- | --- | --- |
| API / tool workflow | OpenAI Agent Builder、Anthropic tool use | 工具选择、参数抽取、流程编排、guardrails | 把 prompt 成功误当成系统稳定 |
| Web agent | WebArena、VisualWebArena、WebArena-Infinity | 浏览器交互、页面状态理解、长程网页任务 | 以为会点网页就等于会用电脑 |
| Desktop / computer-use agent | OSWorld、Anthropic computer use | 视觉定位、跨应用操作、桌面状态管理、长时交互 | 忽略视觉噪声和环境摩擦的巨大影响 |
| Coding agent | SWE-bench | 长上下文代码理解、编辑、运行测试、回归修复 | 把代码生成能力等同于真实工程修复能力 |
| Novel interactive reasoning | ARC-AGI-2、ARC-AGI-3 | 新问题适应、抽象规则发现、交互式探索 | 把它和工作流 agent benchmark 混为一谈 |

这五层对应的不是五个赛道，而是五种不同的“外部世界”。模型能否在某一层表现良好，取决于它是否能在那个世界里获得足够可靠的反馈并修正行为。

## 4. Benchmark 家族：它们分别在告诉你什么

### 4.1 WebArena 系：浏览器并不是一个简化版 API

WebArena 论文的出发点很直接：许多早期 benchmark 过于合成化，和真实网页世界脱节。WebArena 因而强调“高度真实且可复现”的网页环境。它重要，不是因为它让 agent 学会点击网页，而是因为它第一次系统性地把网页任务里的三个问题合并起来了：

- 自然语言目标如何落地成一串交互动作；
- 页面中间状态是否可验证；
- 任务成功到底该看最终答案，还是看中途状态是否符合意图。

WebArena 官方站点后来扩展成了 `WebArena-x` 套件。到 `2026-04-22` 前后，公开站点已经把 WebArena、WebArena-Infinity、VisualWebArena 和 TheAgentCompany 放到同一个连续谱里看待。这件事很关键，因为它说明 web agent 评测正在从“静态网页任务”走向三种更接近现实的维度：

1. 视觉依赖更强：VisualWebArena 明确指出很多真实任务必须读图、看布局、理解页面视觉线索，仅靠文字或 accessibility tree 不够。
2. 环境持续演化：WebArena-Infinity 强调的是“evolving environments”，也就是 benchmark 本身不再假设世界是冻结的。
3. 社会化组织任务：TheAgentCompany 不再只是网页导航，而是把 agent 放进模拟企业任务流，开始接近组织协作。

因此，WebArena 最重要的启发不是“browser agent 能做什么”，而是“只要环境开始变脏，planning、grounding、verification 就必须一起设计”。

### 4.2 OSWorld：网页 agent 和电脑 agent 不是一个难度级别

OSWorld 论文的核心论断非常朴素：已有 benchmark 要么没有真正交互环境，要么局限在特定应用/领域，因而不能反映现实电脑使用的复杂性。它把 agent 从网页扩展到了真实桌面世界：浏览器、办公软件、文件系统、跨应用工作流、不同操作系统。

截至 `2026-04-22`，OSWorld 官方站点已经明确写出在 `2025-07-28` 做了 `OSWorld-Verified` 升级：修复社区反馈、支持 AWS、把评测时间压到 1 小时以内，并给出统一设置下的官方结果。这说明桌面 agent 评测正在发生一个重要变化：从“研究演示”转向“更可验证、可重复、可比较”的评测基础设施。

OSWorld 最有价值的地方，不在于分数，而在于它逼迫你接受一个事实：电脑使用不是单一智能能力，而是至少四种能力的耦合。

- 视觉感知：看懂截图、图标、布局、控件状态。
- 操作规划：知道下一步该点哪里、输什么、何时等待。
- 状态记忆：记住任务目标、已完成步骤和跨应用上下文。
- 错误恢复：点错、找不到、窗口变化、权限不足时是否会重规划。

Anthropic 的 computer use 文档和 OSWorld 恰好互为镜像。前者展示了 agent loop、sandbox、截图与鼠标键盘控制的产品化接口；后者则给出一个更中立的研究环境，用来检验这类 agent 在真实桌面摩擦下是否还能站住。两者放在一起读，你会得到一个更清楚的结论：computer use 的难点从来不只是“能不能操作桌面”，而是“能否在视觉噪声和外部状态变化下持续校正自身行动”。

### 4.3 SWE-bench：从“会写代码”到“能修系统”

SWE-bench 论文把任务定义得非常直接：给定代码库和一个真实 issue，让模型修改代码并通过测试。它之所以在 coding agent 领域具有特殊地位，是因为它把问题从“生成一段代码”改写成了“进入一个已有系统，对真实约束负责”。

这会瞬间改变评估重点。真正困难的地方，不再是语法，而是：

- 需要跨多个文件理解上下文；
- 需要在很长的上下文里定位真正相关的部分；
- 需要运行测试获得环境反馈；
- 需要避免修了一个 bug 却引入另一个回归。

Anthropic 在 Building Effective AI Agents 里把 coding agent 当作 agent 适配度极高的典型案例，原因正是代码世界存在明确验证器，也就是测试。测试让 agent 拿到了“地面真值”，这对 agent 工程化极其重要。没有验证器的任务，agent 往往更像会自信地漂移的写作者；有验证器的任务，agent 才开始像一个能被约束的执行系统。

截至 `2026-04-22`，SWE-bench 官方站点又把信号往前推了一步：它已经不只强调开源仓库 issue 修复，而是突出“private codebases”“verified fixes”“multilingual coverage”。这意味着 coding agent 评测正在从公开竞赛语境，转向更接近企业真实软件环境的语境。也因此，SWE-bench 分数再高，也不能直接等价为“企业代码库里就能稳用”，但它至少比传统代码生成 benchmark 更接近真实工程劳动。

### 4.4 ARC-AGI-2 / 3：它不是 workflow benchmark，而是“新问题学习”压力测试

如果 WebArena、OSWorld、SWE-bench 问的是“agent 在既定外部世界里能不能干活”，ARC 系列问的则是另一个问题：系统能否面对陌生任务，快速形成规则理解，并且高效地解题。

ARC Prize 官方站点在 ARC-AGI-2 页面里反复强调两个主题。第一，`scale is not enough`；第二，`efficiency` 也是 intelligence 的一部分。换句话说，它关心的不只是能不能 brute force 出答案，而是系统能否以接近人的效率发现新规则。到 `2026-04-22`，ARC Prize 2026 页面又进一步公开了 `ARC-AGI-3`，并明确把它叫作 “the first interactive reasoning benchmark”。这意味着 ARC 也在向“交互式、探索式、agentic”的方向演化。

这对 agent 研究有一个很重要的启发：现实系统不只需要完成熟悉工作流，还需要在陌生局面下探索和适应。很多 workflow agent 在熟悉场景里很好用，但一遇到新界面、新规则、新异常就会迅速崩塌。ARC 系列正好提醒你，真正强的 agent 不只是“按既定操作册执行”，还必须具备足够强的抽象和迁移能力。

## 5. Multi-agent 什么时候是结构，什么时候只是装饰

多智能体最常见的错觉，是把“拆成多个角色”误当成“提高系统能力”。Anthropic 的建议非常克制：从简单、可组合的模式开始，只有当复杂性被证明能带来可测的收益时，才继续增加层数。OpenAI 的 eval 文档也表达了同样立场：single-agent 到 multi-agent 的升级，应该由 eval 驱动，而不是由审美驱动。

真正值得用 multi-agent，通常只有四种情况。

### 5.1 工具面太宽，单个 agent 已经不稳定

当一个 agent 同时面对太多工具、太多任务类型、太多策略冲突时，它会开始在“该不该调用工具”和“到底该调哪个工具”上频繁漂移。此时把 triage、领域执行、审核拆开，能减少单个策略头的负担。

### 5.2 不同子任务需要完全不同的世界模型

例如一个 research system 同时包含网页检索、文档抽取、代码运行和结论撰写。它们需要的上下文、工具、容错规则都不同。把它们塞进单个 agent 往往会得到一个“什么都知道一点，但哪一步都不够稳”的系统。

### 5.3 需要显式的校验、辩论或仲裁

当任务存在明显的“生成者/评审者”关系时，多智能体结构是合理的。Anthropic 的 evaluator-optimizer 模式本质上就是最小多智能体：一个负责提案，一个负责批判。关键不在于“两个 agent 比一个高级”，而在于评价标准是否清晰、反馈是否真能改进输出。

### 5.4 需要并行探索

网页探索、检索、候选方案生成这类任务，有时天然适合 worker 并行，因为多个候选路径可以同时展开，再由上层汇总。但只有在并行带来的收益大于状态同步成本时，这才成立。

反过来说，以下情况不值得上 multi-agent：

- 单 agent 配上更好的工具描述和验证就够了；
- 所谓“角色分工”只是 prompt 语气不同，工具权限和责任边界却完全一样；
- handoff 本身比执行还贵，导致系统主要时间都耗在协调上；
- 你没有 trace 与 eval，只是凭感觉觉得“多几个脑子会更聪明”。

多智能体的哲学不是“越像组织越高级”，而是“是否需要引入组织才能管理复杂性”。如果组织本身就是复杂性的来源，那它就不是解法。

## 6. 如何读 benchmark，而不是被 leaderboard 读你

排行榜的诱惑在于它把复杂世界压缩成一个分数。但 agent benchmark 尤其不能只看分数，因为同一个数字背后可能是完全不同的系统结构。看 leaderboard，至少要先问五个问题。

### 6.1 它测的是模型，还是测脚手架

很多 agent 成绩来自强脚手架而不只是强模型。搜索算法、回溯策略、set-of-marks、specialized grounding model、缓存和人为约束都会显著改写结果。真正该问的不是“谁第一”，而是“第一名把哪些能力放在模型里，哪些放在系统里”。

### 6.2 它测的是开放性，还是熟悉度

WebArena、OSWorld、SWE-bench 都在努力降低环境过度合成化的问题，但“可复现”与“真正开放”之间始终有张力。环境越标准化，越利于比较；环境越开放，越接近现实，却也越难统一评测。这意味着 benchmark 分数更适合作为方向性信号，而不是生产力保证书。

### 6.3 它有没有把过程也纳入评估

只看最终成功率，常常会掩盖大量脆弱路径。OpenAI 强调 trace grading，原因就在这里。一个系统也许最终完成了任务，但过程里可能做了很多高风险动作：读了不该读的数据、调用了不必要的工具、绕了很远的路、碰到异常后纯靠运气恢复。只看结果，会把这些风险全部抹平。

### 6.4 它是否区分官方验证与自报结果

OSWorld-Verified、ARC Prize Verified 这类机制的重要性正在上升，因为 agent 系统的结果越来越依赖环境、脚手架和运行细节。只要评测不是由同一套环境和审计流程完成，结果的可比性就会快速下降。

### 6.5 它的成本结构是什么

ARC-AGI-2 把效率明确纳入 benchmark，是一个重要信号。一个分数如果必须靠极高成本、极多 rollout 或极重搜索才能拿到，和人们想象中的“通用可用 agent”其实是两回事。系统研究迟早都会回到这个问题：不是“能不能”，而是“值不值”。

## 7. 研究这组主题时，应该形成怎样的阅读顺序

如果你想把这组知识真正吃透，而不是只记住几个 benchmark 名字，建议按下面的顺序来。

### 第一步：先读 agent 定义，而不是先看排行榜

先用 Anthropic 的 workflows vs agents 区分，和 OpenAI 的 workflow / single-agent / multi-agent 架构分层，建立“控制权与不确定性一起增长”的基本心智模型。

### 第二步：再按环境层读 benchmark

先看 WebArena，因为它是网页交互的基础层；再看 OSWorld，因为它把问题扩展到真实电脑环境；再看 SWE-bench，因为它代表代码世界中的可验证 agent；最后读 ARC，因为它提醒你别把工作流能力误认为一般智能。

### 第三步：最后才谈 multi-agent

先问单 agent 为什么不够，再问多 agent 如何拆。否则你很容易在没有必要的地方引入 triage、planner、reviewer、executor，最后得到一个分工漂亮、性能平庸、解释困难的系统。

## 8. 这一页真正想留下的判断

agent 研究最值得投入的，不是“再堆一个会调工具的 demo”，而是把以下关系看透：

- agent 的价值来自环境反馈，而不是来自拟人化叙事；
- multi-agent 的价值来自复杂性管理，而不是来自角色扮演；
- benchmark 的价值来自暴露系统边界，而不是制造短暂排名；
- 最终会留下来的系统，不是最花哨的，而是最能把任务、工具、环境和评估闭合起来的。

把这四句话记住，再去看任何 agent 产品、论文、框架或排行榜，你会清醒很多。

## 9. 三个 leaderboard 误读案例：为什么很多“高分判断”一开始就问错了问题

真正危险的不是 leaderboard 本身，而是人们总想让一个数字替他们完成概念分辨。下面三个案例都来自官方 benchmark 页面已经明确写出的信息，但恰恰因为这些信息不够刺激，常常被读榜的人自动忽略。

### 9.1 误读案例一：把 SWE-bench Verified 总榜读成“模型榜”

SWE-bench Verified 官方页其实把这件事区分得非常清楚。它一方面说 `The Verified leaderboard features results from a wide variety of AI coding systems`；另一方面又单独设了 `Bash Only: Comparing Language Models`，明确表示这是为了 apples-to-apples 地比较语言模型，在 `mini-SWE-agent` 的最小 bash 环境中统一评测。

这意味着同一个 benchmark 名字下面，至少并存两种不同比赛：

- 一种在比“系统包”，也就是模型、检索、review loop、verifier、rollout、工具约束一起形成的综合能力；
- 另一种更接近在比“模型本体”，因为它尽量削弱脚手架差异。

如果你把前者当成后者，就会高估“换个模型就能复刻成绩”；如果把后者当成前者，又会低估系统工程的真实价值。这个误判之所以常见，是因为人们喜欢一个单一叙事：谁的模型最强。但 coding agent 已经不是那个世界了。

### 9.2 误读案例二：把 OSWorld 旧结果和 OSWorld-Verified 新结果放进同一张表

OSWorld 官方站点在 `2025-07-28` 的升级说明里写得非常直接：OSWorld 已增强为 `OSWorld-Verified`，修复了社区反馈样例、支持 AWS、更新了 benchmark results，并明确要求使用最新版时应与新的 verified results 比较。

这不是一个无足轻重的注脚，而是一个比较学警报。因为只要 benchmark 的任务修复、环境设置、评测基础设施发生实质变化，旧结果与新结果的可比性就会立刻下降。很多“能力飞跃”的社交媒体叙事，实际混在一起的可能是两件事：

- agent 本身确实更强了；
- benchmark 本身也被清洗、统一、重构了。

两者都可能成立，但你必须拆开。否则你会误把版本变迁讲成纯能力跃迁。

### 9.3 误读案例三：只报 ARC-AGI-2 准确率，不报效率

ARC-AGI-2 官方页面几乎把这个问题写成了大字报：从 ARC-AGI-2 开始，所有 ARC-AGI reporting 都要附带 efficiency metric，因为 intelligence 不只是“能否解题”，还包括“以什么效率解题”。页面甚至明确说，brute-force search 在无限资源下也许最终能做对题，但这不代表真正智能。

这对 leaderboard 阅读方式有一个直接后果：如果你只报准确率，而不报成本、搜索预算或 test-time adaptation 代价，你其实是在删掉 benchmark 设计者刻意保留下来的半个 benchmark。你看到的不是“同一个指标的简化版”，而是“被改写过的问题”。

因此，ARC-AGI-2 的读法必须是双维度的：分数告诉你能不能解，效率告诉你这种解法究竟更像 intelligence 还是更像 brute force。

## 10. 一个实用的 leaderboard 判别框架：先拆四层，再决定信不信

任何一个 agent leaderboard，只要你打算让它影响判断，都应该先经过下面四层过滤。

### 第一层：它在测哪个世界

网页世界、桌面世界、代码库世界、抽象推理世界，看上去都叫“agent benchmark”，但它们的环境结构完全不同。先问清 benchmark 的世界是什么，再谈外推。

### 第二层：它在比谁

是在比裸模型、最小脚手架、还是完整系统包。这个问题不先拆开，后面所有“谁更强”的讨论都会混层。

### 第三层：它怎么判

只看最终结果，还是也看中间状态、trajectory、官方验证、human baseline、校准集。判法越丰富，分数越难简化成一句话；但也越有解释力。

### 第四层：它花了什么代价

token、rollout、wall-clock、人工参与、specialized components、环境准备成本，这些都决定了高分究竟是“能力更强”，还是“预算更大”。ARC-AGI-2 把效率显式写进 benchmark，恰恰是在提醒大家：成本不是附注，而是能力解释的一部分。

这个四层框架的意义，不是让你变得更挑剔，而是让你不至于把不同层级的东西压扁成一个漂亮数字。

## 11. 讨论题：你会怎样读一张 agent 榜单

1. 看到 SWE-bench Verified 某系统高分时，你要做的第一个动作为什么不是看模型名字，而是看它属于哪一类评测栏位。
2. 一个团队把 OSWorld 2024 论文里的结果和 OSWorld-Verified 的新结果并排画趋势线，这条图最可能在哪一步误导决策者。
3. 如果 ARC-AGI-2 的准确率升了，但效率恶化了一个数量级，这更像“能力进步”还是“搜索预算进步”。你的答案取决于什么定义。
4. 一个 browser agent 在 WebArena 家族成绩很好，但在你的真实企业流程里频繁卡在权限、异常状态和长尾表单上。你会如何向只看排行榜的人解释，这不是 benchmark 无价值，而是 benchmark 的世界边界到了。

## 来源

- Anthropic, *Building Effective AI Agents*  
  https://www.anthropic.com/engineering/building-effective-agents
- OpenAI, *Evaluation best practices*  
  https://developers.openai.com/api/docs/guides/evaluation-best-practices
- OpenAI, *Evaluate agent workflows*  
  https://developers.openai.com/api/docs/guides/agent-evals
- OpenAI, *Trace grading*  
  https://developers.openai.com/api/docs/guides/trace-grading
- WebArena-x official site  
  https://webarena.dev/
- WebArena official page  
  https://webarena.dev/og/
- VisualWebArena official page  
  https://jykoh.com/vwa
- WebArena paper, arXiv:2307.13854  
  https://arxiv.org/abs/2307.13854
- OSWorld official site  
  https://os-world.github.io/
- OSWorld paper, arXiv:2404.07972  
  https://arxiv.org/abs/2404.07972
- SWE-bench Verified  
  https://www.swebench.com/verified.html
- mini-SWE-agent  
  https://mini-swe-agent.com/
- Anthropic, *Computer use tool*  
  https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool
- SWE-bench official site  
  https://swebench.ai/
- SWE-bench paper, arXiv:2310.06770  
  https://arxiv.org/abs/2310.06770
- ARC Prize Foundation  
  https://arcprize.org/about
- ARC-AGI-2 official page  
  https://arcprize.org/arc-agi/2
- ARC-AGI-3 Competition official page  
  https://arcprize.org/competitions/2026/arc-agi-3
