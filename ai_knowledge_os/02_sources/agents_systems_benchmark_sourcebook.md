# Agents / Systems / Benchmark Sourcebook

写作时间：2026-04-22

这份 sourcebook 不是链接目录，而是一份“资料分工说明书”。它服务的不是收藏，而是判断：

- 你要做 research agent，先看什么。
- 你要读懂 computer-use、coding、benchmark 和 serving，哪些是一手主干，哪些只是补充。
- 你要避免被高分、热词和框架营销带偏，应该用什么材料校正自己。

AI 资料太多时，真正稀缺的不是链接，而是分工感。每一种来源都只擅长回答一类问题。把不擅长回答的问题强行压给它，学习就会变形。

## 一、先给这个专题定边界

这一组文档关注的是三条线：

1. `Agents`
   关心如何把模型变成可以观察、可以接工具、可以迭代的工作流系统。

2. `Systems`
   关心模型为何能在延迟、吞吐、成本、上下文和工具调用约束下真正跑起来。

3. `Benchmark`
   关心今天到底在测什么、什么还没有被测、以及分数与现实之间的距离。

这三条线在 2026 年已经不能分开学。因为真正的前沿变化，不再只是“模型又更聪明了”，而是：

- agent 开始从 prompt 技巧变成系统工程；
- inference 开始从幕后工程变成能力上限与商业可行性的共同决定因素；
- benchmark 开始从学术度量变成产品、融资、舆论和安全讨论里的坐标系。

## 二、Agent 工程主线：从“会调工具”到“能形成闭环”

### 1. OpenAI Agents Guide / Agent Builder / Evals / Trace Grading

这是一条非常典型的工程化资料链。它们分别回答不同问题：

| 来源 | 核心回答的问题 | 你应该从中拿走什么 | 常见误用 |
| --- | --- | --- | --- |
| Agents Guide | agent 的基本能力面是什么，tool、state、workflow、conversation 怎么组织 | agent 是一个可运行系统，而不是一个长 prompt | 只记住工具列表，不去理解 state、context management 和 orchestration |
| Agent Builder | 可视化 workflow 与 hosted 工作流如何被组织、版本化和发布 | 节点式 workflow 的抽象边界，以及“从原型到可维护系统”的过渡方式 | 把可视化 builder 当成“低代码玩具”，忽视它在结构化编排上的价值 |
| Evals | 怎么把主观上“好像能用”的 agent 变成可重复比较的系统 | 评估必须是持续性的，而不是上线前一次性检查 | 把 eval 当作最后补上的 QA，而不是开发流程中心 |
| Trace Grading | 怎么从运行轨迹中做诊断、打分和回归定位 | 不只评最终答案，更评中间行为质量 | 只看最终是否成功，忽略 agent 是如何失败的 |

#### 为什么这组资料值得连起来看

因为它们合在一起，实际上定义了一种新的默认工作方式：

过去很多人做 agent，做法是“先把工具接上，再试几个 prompt，看起来能跑就继续加功能”。这个流程的问题是，系统一旦变复杂，就会迅速失去可解释性与可维护性。

而这组资料在做的事情，是把 agent 工程从 demo 文化拉向闭环文化：

- 先明确工作流；
- 再明确工具和状态；
- 再明确 trace；
- 再明确 grader；
- 最后让修改可以回归验证。

这不是小升级，而是范式升级。它意味着 agent 不再只是 prompt engineering 的延伸，而是逐渐接近传统软件工程对可观测性、回归测试与版本演化的要求。

#### 这组资料最适合谁

- 想做 research agent、coding agent、workflow agent 的工程型学习者。
- 想理解“为什么很多 agent demo 不能稳定上线”的人。
- 想把 benchmark 分数与产品质量重新接上关系的人。

#### 最常见误用

最大误用是把这套文档只当“接 API 的手册”，而不把它理解成一种工程 discipline。真正重要的并不是某个接口怎么调，而是这套资料背后默认了一件事：agent 必须被测量，必须被拆解，必须能被持续改进。

### 2. MCP Architecture

MCP 文档的价值，不在于它又提供了一个“新工具”，而在于它给出了 agent 外部能力接入的一套协议化语言。

