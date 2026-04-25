# Agent Workflow 与 Evals 落地指南

> 很多 agent 项目死在两个误解上。第一个误解是：只要 prompt 写得够长，系统自然会稳定。第二个误解是：demo 能跑通，就说明流程已经成立。真正可用的 agent workflow 不是“能跑”，而是“能被设计、观察、评估、回归、收敛”。

## 1. 从任务合同开始，而不是从 prompt 开始

写 agent 最危险的习惯，是一上来就写提示词。提示词当然重要，但它不应该是规格书。真正的规格书应该先回答：这个系统到底要完成什么任务、什么叫成功、什么叫不可接受的失败、哪些动作必须有人类批准。

在进入任何工具调用之前，先强行写下四个问题：

1. 用户要的最终产物是什么？
2. 哪些步骤必须基于外部真实反馈，而不能靠模型想象？
3. 哪些错误是可恢复的，哪些错误一旦发生就会造成真实损失？
4. 任务是否需要固定流程，还是需要模型在执行中动态决定路径？

Anthropic 在《Building Effective AI Agents》里反复强调：先从最简单的方案开始，只有当复杂性被证明有收益时，再引入更 agentic 的结构。OpenAI 的评估文档则提供了一个非常实用的架构谱系：单次调用、workflow、single-agent、multi-agent。把这两个视角合在一起，得到的结论很清楚：

- 任务边界清楚、步骤相对固定时，优先用 workflow；
- 只有在无法提前枚举步骤、且模型确实需要在环境反馈中自主决策时，才上 agent；
- multi-agent 必须由 eval 驱动，而不是由“听起来更高级”驱动。

## 2. 架构选择：不是越 agentic 越好，而是越合任务越好

### 2.1 单次调用适合“答案式任务”

如果任务主要是在已有上下文中做判断、分类、抽取或单次生成，单次模型调用往往已经足够。此时最该花时间的地方，是输入规范和输出约束，而不是 orchestration。

### 2.2 Workflow 适合“流程已知、步骤可验证”的任务

如果你能明确写出“输入归一化 -> 检索 -> 工具调用 -> 校验 -> 汇总”这样的链路，就不要急着把控制权交给模型。Anthropic 把这种结构称为 workflows，OpenAI 的 Agent Builder 也把 workflow 设计成一组可连接的 nodes，并强调每条连接都是 typed edge，也就是上下游之间有数据契约。这种“显式边”之所以重要，是因为一旦边界清楚，评估和排障都会简单很多。

### 2.3 Single-agent 适合“步骤数未知，但工具集可控”的任务

OpenAI 的 evaluation best practices 对 single-agent 的描述很准确：agent 处理的是开放任务，它拥有指令和一组工具，并动态决定该调用什么。这意味着它天然带来新的不确定性，尤其是在工具选择、参数抽取和流程收束上。

### 2.4 Multi-agent 只适合“复杂性已经压垮单 agent”的任务

OpenAI 的文档直接指出，是否使用 multi-agent 应由 eval 驱动；一上来就使用 multi-agent 会引入不必要复杂度，拖慢到生产的路径。换句话说，多智能体不是默认答案，它是单智能体在工具面、任务类型或角色冲突上已经明显失稳之后的结构性补丁。

## 3. 工具契约：agent 成败，常常不取决于 prompt，而取决于工具长什么样

Anthropic 在文章里讲了一个极具代表性的例子：他们做 SWE-bench agent 时，花在工具优化上的时间比花在总 prompt 上的时间还多。一个具体改动是，把相对路径工具改成只接受绝对路径，结果模型使用得“flawlessly”。这件事值得反复咀嚼，因为它揭示了 agent 工程最核心的现实：

模型并不生活在你的抽象设计里，它生活在你给它的接口里。

因此，工具设计必须像产品设计，而不是像给自己写个 helper function。至少要做到下面四件事。

### 3.1 输入必须结构化

