# 官方学习栈：用一手源搭出 AI 的主干，而不是堆资料

写作时间：2026-04-22

这页不是“AI 资料大合集”，而是一份主干学习栈。它要解决的不是“还能再看什么”，而是更难也更重要的问题：

1. 先看什么，后看什么，为什么。
2. 哪些官方资料是在教原理，哪些是在教接口，哪些是在校正现实感。
3. 哪些资料看上去很权威，其实很容易被误用。

一个成熟的 AI 学习者，不是把资料越攒越多，而是越来越知道每一类资料该在什么位置上发挥作用。课程负责搭骨架，开发者文档负责落地接口，benchmark 与竞赛负责告诉你真实难点在哪，年度报告负责把技术变化放回产业、治理与社会约束中重新理解。

如果把知识库比作一座湖，那么这页就是引水系统。它决定水从哪里来，经过哪些渠道净化，最终流进哪些支流与实验路线。

## 一、先建立四种“资料角色感”

在进入具体来源之前，先把四类一手源分清楚：

### 1. 课程主页与官方课程

它们解决的是“知识树长什么样”。好的课程不会告诉你今天哪家模型又刷新了哪个榜，而是告诉你一个主题内部的因果关系、依赖关系与递进顺序。

如果你没有课程骨架，就很容易出现两种假进步：

- 你知道很多术语，但不知道它们在系统里各自解决什么瓶颈。
- 你会讨论前沿热点，但不知道热点为什么重要，或者为什么其实不重要。

### 2. 官方开发文档与协议文档

它们解决的是“怎么把能力接进真实系统”。这类资料最擅长回答接口、工作流、限制条件、部署方式和评估闭环。

它们不负责替你形成第一性原理，所以最常见的误用是把文档当课程读，结果学会了接口，却没有形成方法论。

### 3. benchmark 官方站点与论文原文

它们解决的是“今天到底在测什么、没测什么，以及还卡在哪”。这类资料是现实感校准器。

它们最容易被误用的方式是被当成胜负榜。benchmark 的真正价值不是给模型排座次，而是暴露任务结构、误差结构、环境约束和可迁移性的边界。

### 4. 官方报告与机构主页

它们解决的是“技术变化怎样进入经济、组织、风险与政策世界”。这类资料给的是外部约束。

如果只读技术资料，不读报告，你会高估能力增长的单向性；如果只读报告，不读技术资料，你会高估宏大叙事的解释力。

真正稳的学习路径，是四类资料互相校正。

## 二、总学习原则：先骨架，后接口；先问题，后工具；先测量，后判断

这套学习栈遵循三条原则：

### 1. 先骨架，后接口

先用课程与官方报告建立世界观，再读具体 SDK、API、协议和 serving 文档。否则你很容易把“会调参数”误认为“懂系统”。

### 2. 先问题，后工具

不要从“最近什么框架火”开始，而要从“我要解决什么问题”开始。比如：

- 我要理解 LLM 从数据到训练到部署的完整链条。
- 我要做 research agent，核心难题是工具接入、状态管理，还是评估回归。
- 我要做推理服务优化，瓶颈到底在 KV cache、batching、并行策略，还是成本结构。

### 3. 先测量，后判断

很多看起来像“认知升级”的讨论，本质上只是叙事升级。没有 benchmark、human baseline、eval 闭环与系统指标，讨论很容易滑向立场、口号和情绪。

## 三、第一层：先读世界，再读模型

这一层的目标不是学会实现，而是先建立“AI 当前在世界中的位置感”。没有这层，后续所有技术学习都容易变成局部过拟合。

| 来源 | 适合解决的问题 | 在学习路径中的位置 | 常见误用 |
| --- | --- | --- | --- |
| Stanford AI Index 2026 | 能力、采用、投资、硬件、教育、治理到底在发生什么变化 | 每季度回看一次，用来校正你的知识树主干 | 把它当“新闻年鉴”看，却不把其中的结构性变化映射回自己的学习路线 |
| International AI Safety Report 2026 | 哪些能力上升与哪些风险上升是绑定的；哪些证据已经足够强 | 作为技术判断的外部约束层，尤其适合和 benchmark、agent、autonomy 线并读 | 把“安全”理解成独立主题，而不是理解为能力、部署与治理的耦合问题 |