它最关键的两层划分是：

- `Data layer`
  用 JSON-RPC 语义定义 lifecycle、tools、resources、prompts、notifications 等核心原语。
- `Transport layer`
  用 stdio 或 streamable HTTP 处理连接、认证和消息交换。

#### 它到底解决什么问题

它解决的是长期困扰 agent 工程的老问题：每接一个外部系统都像写一次胶水层。  
MCP 不是让复杂性消失，而是把复杂性搬到一个更可组合的层面。

这对学习路径的意义非常大。它提醒你：

- 工具调用不是“模型突然会做事了”，而是协议设计与宿主编排一起作用的结果。
- 真正的 agent 能力，不止取决于模型，还取决于 host、client、server 的分工是否清晰。
- 一旦协议层开始收敛，创业和工程差异化就会往更上层移动：工作流理解、领域数据、评估闭环、组织集成与 reliability。

#### 常见误用

最常见的误用是把 MCP 当成“生态热点”而不是“接口哲学”。如果你只是知道 host/client/server 这些词，却没有意识到它们在重划责任边界，那你学到的只是名词。

## 三、系统主线：从“模型会回答”到“系统能交付”

### 1. Stanford CS336

如果你想理解 agent 和 benchmark 为什么会在现实里卡住，CS336 是非常关键的底层来源。它把 tokenizer、Transformer、数据、训练、推理、系统优化、alignment 和 reasoning RL 连成了一条完整链路。

它对 systems 学习的意义在于：你不再会把推理性能、上下文长度、attention 优化、分布式训练、对齐策略看成彼此无关的专题，而会开始把它们视为一个统一系统中的不同压力面。

#### 适合解决的问题

- 为什么上下文、推理和效率经常互相掣肘。
- 为什么某些 benchmark 上的提升，背后其实是系统和推理时计算在起作用。
- 为什么 serving、训练和后训练不能在认知上被切碎。

#### 常见误用

很多人只把 CS336 当“理论课”看，这是对这门课最大的误读。它其实非常 implementation-heavy。你如果不认真面对它对 profiling、FlashAttention、parallelism、data processing 和 reasoning RL 的强调，就会错过它最有价值的部分。

### 2. Full Stack Deep Learning

FSDL 不是用来教你一个更聪明的模型，而是用来教你一个更完整的 AI 系统。

它的独特价值在于：它始终逼你把模型放在开发基础设施、实验管理、部署、监控、团队协作和产品生命周期里看。

#### 为什么它对 benchmark 学习也重要

因为 benchmark 本身就是一种“系统输出”。  
如果你不理解实验设计、数据管理、部署监控和组织反馈，你就会高估 benchmark 的纯技术含义，低估它作为工程 artifact 的一面。

换句话说，FSDL 会让你更容易看穿“漂亮分数”和“可持续能力”之间的差异。

### 3. vLLM

vLLM 是开源 serving 栈的重要参考点。它的文档体系揭示了一件事：今天的推理系统已经不是简单的“加载模型并生成 token”，而是在处理一个完整的资源调度与复用问题。

你应该重点从 vLLM 中理解这些主题：

- automatic prefix caching
- disaggregated prefill
- context/data/expert parallel
- OpenAI-compatible serving
- tool calling / structured outputs / reasoning outputs

#### 它适合回答什么问题

- 为什么很多表面上的“模型能力提升”其实依赖系统把成本和延迟压到了可接受区间。
- 为什么长上下文、工具调用和 multi-turn reasoning 会对 serving 结构提出新要求。
- 为什么 inference optimization 已经开始反向影响产品形态。

#### 最常见误用

最大误用是把 vLLM 当成“部署工具选型指南”而不是学习推理系统抽象的入口。  
如果你只记命令，不理解 prefix caching、调度与内存复用，你会知道怎么跑，但不知道为什么会慢、为什么会贵、为什么会不稳。

### 4. TensorRT-LLM

TensorRT-LLM 给的是另一种视角：当你站到 NVIDIA 深度优化栈里看推理，问题会被重新表述为更硬的性能语言：

- KV cache system
- paged attention
- overlap scheduler
- speculative decoding
- quantization
- parallelism
- benchmarking