工具的输入 schema 不只是类型约束，更是思考约束。模糊的参数名、重叠的工具边界、含糊的单位和格式要求，都会直接转化成 agent 的错误行为。

### 3.2 描述必须可操作

Anthropic 的 `Define tools` 文档专门给出了 “good tool description” 的例子，并明确指出好描述要讲清楚：这个工具做什么、什么时候该用、返回什么、参数到底是什么意思。工具描述写得像高质量 docstring，agent 的表现通常会立刻改善。

### 3.3 错误语义必须明确

一个工具如果失败了，agent 应该重试、换工具、请求更多信息，还是交给人？如果错误返回只是一个模糊字符串，agent 通常只能猜。

### 3.4 危险动作必须能被拦住

OpenAI 在 `Safety in building agents` 中强调，对 MCP tools 要开启 approvals，让用户能逐次确认读写操作。不要把“模型会谨慎”当安全策略；安全策略必须体现在工具和审批边界里。

## 4. 工作流最小骨架：不是为了好看，而是为了可诊断

一个可维护的 agent workflow，至少应该显式拥有以下阶段。

### 4.1 Input normalization

把用户请求、已有上下文、外部约束、默认参数整理成统一输入。很多后续错误，其实都源于这里没做干净，例如缺失关键字段、把不可信内容直接塞进高优先级消息、让多个来源的意图互相污染。

### 4.2 Planning 或 routing

不是所有任务都需要复杂 planning，但几乎所有非平凡任务都需要某种 routing。此处要决定：问题属于哪个子域、需要哪些工具、是否需要先问清楚再继续、是否需要 human approval。

### 4.3 Action execution

真正的工具调用、网页操作、代码编辑、数据库读写都发生在这里。系统最容易在这个阶段犯两种错：一是选错工具，二是选对工具但参数错了。

### 4.4 Verification

这是很多 demo 流程最缺的一步。没有 verification，agent 只是“做了动作”；有了 verification，它才开始“对结果负责”。在 coding agent 里，测试就是 verification；在 research agent 里，引用和来源核验就是 verification；在 computer use 里，回看截图和页面状态就是 verification。

### 4.5 Response synthesis / handoff

执行结果不应该原样抛给用户，而应该按任务合同进行解释、摘要、披露不确定性，必要时转入人类接管。

如果任务更长，还要显式补上：

- memory：什么该记，什么不该记；
- retries / recovery：什么错误允许重试，重试几次；
- human handoff：什么时刻必须停下来交给人。

## 5. 为什么 trace 是 agent 时代的基本盘

没有 trace，你只能知道系统“好像失败了”；有了 trace，你才知道它到底是理解错、路由错、工具错、参数错，还是验证缺失。OpenAI 在 `Trace grading` 中把 trace 定义为 agent 的 end-to-end record，包含决策、工具调用和推理步骤，并用 grader 给这些轨迹打结构化标签。

更重要的是，OpenAI 在 `Evaluate agent workflows` 里给出了一个非常实战的顺序：

1. 当你还在调行为时，先看 traces；
2. 当你已经知道什么是“好”时，再转向 datasets 与 repeatable eval runs；
3. 通过 traces 找 failure mode，再通过 evals 验证修复是否真的成立。

这套顺序背后的哲学非常值得采纳：先学会看病，再学会做体检。很多团队一开始就急着建一个“大而全”的 eval 集，结果连系统到底哪层在崩都看不明白。

## 6. Eval 不是锦上添花，而是 workflow 的第二骨架

OpenAI 的 eval best practices 给了一个关键提醒：随着系统从 single-turn 走向 workflows、single-agent、multi-agent，不确定性只会增加。因此 eval 必须围绕真实系统行为和业务目标设计，而不是停留在抽象 benchmark 崇拜上。

真正有用的 eval，至少要覆盖四层。

### 6.1 结果层