### 为什么先读这两份报告

因为它们会强迫你承认三件事：

第一，能力提升不是抽象的。它会具体体现在 coding、computer use、science、math、economy 和 labor 的不同维度上，而且这些维度进展并不齐平。

第二，进步不是平滑的。2026 AI Index 明确强调能力前沿呈现 jagged frontier：模型可能在一些困难任务上逼近或超过人类，却在看似简单的任务上持续犯错。这个事实直接决定了你不能用单一分数、单一 demo 或单一故事来理解 AI。

第三，风险管理不是能力之后的附录。International AI Safety Report 2026 反复强调的一点是：能力变强、应用扩散、风险证据积累，与评估和治理能力的成熟速度并不同步。也就是说，真实世界不是“模型变强了，所以一切顺理成章”，而是“模型变强了，于是更多约束暴露出来”。

### 这一层应该怎么读

- 先读 AI Index 的总 takeaways，再只挑与你当前主线最相关的章节。
- 再读 International AI Safety Report 的 executive summary 或 extended summary，把其中的能力判断和风险判断对照起来读。
- 读完以后，不要只做摘录，要强迫自己回答一句话：这份报告要求我把知识树上的哪一枝升格为主干。

## 四、第二层：模型与系统的主干课程

这一层决定你是否真的理解“模型是怎么长出来的”，以及“系统为什么会在工程上变成那样”。

| 来源 | 适合解决的问题 | 在学习路径中的位置 | 常见误用 |
| --- | --- | --- | --- |
| Stanford CS336: Language Modeling from Scratch | 语言模型从 tokenizer、架构、训练、数据、对齐到推理的完整因果链条是什么 | LLM 主干课程，适合作为模型世界观的中轴 | 只看 lecture 标题，不做 assignment，从而高估自己对实现细节和系统瓶颈的理解 |
| Full Stack Deep Learning | 模型如何走向实验、部署、监控、团队协作与产品化 | 把“模型能力”桥接到“真实产品系统” | 把它当成“工程杂项课”，错过它真正的价值：把技术放入组织与产品生命周期理解 |

### 1. Stanford CS336 的位置

CS336 的价值不只是“斯坦福课程”四个字，而在于它明确把语言模型当作一个可以从头实现、度量、优化、扩展和对齐的系统来教。2026 春季课程的 assignment 已经非常说明问题：

- 从 tokenizer、Transformer、optimizer 自己实现起步。
- 进入 profiling、benchmarking、FlashAttention/Triton、distributed training。
- 再到 scaling law、数据处理、alignment、reasoning RL。

这等于把今天大模型工作的核心链条公开摆了出来。对学习者来说，CS336 最重要的不是“跟着做完作业”，而是你会第一次真正看到：模型能力、数据质量、系统效率和对齐方法不是四条平行线，而是一条连续的工程链。

### 2. FSDL 的位置

如果说 CS336 在回答“模型是怎么造出来的”，FSDL 回答的是“模型怎么活下去”。它把开发基础设施、实验管理、数据管理、部署、监控、团队协作这些常被忽视的部分重新拉回主舞台。

很多人学 AI 时会犯一个非常典型的错误：他们误以为“模型”就是主角，而系统、组织和产品只是配角。FSDL 逼你接受相反的事实：当模型开始进入真实使用场景，真正决定成败的常常不是模型是否更聪明 2%，而是系统是否更稳定、迭代是否更快、指标是否更清楚、团队是否能把失败变成反馈。

## 五、第三层：开发者主线，理解今天的 agent 与应用是如何被搭起来的

这一层适合在你已经有模型骨架之后进入。它告诉你今天前沿应用是如何被拼接成工作流的。