#### 它为什么重要

因为它提醒你，能力竞争越来越像“算法 + 系统 + 供应链 + 硬件亲和性”的复合竞争。  
很多团队谈 AI 还停留在“哪个模型更强”，但 TensorRT-LLM 会逼你看到：在真正大规模使用中，硬件利用率、调度路径、内存策略和量化方案，都会进入能力与商业性的共同定义。

#### 常见误用

不要把 TensorRT-LLM 的峰值性能叙事直接等同于你的最优路线。它高度依赖 NVIDIA 栈，也意味着生态锁定、迁移成本和团队技能结构会变成你必须一起考虑的问题。

## 四、Benchmark 主线：你测到的，不等于你真正需要的

benchmark 的真正价值，不是“给模型排位”，而是让你学会分辨能力边界、环境差异和迁移难度。

### 1. WebArena-x 家族

WebArena-x 今天已经不只是单个 benchmark，而是一个 benchmark family。它至少向你传达了三件重要事情：

1. web agent 评测开始从单一网页任务，走向家族化、视觉化、连续化。
2. environment fidelity 和 reproducibility 被放到中心位置。
3. benchmark 本身也在演化，因为旧 benchmark 会逐渐被 agent 设计和数据分布“吃透”。

其中 WebArena 的原始价值在于，它让 web agent 进入一个高度可交互、可复现实验环境，任务强调长程、多页面、功能正确性，而不是单轮回答。

#### 它最适合回答的问题

- agent 是否具备网页操作与信息搜集的基本长程能力。
- observation 设计对 performance 的影响是什么。
- 功能正确性如何被程序化评估。

#### 它不适合替你回答的问题

- 企业真实 workflow 是否已经可自动化。
- agent 是否具备跨组织权限、隐含流程、模糊需求处理能力。
- 模型是否因此就“会工作”。

#### 常见误用

WebArena 高分很容易让人产生“office automation 已经近在眼前”的错觉。但 web benchmark 的长程性，不等于组织工作流的脏性。真实世界里，权限、例外、模糊目标、人机协作和后果责任，往往比网页导航本身更难。

### 2. OSWorld

OSWorld 的重要性在于，它把 agent 评测从网页推进到了“真实计算机环境”的层级：多操作系统、真实 web 与 desktop app、文件 I/O、跨应用工作流、可重复 setup 与 execution-based evaluation。

原始论文的基线结果很残酷：人类完成率超过 72%，最强模型约 12%。而 2026 AI Index 已经把 OSWorld 上的 agent 进展写进技术性能章节，说明这个 benchmark 已经被视为现实 computer-use 能力的重要信号源。

#### 它真正测到了什么

- GUI grounding
- 操作知识
- 长程状态维持
- 跨应用 workflow
- 多模态观察与行动的配合

#### 它真正没有测到什么

- 组织上下文和 tacit knowledge
- 高责任场景下的人类兜底机制
- 产品级的权限、审计、异常恢复流程

#### 为什么它对知识树很重要

因为 OSWorld 迫使你承认：computer use 不是“多模态模型 + 鼠标键盘 API”这么简单。它是视觉理解、交互语义、错误恢复、界面适应和任务分解共同作用的系统问题。

### 3. SWE-bench / Verified / Multimodal

SWE-bench 的地位很特殊。它一方面已经成为 coding agent 能力讨论的公共坐标，另一方面又特别容易被误读。

最重要的三个事实是：

1. 原始 SWE-bench 来自真实 GitHub issue 与 PR 对。
2. Verified 是 500 个经人工确认可解、描述清晰、测试可靠的子集。
3. 2026 年的相关分析论文已经显示，leaderboard 生态被大量工业系统和专有模型主导。

#### 它适合解决的问题

- coding agent 在真实 repo issue-resolution 上到底能否有效工作。
- 不同 agent scaffold、tool use、multi-rollout 策略的差异是否显著。
- 真实代码上下文与测试约束会怎样改变模型行为。

#### 它最容易被误用的方式

把 SWE-bench 高分直接翻译为“软件工程已被解决”。  
这会忽略至少四个关键事实：