最终回答是否正确、任务是否完成、输出是否符合格式和业务要求。这一层最接近用户体验，但也是最容易掩盖过程风险的一层。

### 6.2 过程层

有没有调用正确工具、顺序是否合理、参数有没有抽错、关键步骤之后是否进行了验证、是否无谓地绕远路。对 agent 系统来说，这一层往往比最终结果更能解释失败。

### 6.3 系统层

延迟、成本、并发稳定性、工具失败恢复能力、重复执行的一致性。这一层决定系统能不能从 demo 变成服务。

### 6.4 安全层

OpenAI 的安全文档重点提到 prompt injection、private data leakage、MCP tool misuse 等风险。它们都应该进入 eval 设计，否则系统一旦接真实外部工具，就会以非常不优雅的方式教育你。

## 7. Grader 设计：不要把“通过/不通过”当成评估的全部

OpenAI 的 graders 文档有两条建议特别值得吸收。

第一，不要只做 pass/fail，而要尽量让分数是平滑的。文档明确写道，`smooth score` 比二元标签更有利于优化，因为它能让系统知道“改进了一点”也算改进。

第二，要防范 `reward hacking`。OpenAI 直接提醒，模型可能学会钻 grader 的漏洞，表现为在 model grader 上分数很高，但在人类专家评审里很差。对 agent 系统来说，这种风险更大，因为一旦 grader 只看结果而不看过程，模型就可能用奇怪甚至危险的路径“撞对答案”。

因此，grader 设计最好满足以下原则：

- 能给部分分，而不是只有 0/1；
- 尽量同时看结果和过程；
- 尽量用结构化标准，而不是“看起来不错”；
- 必须和人工评审定期校准；
- 对高风险任务，宁愿加 human review，也不要迷信全自动 judge。

## 8. 安全不是单独一章，而是每一层都要长出来

OpenAI 的 agent safety 文档写得很直接：prompt injection 是常见且危险的攻击；private data leakage 即使没有攻击者也可能发生；连接 MCP 工具时风险会进一步上升。它给出的建议不是抽象口号，而是非常具体的工程要求：

- 不要把不可信输入直接塞进 developer messages；
- 在节点之间尽量使用 structured outputs，减少自由文本把恶意指令一路带下去；
- 开启 tool approvals；
- 用 guardrails 清洗用户输入；
- 用 traces 和 evals 监控行为；
- 让不可信数据永远不要直接驱动敏感动作。

这几条建议真正厉害的地方，在于它们把安全问题翻译成了 workflow 设计问题。你不需要等“安全团队”来接管，workflow 自己就应该内生出这些边界。

## 9. 一个能落地的最小迭代节奏

如果你现在要把某个 agent 真正做起来，不建议一口气上全套平台。更可行的做法是：

### 第 1 周：写任务合同和工具合同

把“成功标准、必须验证的步骤、危险动作、输入输出 schema”写清楚。只做一两个最关键工具，先别铺太多接口。

### 第 2 周：做可回放的最小 workflow

先让系统在 20 到 50 个代表性任务上可重复运行，并把每次执行的关键节点、工具调用和中间结果记录下来。

### 第 3 周：基于 traces 找 failure modes

不要急着扩功能。先看失败主要集中在哪：理解、路由、工具参数、环境摩擦、验证缺失、还是安全边界。

### 第 4 周：把 failure modes 变成 eval

先做一小套高信号 eval，不追求“覆盖宇宙”，只覆盖当前最常见、最昂贵的失败模式。然后再继续扩工具和任务。

这是一个比“先做大而全 agent，再慢慢稳定”更慢、更笨，却更接近会成功的方法。

## 10. 这一页最后想留下的原则

workflow 设计的最高目标，不是让模型显得聪明，而是让系统显得可靠。可靠不是来自一个神奇 prompt，而是来自五件事一起成立：

1. 任务合同清楚；
2. 工具契约清楚；
3. 路径可以观察；
4. 评估可以回归；
5. 风险有真实边界。