| 来源 | 适合解决的问题 | 在学习路径中的位置 | 常见误用 |
| --- | --- | --- | --- |
| OpenAI Developer Docs | 当前主流 API、tool、agent、eval、optimization 能力如何组合 | 进入实际构建阶段后的总接口地图 | 把文档当理论教材，或者只学调用不学评估 |
| OpenAI Agents SDK / Agent Builder / Evals / Trace Grading | agent 如何被编排、观察、评估、回归改进 | 作为 agent 工程的闭环文档 | 只看“会不会调工具”，忽略 trace、grader 与回归体系 |
| Model Context Protocol Architecture | 外部工具、资源、prompt 如何通过统一协议接入 | 进入工具生态与可组合工作流的协议层 | 把 MCP 当作新概念热点，而不去理解 host/client/server 和 data/transport 两层分工 |
| Hugging Face Learn | 官方课程聚合入口，覆盖 LLM、Agents、MCP、Diffusion、Audio 等 | 作为课程化补充与 hands-on 并行入口 | 只做交互式教程，不回到原始论文、协议和系统文档 |
| Hugging Face Agents Course | 对 agent fundamentals、frameworks、benchmark-based hands-on 的系统化训练 | 适合作为 agent 的实践支线 | 把它误当成 agent 的全部，而忽略生产环境中的评估、成本与可靠性问题 |

### 1. 为什么要把 OpenAI 文档和 MCP 文档一起读

因为前者告诉你“今天如何把 agent 系统搭起来”，后者告诉你“这个系统为什么开始走向协议化”。

Model Context Protocol 的关键价值不在于又多了一个名词，而在于它把 tools、resources、prompts、notifications、lifecycle 和 transport 清晰拆开。你一旦真正理解这一点，就不会再把工具调用理解成一堆 ad-hoc glue code，而会开始把它看成协议层、能力层和应用层之间的分工问题。

这会直接改变你的工程审美：你会不再执着于“给每个应用写一套独立接入逻辑”，而会开始思考“哪些外部能力应该被协议化、标准化、可替换化”。

### 2. 为什么 agent 学习不能只学框架

Hugging Face Agents Course 很适合入门和 hands-on，因为它把 think-act-observe、tools、frameworks 和 benchmark-based final project 串了起来。但如果你只学框架，很快会卡住。

你会发现，真实 agent 系统里的难题通常不是“会不会写 agent loop”，而是：

- 怎么定义任务边界与成功标准。
- 怎么设计 observation 和 state。
- 怎么接入工具又不把系统弄得脆弱。
- 怎么做 trace、grader 与回归。
- 怎么把 benchmark 得分和真实业务价值区分开。

所以，正确读法不是“课程替代文档”，而是“课程用来快速掌握形状，文档用来理解接口和限制，benchmark 与 eval 用来校准现实”。

## 六、第四层：推理系统与 serving，理解为什么能力增长最后会碰到工程与经济的墙

这一层决定你会不会把 AI 看成真正的计算系统，而不是只会把模型当黑盒。

| 来源 | 适合解决的问题 | 在学习路径中的位置 | 常见误用 |
| --- | --- | --- | --- |
| vLLM docs | 开源 serving 栈如何处理 prefix caching、tool calling、disaggregated prefill、parallel deployment | 开源通用推理系统的主入口 | 只抄 deployment 配方，不理解 latency、throughput、memory 是如何交换的 |
| TensorRT-LLM docs | NVIDIA 栈如何做高性能推理优化、KV cache、paged attention、speculative decoding 与 benchmarking | 面向 NVIDIA 深度优化的高性能推理路线 | 只盯峰值吞吐，不看硬件依赖、生态锁定与迁移成本 |

### 1. 为什么这层在 2026 年已经不是可选项

2026 AI Index 直接把数据中心、芯片制造集中度与能源消耗拉到主舞台，意味着一个很现实的问题已经无法回避：能力不是只由“模型有多聪明”决定，也由“推理有多贵、延迟有多高、服务能撑多大流量”决定。