- 任务分布主要来自特定 repo 与语言生态。
- 评测成功是执行与测试定义下的成功，不是组织生产环境中的全部成功。
- agent scaffold 差异巨大，不是单纯 base model 对决。
- 真实开发工作还有需求澄清、沟通、上线、回滚、协作和责任划分。

#### 作为资料源，应该优先看什么

优先顺序应该是：

1. 原始论文与 GitHub 组织
2. Verified 页面与 leaderboard 定义
3. 相关后续分析论文

这是因为到了 2026 年，SWE-bench 相关站点和生态已经变得很繁杂。你必须优先回到论文、GitHub 组织和官方 leaderboard 定义，避免把周边产品页、营销页或第三方总结误认成 benchmark 本体。

### 4. RE-Bench、HCAST 与 METR Time Horizons

这是 2025-2026 年最值得重视的一组能力度量来源。它们不再只问“模型会不会某种题型”，而是开始问更难也更接近现实的问题：

- 一个 agent 能在多长的人类任务时间尺度上稳定工作。
- 它在 research engineering、software、cybersecurity 等复杂任务里和人相比是什么量级。
- benchmark 是否能够直接连接到现实里我们真正关心的影响。

#### RE-Bench 的独特价值

RE-Bench 专门盯的是 frontier AI R&D 能力。它用 7 个开放式 ML research engineering 环境和人类专家对比，发现 agent 在短预算下可能非常快、非常能试，但人类在时间预算变长后仍有更强回报。

这个结果非常重要，因为它击中了当前 agent 进展的核心特征：

- 它们在局部搜索、快速试错、低成本并行上越来越强；
- 但在人类长期依赖的高上下文、跨阶段积累和开放式判断上，优势还不稳定。

#### HCAST 的独特价值

HCAST 明确把 benchmark 和 grounding 绑定起来。它不只测 task success，还用人类完成时间来校准任务难度，追问“这个 agent 能不能被信任去做一个要花人类 X 小时的任务”。

这是一种非常重要的 benchmark 观：不是把分数抽象化，而是把它重新翻译回人类世界中的时间、难度和信任阈值。

#### METR Time Horizons 的独特价值

Time Horizon 系列把“任务长度”变成能力度量，并公开讨论外部效度、任务分布、messiness 和可靠性层级。它给你的不是一个神奇预测，而是一种更诚实的测量语言：

- frontier 模型在低上下文、可评分的软件类任务上，自主时程正在变长；
- 但这种增长并不等价于“所有工作都会同速被自动化”；
- benchmark 的任务分布、评分方式和上下文条件，会强烈影响你对进展的解释。

这组资料最值得学习的地方，不只是结论，而是它们对“测量什么才算有意义”这个问题的认真程度。

### 5. ARC Prize 与 AIMO

这两类来源代表的是另一种 frontier：它们不是在测 workflow 完成度，而是在逼问 generalization、reasoning、search、test-time compute 与 problem-solving style 的边界。

#### ARC 的价值

从 Chollet 的《On the Measure of Intelligence》到 ARC Prize 2024 Technical Report，这条线一直在强调一个很不舒服、但非常有启发性的事实：

单纯测 skill 很容易被 priors、训练数据和经验“买出来”；如果你真正关心 intelligence，就必须关心新任务上的 skill-acquisition efficiency。

ARC 因此重要，不是因为它给了世界一个终局 benchmark，而是因为它不断提醒你：  
“会做训练中见过的问题”与“能快速学会没见过的问题”不是同一回事。

#### AIMO 的价值

AIMO 把数学竞赛推理、难度升级和算力资源结合起来。AIMO3 在 2025 年 11 月启动时提高了难度和 compute 资源，这意味着它不只是数学题竞赛，也是 reasoning 与 test-time compute 经济学的一面镜子。

#### 这类来源最容易被误用的方式

把 reasoning 竞赛成绩直接翻译成产品成熟度。  
这类 benchmark 更适合告诉你“能力上限是否在移动”“哪类 search / training / verification 思路有效”，而不是告诉你“某个产品明天就会替代某类团队”。

## 五、把这些来源连起来：它们分别适合回答什么问题