只要这五件事成立，agent 才会越做越稳；否则，功能越多，只会让你越不知道系统究竟在干什么。

## 11. Worked Case：把一个 issue-to-patch coding agent 从任务合同推到 eval

如果要选一个最适合练习 agent workflow 的案例，coding agent 几乎是默认答案。不是因为它“最酷”，而是因为它同时具备 Anthropic 所强调的几项稀缺条件：问题边界相对清楚、环境反馈明确、自动化测试可以充当地面真值、人类 review 又能补上“测试没测到”的那一层判断。它既足够开放，能暴露 agent 的真实问题；又足够可验证，不会让整个系统陷入纯主观争论。

下面这个 worked case，不是为了给出唯一正确答案，而是为了示范一条更稳的思考路径。

### 11.1 任务合同：先决定系统对什么负责

假设你要做的不是“万能 coding agent”，而是一个更窄、更可评估的系统：给定一个真实 repo issue，让 agent 在受限环境里尝试修复，并把结果交给人类 reviewer。

先写任务合同，而不是先写 prompt。

| 合同字段 | 本案例的写法 | 为什么这样写 |
| --- | --- | --- |
| 用户任务 | 根据 issue 描述，在指定仓库内提交一个候选补丁，并说明修复思路、证据与残余风险。 | 目标不是“看起来会写代码”，而是“形成可审阅补丁”。 |
| 成功标准 | 相关测试通过；补丁与 issue 语义一致；变更范围可解释；输出包含证据链。 | 只看测试会漏掉错修与过修。 |
| 非目标 | 不追求自动 merge；不追求一次覆盖所有潜在重构；不替代人类 reviewer。 | 先压缩责任边界，避免系统目标膨胀。 |
| 外部地面真值 | 测试结果、静态检查、仓库文件内容、issue 与错误日志。 | agent 必须从环境拿反馈，而不是靠想象。 |
| 必须人工介入的点 | 执行危险命令、修改安全敏感文件、测试通过但补丁涉及架构性变更。 | OpenAI 的 agent safety 与 tool approvals 逻辑都指向同一件事：高风险动作要有人类边界。 |
| 不可接受失败 | 删除数据、跳过验证、读取无关秘密、为追求通过测试而做明显投机修复。 | 先定义什么绝不能发生，系统才可能真正可控。 |
| 停止条件 | 达到成功标准；或在固定轮数后仍无高置信候选补丁；或遇到需要人工判断的岔路。 | agent 不是永动机，必须有收束条件。 |

这个合同的关键，不在“写得完整”，而在它把系统从模糊愿望拉回可问责的边界。只要任务合同没有写清，后面的 trace、grader、benchmark 都会变成装饰。

### 11.2 工具合同：别给 agent 一个“万能 bash”，再怪它不稳

SWE-bench Verified 的价值之一，在于它逼你承认 coding agent 的得分并不只是模型得分，而是模型、工具面、脚手架、验证环的合成结果。Anthropic 在 2025 年关于 agent tools 的文章里进一步把这个问题说透了：工具是 deterministic systems 与 non-deterministic agents 之间的契约；名字、参数、返回信息、错误语义都会直接改变 agent 行为。

因此，这个案例里的工具最好故意收窄，而不是默认把整台机器全暴露给 agent：

| 工具 | 推荐约束 | 设计理由 |
| --- | --- | --- |
| `search_code` | 允许按符号、路径、关键字检索；返回命中的文件路径、片段与行号。 | 先让 agent 学会定位，再谈编辑。 |
| `read_file` | 支持明确路径与行段范围；禁止读取 secrets 目录。 | 限制上下文膨胀，也限制越权。 |
| `apply_patch` | 只接受结构化 diff；要求说明意图。 | 把编辑动作变成可审阅对象。 |
| `run_tests` | 支持 `targeted` 与 `full` 两档；必须返回失败测试名、错误摘要、耗时。 | 测试既是验证器，也是下一步规划的输入。 |
| `ask_human` | 用于 issue 含糊、设计取舍、危险动作审批。 | 把“不知道”翻译成 workflow 的显式节点。 |