换句话说，今天的 AI 能力边界越来越像一个经济系统，而不只是一个算法系统。

### 2. 读 vLLM 和 TensorRT-LLM 的正确姿势

这两套文档不要当成“我暂时不做 infra，所以以后再读”。它们其实回答的是两个很核心的判断问题：

- 哪些能力提升来自模型本身，哪些来自 serving 与 inference engineering。
- 当 benchmark、reasoning 和 agent 都开始依赖更长上下文、更复杂工具、更高 test-time compute 时，成本与时延会怎样反过来塑造产品路线。

vLLM 适合用来建立开源 serving 的通用视角：prefix caching、OpenAI-compatible server、context parallel、tool calling、structured outputs、reasoning outputs。它让你看到今天开源推理栈已经不是“把模型跑起来”这么简单，而是一个围绕内存复用、请求调度、并行策略和生态兼容不断进化的系统。

TensorRT-LLM 则更像性能工程的极限训练。它把高性能推理拆成更接近硬件和调度的语言：KV cache system、paged attention、overlap scheduler、speculative decoding、quantization、parallelism。它会逼你意识到，很多能力看上去像“模型突然更强了”，其实背后是系统把原本太贵、太慢、太不稳定的推理路径变得可用。

## 七、第五层：benchmark 与竞赛，负责把你的幻想拉回现实

这一层是很多人最容易误读的一层。它不负责给你一个“谁最强”的永久答案，而是负责不断提醒你：什么是正在被测量的，什么仍然没有被测量。

| 来源 | 适合解决的问题 | 在学习路径中的位置 | 常见误用 |
| --- | --- | --- | --- |
| WebArena-x / WebArena / VisualWebArena / WebArena-Infinity | 网页环境中的 agent 是否真的能完成长程任务、视觉任务与连续评测 | web agent 主线 benchmark | 把网页 benchmark 高分直接当成真实企业 workflow 成熟 |
| OSWorld | multimodal computer-use agent 是否具备 GUI grounding、跨应用操作与长程执行能力 | computer-use 主线 benchmark | 忽略其环境仍然是可复现实验环境，而不是组织流程全貌 |
| SWE-bench / Verified / Multimodal | coding agent 在真实 repo issue-resolution 上是否有效 | coding agent 主线 benchmark | 把它当成“软件工程整体自动化”指标，忽略 repo、语言、scaffold 与评测设置限制 |
| RE-Bench / HCAST / METR Time Horizons | agent 的长时程自主性、R&D 能力与 human-calibrated performance 到底到哪了 | autonomy 与能力趋势判断的关键层 | 把“低上下文、可评分任务”的结果直接外推到所有工作 |
| ARC Prize / AIMO | generalization、数学推理、test-time compute 与 search 是否在改写能力上限 | reasoning frontier 的前沿信号层 | 把竞赛成绩直接等同于真实产品价值 |

### 1. 为什么 benchmark 是学习栈的一部分，而不是附录

因为课程与文档经常会让你以为问题已经被讲清楚了，但 benchmark 会把“讲清楚”和“做出来”之间的距离暴露出来。

WebArena 让你看到长程网页任务的难度不在于单步操作，而在于跨页面状态、信息搜集、行动顺序与功能正确性。

OSWorld 让你看到 computer use 的难点不只是“能看图”，而是 GUI grounding、UI layout、操作知识和跨应用工作流。

SWE-bench 让你看到 coding agent 的难点不只是生成补丁，而是理解 issue、定位上下文、做出可通过测试的修改。

RE-Bench、HCAST 和 METR 则把问题推进了一步：不仅测“能不能做”，还开始问“能做多久、能做多复杂、和人相比到底在什么量级上”。

### 2. 读 benchmark 时必须警惕的误区