### 问题 1：我想做一个 research / workflow agent，从哪里开始

先读：

1. OpenAI Agents Guide
2. Evals / Trace Grading
3. MCP Architecture
4. WebArena / OSWorld / HCAST 中最接近目标环境的一项

原因是你首先要掌握工作流、协议和评估，而不是先迷失在框架细节里。

### 问题 2：我想搞懂为什么某些 agent 看起来强，但一上线就脆

先读：

1. FSDL
2. Evals / Trace Grading
3. HCAST / METR Time Horizons

因为上线脆弱通常不是单点能力问题，而是系统化评估、可观测性和任务分布理解不足。

### 问题 3：我想理解推理系统为什么会改写产品路线

先读：

1. CS336 的系统部分
2. vLLM
3. TensorRT-LLM
4. AI Index 2026 的硬件与经济章节

这样你会把 latency、throughput、成本、context 和 hardware 看成一张图，而不是五个分散术语。

### 问题 4：我想判断 benchmark 分数到底值不值得信

先读：

1. 原始 benchmark 论文
2. benchmark 官方站点的任务、环境、评分定义
3. HCAST / METR 这种强调 grounding 的工作
4. “What’s in a Benchmark? The Case of SWE-Bench...” 这类二阶分析

因为真正的判断来自 construct validity、任务分布、评分方式、agent setup 与外部效度，而不是来自朋友圈截图。

## 一个常见误判案例：按热度读资料，最后会把系统问题读成模型问题

这是这一组资料最容易发生的学习事故。

一个人想理解为什么 coding agent 最近这么热，于是他的阅读顺序通常会变成：

1. 先看 leaderboard 新闻或社交媒体战报。
2. 再看某个 agent 框架或产品博客。
3. 最后看几篇“某某模型更强了”的总结。

这个顺序有很强的即时满足，因为它最先给你结果感。但它的问题是，你会在一开始就把真正决定结果的多层因素揉成一个模糊印象，最后只记住一句话：`模型更强了，所以 agent 更强了。`

这恰好是最危险的误读。因为在 2026 年，很多重要结果其实是这些因素共同作用的产物：

- model base capability
- post-training
- scaffold design
- tool protocol
- environment harness
- rollout budget
- verifier / grader
- serving efficiency

如果你的阅读顺序跳过了这些层，最后就会把系统工程、benchmark 设计和推理预算的贡献，误读成“模型魔法”。这不仅会让你学错知识，也会直接带偏项目判断。你会高估替换一个模型所能带来的改善，低估 workflow、eval、protocol 和 infra 对结果的决定性作用。

更稳的阅读顺序，应该反过来：

1. 先读 benchmark 或任务定义，弄清楚到底在测什么。
2. 再读 agent workflow 与 eval 文档，弄清楚系统是怎样被观察和改进的。
3. 再读 serving / systems 文档，弄清楚能力为什么能在成本与延迟约束下成立。
4. 最后才回来看模型和后训练，在整个系统里它到底贡献了什么。

这不是故意把学习变慢，而是为了防止你把最亮的一层误当成最深的一层。

## 读这一组资料时，应该持续追问的研究问题

sourcebook 的意义，不是替你把链接存起来，而是给你一套读法。下面这些问题，适合作为你读完任何一类来源后都要留下来的“分析残余”。

### 问题 1：这份资料真正把哪个变量固定住了

读 benchmark 时要问：它固定了哪些环境、任务和预算。  
读 systems 文档时要问：它默认了什么硬件和吞吐目标。  
读官方 guide 时要问：它为了让你上手更快，隐去了哪些工程代价。

如果你不先看清固定变量，几乎一定会在迁移时犯错。

### 问题 2：它解决的是“能力问题”，还是“可交付问题”

很多人把这两者混在一起。模型会不会做，是能力问题；能不能稳定、便宜、可观察地做，是可交付问题。前沿真正有价值的变化，通常同时影响两者，但比例不同。把这两类问题分开，能显著提升你的判断清晰度。

### 问题 3：这份资料对我的项目路线意味着什么动作

好资料应该逼出动作，而不只是增加敬畏感。每读完一份一手来源，至少应该回答：