真正的经验是：一个看起来“更强大”的工具面，往往会让 agent 更弱。你给得越多，它越可能在错误空间里浪费预算。先把工具面收成一个能稳定闭环的工作台，再决定是否扩权。

### 11.3 workflow 不是提示词链，而是一份可审计的因果图

这个 coding agent 的最小 workflow 可以写成下面这条链：

1. `Normalize issue`：抽取 issue 的目标、症状、可能影响的模块、已知约束。
2. `Plan first probe`：决定先搜哪里、先跑哪类测试、需要哪些文件。
3. `Search and inspect`：使用 `search_code` 与 `read_file` 建立最小问题模型。
4. `Edit candidate patch`：用 `apply_patch` 形成候选修复。
5. `Verify`：先跑 targeted tests，再决定是否扩展到更大验证范围。
6. `Decide next move`：通过、继续迭代、还是请求人工判断。
7. `Handoff`：输出补丁摘要、证据、未决风险与建议 reviewer 关注点。

注意这里最重要的不是“节点名字”，而是每个节点都能留下结构化痕迹。OpenAI 在 `Evaluate agent workflows` 与 `Trace grading` 里强调的正是这一点：当你还在调行为时，trace 是最快的诊断方式，因为它能把“模型到底错在哪一层”拆出来。

### 11.4 把 eval 拆成四层，而不是只问“测试过了没有”

按照 OpenAI 的 eval best practices，这个案例的评估流程至少要写出 objective、dataset、metrics、compare、continuous evaluation 五个部件。但真正让它有操作性的，是把指标拆到系统的不同层上。

#### 结果层

- 相关测试是否通过。
- 补丁是否真正回应 issue，而不是只让测试静默。
- reviewer 是否认为说明文字足够解释变更。

#### 过程层

- agent 是否先检索再编辑，还是直接盲改。
- issue 不清楚时，是否调用 `ask_human` 或继续采证。
- 失败后是否根据测试反馈修正假设，而不是机械重试。
- 是否进行了从 targeted tests 到更大范围验证的升级判断。

#### 系统层

- 每个样本平均执行轮数。
- 平均 wall-clock time。
- 平均测试调用次数与 token 成本。
- 重复运行的一致性。

#### 安全层

- 是否尝试读取无关敏感路径。
- 是否尝试执行未批准的危险命令。
- 是否把 issue 中的不可信文本直接变成 shell 动作。

这四层拆开后，你会得到一个非常重要的好处：即使“结果层”还没明显上升，你也能看见过程层和系统层是不是已经改善。OpenAI 的 grader 文档强调 `smooth score`，本质上就是要保留这种中间改进信号。

### 11.5 一个足够小、但足够真的首批 eval 集

很多团队做 eval 的第一反应是“先攒 1000 条”。这通常是错的。更好的起点是一个带偏见的小集合，但偏见要朝你最贵的失败模式倾斜。

这个案例的第一批 24 条样本，可以故意按失败类型分桶：

- `局部修复型`：单文件、小范围、测试失败明确。
- `跨文件依赖型`：至少两个文件联动，验证依赖调用链理解。
- `描述含糊型`：issue 语义不完整，逼 agent 选择继续采证还是请求人工判断。
- `假朋友型`：表面看像 A 模块问题，根因在 B 模块。
- `验证陷阱型`：targeted tests 能过，但 full test 暴露回归。
- `环境摩擦型`：测试耗时长、报错脏、日志嘈杂，考验 agent 是否会收缩搜索。

这个小集合的价值，不在统计意义，而在方法论价值。它会迫使你面对一个现实：eval 不是抽样宇宙，而是优先覆盖你现在最昂贵的认知漏洞。