- 不要把 leaderboard 当结论。leaderboard 只能告诉你在特定设定下谁做得更好，不能告诉你为什么更好、能迁移到哪、真实代价有多大。
- 不要把单一 benchmark 的高分当成通用能力。AI Index 2026 已经给出非常清楚的提醒：能力前沿是 jagged 的。
- 不要把 benchmark 当产品 KPI。public benchmark 适合做外部比较，但真正做产品时，最终一定要回到内部 eval、真实 workflow、失败样本与成本约束。

## 八、给不同目标的阅读顺序建议

### 路线 A：你想建立长期扎实的 AI 主干

1. AI Index 2026
2. International AI Safety Report 2026
3. Stanford CS336
4. Full Stack Deep Learning
5. Hugging Face Learn 中对应课程
6. 再进入具体 benchmark、agent 与 serving 文档

这条路线的核心是先建立“世界观”和“模型观”，再进入工程接口。

### 路线 B：你想快速做出高质量的 agent / workflow 系统

1. OpenAI Developer Docs 中的 Agents、Evals、Trace Grading、Optimization
2. MCP Architecture
3. Hugging Face Agents Course
4. WebArena / OSWorld / SWE-bench / HCAST 中与你目标最接近的 benchmark
5. 回头补 CS336 和 FSDL

这条路线的核心是先获得工作流能力，但必须尽快补理论和 benchmark literacy，否则很容易进入“demo 很聪明，系统很脆弱”的阶段。

### 路线 C：你想做系统、infra、serving 与 cost optimization

1. CS336 中系统相关部分
2. FSDL 的部署与监控部分
3. vLLM docs
4. TensorRT-LLM docs
5. AI Index 2026 中硬件、数据中心与经济章节

这条路线的关键是把性能优化理解成能力经济学，而不是单纯的工程手艺。

## 九、最常见的三种学习误用

### 1. 只读课程，不读文档

结果是概念很漂亮，系统一落地就手忙脚乱。你会知道 agent 的一般定义，却不知道 tool、transport、trace、grader、cache、rate limit 和 deployment 是如何在现实里约束它的。

### 2. 只读文档，不读 benchmark

结果是你知道怎么调用能力，却不知道这类能力在真实任务上的边界。你会写出很多“看起来有能力”的系统，但不一定知道它们在哪些任务分布上会系统性失败。

### 3. 只盯 benchmark，不读报告与系统资料

结果是你会过度崇拜分数。你会把短期 leaderboard 变化误认为产业格局变化，却看不见硬件供给、推理成本、部署约束、风险管理和组织采纳对路线的反向塑形。

## 十、这套学习栈真正想培养什么能力

不是“知道更多”，而是形成三种更难的能力：

第一，能区分什么是主干问题，什么是枝叶问题。  
第二，能把一个来源放回它该在的位置上理解，而不是被它牵着走。  
第三，能在能力、工程、评估、经济与治理之间来回切换视角。

AI 学习最怕的不是信息少，而是信息没有层级。没有层级，知识就会像碎石；有了层级，知识才会开始像地形。

## 十一、怎样真正读一手源：不是摘录，而是重建

一手源最难的地方，常常不是术语，而是它不会主动替你整理层级。论文、课程、官方文档、benchmark 站点，往往都默认你已经知道自己在找什么。于是很多人虽然读的是一手源，最后做出来的却仍然是二手笔记：把别人说过的话按顺序记下来，却没有重建出它的内部因果。

更有效的读法，通常至少要过四遍。

### 第一遍：先定位“这份材料在回答什么旧问题”

不要一上来就看结论，更不要一上来就抄术语。第一遍最该问的是：在这份材料出现之前，人们卡在什么地方？

例如：

- `CS336` 回答的不是“如何学会一点 LLM 知识”，而是“怎样把语言模型当成一个从 tokenizer 到 alignment 都可拆解、可实现、可度量的系统来理解”。
- `OpenAI Evals` 回答的不是“如何写一个评测脚本”，而是“如果不先有稳定评估，任何 prompt、tool 和 workflow 优化都缺乏可信反馈”。
- `AI Index 2026` 回答的不是“这一年发生了什么”，而是“能力进展、基础设施约束和社会部署正在如何共同改写现实边界”。