- 我要补哪一个概念节点。
- 我要删掉哪一个原先的误判。
- 我要新增哪一个实验或评估。
- 我要把哪一个主题从“知道名词”升级到“必须主修”。

### 问题 4：它最可能被怎样误用

这是防止知识被流行叙事收编的关键问题。  
benchmark 最容易被误用成营销坐标。  
systems 文档最容易被误用成选型广告。  
官方 guide 最容易被误用成“能力已经稳定”的暗示。  
报告最容易被误用成线性预测机器。

如果你能先写出一份资料最可能被怎样误用，你才算真正开始掌握它。

## 六、这份 sourcebook 最重要的结论

### 1. Agent 不再只是模型问题

Agent 现在越来越像“模型 + 协议 + workflow + 评估 + serving”的复合系统。任何只盯单一层的学习都会失真。

### 2. 系统不再只是幕后工程

inference、cache、scheduling、parallelism 和 hardware economics 正在进入能力定义本身。  
理解 systems，不再只是为了省钱，而是为了理解哪些能力为什么能被释放出来。

### 3. Benchmark 不再只是学术装饰

它正在影响产品路线、创业叙事、市场预期和安全讨论。  
但它之所以重要，不是因为它能给出终局答案，而是因为它是今天少数能让讨论落地的共同坐标。

### 4. 真正高价值的资料，不是最热的，而是最能帮助你重新分层的

一份好资料的价值，不在于它让你多知道一个术语，而在于它能不能改变你看问题的层级。  
能改变层级的资料，才配进入这份 sourcebook。

## 来源列表

- OpenAI Agents Guide: https://developers.openai.com/api/docs/guides/agents
- OpenAI Agent Builder: https://platform.openai.com/docs/guides/agent-builder
- OpenAI Evals Guide: https://developers.openai.com/api/docs/guides/evals
- OpenAI Evaluation Best Practices: https://developers.openai.com/api/docs/guides/evaluation-best-practices
- OpenAI Trace Grading: https://platform.openai.com/docs/guides/trace-grading
- Model Context Protocol: https://modelcontextprotocol.io/
- MCP Architecture Overview: https://modelcontextprotocol.io/docs/learn/architecture
- Stanford CS336: https://cs336.stanford.edu/
- Full Stack Deep Learning: https://fullstackdeeplearning.com/
- vLLM docs: https://docs.vllm.ai/en/latest/
- vLLM Prefix Caching: https://docs.vllm.ai/en/latest/design/prefix_caching/
- TensorRT-LLM Overview: https://nvidia.github.io/TensorRT-LLM/overview.html
- TensorRT-LLM Performance Tuning Guide: https://nvidia.github.io/TensorRT-LLM/performance/performance-tuning-guide/index.html
- WebArena-x: https://webarena.dev/
- WebArena: https://webarena.dev/og/
- WebArena paper: https://arxiv.org/abs/2307.13854
- OSWorld: https://os-world.github.io/
- OSWorld paper: https://arxiv.org/abs/2404.07972
- SWE-bench GitHub: https://github.com/SWE-bench/SWE-bench
- SWE-bench Verified: https://www.swebench.com/verified.html
- SWE-bench paper: https://arxiv.org/abs/2310.06770
- What’s in a Benchmark? The Case of SWE-Bench in Automated Program Repair: https://arxiv.org/abs/2602.04449
- RE-Bench: https://arxiv.org/abs/2411.15114
- HCAST: https://arxiv.org/abs/2503.17354
- METR Time Horizon 1.1: https://metr.org/blog/2026-1-29-time-horizon-1-1/
- METR Time Horizons: https://metr.org/time-horizons/
- Measuring AI Ability to Complete Long Software Tasks: https://arxiv.org/abs/2503.14499
- ARC Prize Foundation: https://arcprize.org/about
- ARC Prize 2024 Technical Report: https://arxiv.org/abs/2412.04604
- On the Measure of Intelligence: https://arxiv.org/abs/1911.01547
- AIMO Prize: https://aimoprize.com/participate
- Stanford AI Index 2026: https://hai.stanford.edu/ai-index/2026-ai-index-report