### 11.6 trace review 应该怎么看，而不是只截图留念

对这个 coding agent，我会要求每条失败 trace 至少回答下面六个问题：

1. agent 第一次形成的工作假设是什么，它错在何处。
2. 第一轮工具选择是否合理。
3. 错误信息是否足以引导下一轮，还是工具返回本身就不利于纠错。
4. agent 有没有在错误轨道上高速前进。
5. 如果最后成功，它是“理解后成功”，还是“撞运气成功”。
6. 如果最后失败，人类要修它，先修 prompt、工具、workflow，还是直接换 benchmark / 换任务定义。

这个 review 框架的核心用途，是把 failure mode 从“感觉不稳”翻译成“可行动的改造位点”。

## 12. 一张可以直接套用的 eval worksheet

下面这张 worksheet 的目的，是逼你把抽象方法论压成一页纸。只要这页纸填不出来，workflow 还不该进入更大规模实现。

| 栏位 | 你必须写出的内容 | 常见偷懒方式 |
| --- | --- | --- |
| 任务对象 | 我到底在自动化什么，不自动化什么 | 写成“提升研发效率”这种空话 |
| 最终成功标准 | 用户真正关心什么结果 | 只写“测试通过” |
| 外部验证器 | 哪些反馈来自环境，而不是模型自评 | 把模型自信当验证 |
| 关键过程判据 | 哪些中间行为好坏会决定结果 | 完全不看中间过程 |
| 样本来源 | 生产日志、专家构造、历史失败案例各占多少 | 只收容易样本 |
| 高风险失败 | 哪些错误绝不能上线 | 把所有失败一视同仁 |
| 人工裁决点 | 哪些情况必须 handoff | 以为“更自动化”就更先进 |
| 连续评估策略 | 每次改 prompt、工具、路由后如何回归 | 调完手感好就上线 |

这张表的价值，不是文档完整性，而是它会暴露“你究竟是在做系统，还是在做 demo”。

## 13. 练习题：用方法，不要背概念

1. 你的 coding agent 在 20 个样本里有 14 个通过，但其中 5 个补丁虽然通过测试，却被 reviewer 判为“修错问题”。这主要暴露的是结果层、过程层还是 grader 设计问题？你会先改哪一层，为什么。
2. 某次改动后，agent 的最终通过率没变，但平均执行轮数下降 35%，无效检索减少，且 targeted tests 的命中率更高。按照 OpenAI 的 `smooth score` 思路，这算不算真实改进？为什么。
3. 如果一个样本的问题描述含糊，agent 先连续搜索 8 次才去问人，而人工本来在第 2 次搜索后就能判断“需要澄清”，这说明是 prompt 问题、工具问题，还是 handoff 边界问题。
4. 你发现 agent 在 model grader 上分数变高，但人工评审认为补丁更投机。这更像性能进步，还是 grader hacking？你打算如何验证。

## 来源

- Anthropic, *Building Effective AI Agents*  
  https://www.anthropic.com/engineering/building-effective-agents
- Anthropic, *Writing effective tools for AI agents — with agents*  
  https://www.anthropic.com/engineering/writing-tools-for-agents
- Anthropic, *Define tools*  
  https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools
- OpenAI, *Agent Builder*  
  https://developers.openai.com/api/docs/guides/agent-builder
- OpenAI, *Evaluate agent workflows*  
  https://developers.openai.com/api/docs/guides/agent-evals
- OpenAI, *Trace grading*  
  https://developers.openai.com/api/docs/guides/trace-grading
- OpenAI, *Evaluation best practices*  
  https://developers.openai.com/api/docs/guides/evaluation-best-practices
- OpenAI, *Graders*  
  https://developers.openai.com/api/docs/guides/graders
- OpenAI, *Safety in building agents*  
  https://developers.openai.com/api/docs/guides/agent-builder-safety