第一遍若没抓住旧问题，后面越读越容易变成名词堆。

### 第二遍：重建它的机制，而不是只记它的结论

很多人读完之后能复述“它说了什么”，却无法解释“为什么这件事会成立”。但只要不能把机制讲清楚，知识就无法迁移。

比如读 `MCP Architecture`，真正要重建的不是“host/client/server 的定义”，而是为什么协议要把 data layer 和 transport layer 分开，为什么 tools、resources、prompts、notifications 被视为不同原语。这些结构性的拆分，决定了你以后是写一堆耦合 glue code，还是在设计可替换的能力接口。

### 第三遍：找到它的边界、假设与误用方式

成熟的阅读不会只问“它多强”，还会问“它在什么前提下才强”。这一步尤其重要，因为一手源本身也有视角边界。

- 课程擅长搭骨架，但未必替你处理最新接口变化。
- 官方文档擅长讲接口和约束，但不会替你自动生成理论框架。
- benchmark 站点擅长暴露任务结构，但不会保证对真实业务直接外推。
- 年度报告擅长给出宏观校准，但不会替你做具体工程取舍。

如果一份来源的边界不被主动标出来，它迟早会被错误使用。

### 第四遍：把它转成一个可执行的下一步

读完一手源之后，至少要留下一个动作，而不只是一个印象。这个动作可以很小，但必须具体：

- 回写一段概念节点，把旧问题和机制讲清楚。
- 设计一个最小实验，检查一条方法是否真的适用于自己的任务。
- 改写一个现有工作流，让它更符合文档或协议揭示的结构。
- 调整学习路线，把原先低估的一层升格为主线。

如果一手源没有改变你下一步怎么学、怎么写、怎么做，它大概率还没有真正进入系统。

## 十二、两个对照案例：为什么同样读官方源，有人越读越稳，有人越读越散

### 案例 A：把官方源读成一条连续因果链

设想一个学习者想做 `coding agent`。更稳的读法通常会是这样的：

1. 先用 `AI Index 2026` 和 `International AI Safety Report 2026` 校正现实感，知道 agent 能力正在上升，但前沿能力是 jagged 的，安全与可靠性没有自动跟上。
2. 再用 `CS336` 和 `FSDL` 建骨架，理解模型、数据、训练、系统、部署并不是彼此独立的话题。
3. 再进入 `OpenAI Agents / Evals / Trace Grading` 与 `MCP Architecture`，学习今天的 agent workflow 是如何被组织和观测的。
4. 最后再看 `SWE-bench Verified`、`HCAST` 或 `METR Time Horizons`，把“我会搭”与“它在真实任务上到底到哪”分开。

这样读的结果，是知识会自动形成因果顺序：先看世界约束，再看能力机制，再看工程接口，最后看现实表现。

### 案例 B：把官方源读成互相孤立的收藏夹

另一种常见路径是：

- 先收藏一堆 agent 框架与 API 文档；
- 再收藏几个 leaderboard；
- 偶尔看一点课程，但没有形成系统顺序；
- 每读完一份材料，只留下若干“以后可能有用”的链接。

这种读法的问题，不是来源不够好，而是没有把来源放回它应在的位置上。结果往往是：

- 文档学会了调用，却没有建立评估纪律。
- benchmark 看到了分数，却没有理解任务结构。
- 课程看到了骨架，却没有进入真实构建。
- 年度报告看到了趋势，却没有映射回自己的技术主线。

久而久之，人会产生一种很危险的错觉：我读的全是一手源，所以我一定在进步。其实未必。来源是一手，不等于理解也是一手。

## 十三、把学习栈变成季度节律，而不是一次性热情

真正有效的学习栈，不是“按顺序刷完一遍”，而是不断回环。因为 AI 本身就是一条在快速变化的河流，你不可能指望靠一次系统学习永久解决路线感。

比较稳的做法，是把学习节律做成一个季度循环。

### 第一阶段：现实校准

每个季度先回看一次 `AI Index`、`International AI Safety Report`、`METR` 这类现实校准源。目的不是追新闻，而是修正判断：哪些能力进展值得升格为主线，哪些只是局部热闹。

### 第二阶段：机制深化

再回到 `CS336`、`Hugging Face Learn`、关键论文与课程讲义，把自己最薄的一层补厚。这一阶段最重要的，不是多，而是把先前只会复述的部分重新讲成自己的语言。

### 第三阶段：构建与实验

随后进入 `OpenAI Docs`、`MCP`、`vLLM`、`TensorRT-LLM` 等工程文档，在一个具体问题上做最小实现或最小实验。这里的关键词不是“做大”，而是“让知识承受真实约束”。

### 第四阶段：评估与回写

最后再用 `WebArena`、`OSWorld`、`SWE-bench Verified`、`HCAST`、`RE-Bench` 等 benchmark 或任务基准，把前面得到的能力想象压回现实，并把经验回写成地图页、节点页和 workflow 页。

一个季度结束时，你真正应该问自己的不是“我看了多少”，而是：

- 哪一层被我升格了？
- 哪一层仍然是口头理解？
- 哪条路线因为新证据而应该收缩或转向？

> 旁批：学习栈一旦进入节律，它就不再只是资料表，而会慢慢变成一种带有自我修正能力的研究习惯。

## 来源列表

- Stanford CS336
  https://cs336.stanford.edu/
- Full Stack Deep Learning
  https://fullstackdeeplearning.com/
- Hugging Face Learn
  https://huggingface.co/learn
- Hugging Face Agents Course
  https://huggingface.co/learn/agents-course/en/unit0/introduction
- OpenAI Developers
  https://developers.openai.com/
- OpenAI API: Agents SDK
  https://developers.openai.com/api/docs/guides/agents
- OpenAI API: Working with evals
  https://developers.openai.com/api/docs/guides/evals
- OpenAI API: Evaluation best practices
  https://developers.openai.com/api/docs/guides/evaluation-best-practices
- OpenAI API: Agent Builder
  https://developers.openai.com/api/docs/guides/agent-builder
- OpenAI API: Trace grading
  https://developers.openai.com/api/docs/guides/trace-grading
- Model Context Protocol
  https://modelcontextprotocol.io/
- MCP Architecture Overview
  https://modelcontextprotocol.io/docs/learn/architecture
- vLLM docs
  https://docs.vllm.ai/en/latest/
- vLLM Prefix Caching
  https://docs.vllm.ai/en/latest/design/prefix_caching/
- TensorRT-LLM Overview
  https://nvidia.github.io/TensorRT-LLM/overview.html
- TensorRT-LLM Performance Tuning Guide
  https://nvidia.github.io/TensorRT-LLM/performance/performance-tuning-guide/index.html
- Stanford HAI: The 2026 AI Index Report
  https://hai.stanford.edu/ai-index/2026-ai-index-report
- International AI Safety Report 2026
  https://internationalaisafetyreport.org/publication/international-ai-safety-report-2026
- WebArena-x
  https://webarena.dev/
- WebArena
  https://webarena.dev/og/
- OSWorld
  https://os-world.github.io/
- SWE-bench
  https://www.swebench.com/SWE-bench/
- SWE-bench Verified
  https://www.swebench.com/verified.html
- RE-Bench
  https://arxiv.org/abs/2411.15114
- HCAST
  https://metr.org/hcast.pdf
- METR Time Horizons
  https://metr.org/time-horizons/
- Measuring AI Ability to Complete Long Software Tasks
  https://arxiv.org/abs/2503.14499
- ARC Prize Foundation
  https://arcprize.org/about
- AIMO Prize
  https://aimoprize.com/participate
