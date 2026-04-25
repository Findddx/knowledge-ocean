# 推理系统与 GPU 优化地图

> 模型能力决定上限，推理系统决定这份能力能否被稳定、廉价、可预期地交付。很多人把“推理优化”理解成把同一个模型跑得更快一点；真正做进去才会发现，它更像一门资源编排学：如何把长度不一、延迟敏感、并发波动极大的请求，整理成 GPU 愿意高效处理的工作。

## 1. 为什么这条线值得长期投入

训练决定模型会什么，推理系统决定这些能力会不会在真实流量里活下来。只要应用离开 demo，问题就会立刻从“模型强不强”转成一组更硬的矛盾：

- 长上下文通常更有帮助，但会明显抬高首 token 延迟；
- 更大的 batch 往往能提升吞吐，却可能恶化尾延迟；
- KV cache 能省掉重复计算，却会把问题转移到显存容量、碎片和淘汰策略上；
- 更激进的调度、跨请求复用和 speculative decoding 能提速，但会引入更高的复杂度、兼容性成本和安全边界问题。

所以推理系统不是“模型之后的配件”，而是把模型能力翻译成产品能力的中间层。真正稀缺的，不是知道多少框架名，而是知道一个慢系统到底慢在算力、访存、调度、隔离，还是慢在自己对工作负载的想象过于粗糙。

## 2. 先把一次请求走完

理解推理系统，第一步不是背术语，而是把一条请求拆成一条阶段差异非常明显的流水线：

1. 请求进入服务层，完成排队、路由、鉴权、配额与并发控制。
2. 文本被 tokenizer 和上层预处理逻辑变成 token 序列。
3. 系统进入 `prefill`，一次性读取整段输入，建立后续生成所需的注意力状态。
4. 系统进入 `decode`，按小步循环不断生成新 token，并读取已缓存状态继续向前。
5. 输出被流式返回、计费、记录日志，并进入上层业务流程。

TensorRT-LLM 的 Performance Tuning Guide 明确把推理分成 `Context (Prefill) Phase` 与 `Generation Phase`。这不是教学式分类，而是整个系统设计的起点，因为两个阶段消耗资源的方式并不一样。结合 vLLM 与 TensorRT-LLM 的一手资料，可以得到一个非常稳定的工程判断：

- `prefill` 更像一次大规模读取输入并建立状态，常受长上下文、共享前缀复用和初始算力影响；
- `decode` 更像一个重复的小步循环，常受 KV cache 组织、访存效率、调度与批处理策略影响。

这不是某一页文档的原话，而是对多份官方资料的归纳。后面几乎所有推理优化原语，都是围绕这两段在做文章。

## 3. 一张真正有用的系统地图：你到底在优化什么

谈推理系统最怕一上来就说“我要优化 latency”，因为 latency 只是结果，不是结构。更有用的做法，是先把系统压力拆成四层。

### 3.1 计算层：有没有做太多本可避免的算

长 prompt 的首次处理本来就贵，但很多系统慢，不是因为模型太大，而是因为重复做了本可复用的前缀计算。Prefix caching、KV reuse、chunked prefill 这类机制，本质上都在回答同一个问题：哪些计算是必须重做的，哪些只是因为系统组织差而被迫重做。

### 3.2 显存层：是不是把显存浪费在了错误的数据结构上

高并发 serving 里，显存已经不是单纯的“够不够大”问题，而是“是否被高质量组织”问题。KV cache 是否连续分配、块是否可复用、是否容易碎片化、何时淘汰、是否支持多租户隔离，这些都会直接决定并发能力。

### 3.3 调度层：GPU 有没有在空等

vLLM 强调 `continuous batching`，TensorRT-LLM 强调 `inflight batching`。名字不同，直觉一致：GPU 不应该被绑死在“跑完这一批，再处理下一批”的线性时间里，而应持续接收可混合的工作，尽量让 prefill 与 decode 都保持高价值占用。

### 3.4 拓扑层：你是在为性能扩卡，还是只是为了塞得下

当模型规模、上下文长度和吞吐目标继续上升，单卡很快不够。此时 TP、PP、EP、context parallelism 不再是研究名词，而是交付条件的一部分。真正重要的不是缩写，而是搞清楚：你是在用更多卡换更低延迟、换更高吞吐，还是只是为了让模型勉强部署成功。

## 4. 关键指标：不要只盯一个 `tokens/sec`

成熟的推理系统不会用单一指标描述自己。至少要同时看下面三类信号。

### 4.1 用户感知指标

- `TTFT`：首 token 延迟，决定系统是否“像在思考”；
- 平均每 token 延迟：决定流式交互是否顺滑；
- 尾延迟：决定高峰期体验是否突然塌陷。

### 4.2 运营指标

- 吞吐量：单位时间处理多少请求或 token；
- 并发容量：在 SLA 内能支撑多少活跃请求；
- 单位成本：每 token、每会话、每成功任务到底多少钱。

### 4.3 系统健康指标

- GPU 利用率是否持续而非短暂冲高；
- KV cache 命中、复用、offload、eviction 是否健康；
- OOM、降级、重试、超时是否频繁；
- 是否为了吞吐挤压了 TTFT 与尾延迟。

如果指标体系不完整，优化就很容易变成局部幻觉。很多所谓“吞吐提升”，其实只是把等待时间偷偷转嫁给用户。

## 5. 不同 workload，会把系统推向不同形状

真正成熟的系统判断，不是“我喜欢哪套框架”，而是“我的流量把我推向哪种系统形状”。下面三类工作负载，常常会把设计重点拉向完全不同的地方。

### 5.1 交互式助手：首 token 比峰值吞吐更重要

典型形态是知识助手、企业问答、客服对话。它们往往有：

- 明显的共享前缀，比如长 system prompt、工具描述、统一规则；
- 较强的 TTFT 敏感性；
- 输入长度差异大，但输出通常中等。

这类系统通常优先关心：

- 前缀复用是否真实命中；
- system prompt 是否被写得过长；
- `prefill` 与 `decode` 的混排是否拖累首 token；
- 缓存复用是否在多租户里保持隔离。

这里一个常见误判是：觉得“只要支持 prefix caching，TTFT 就会自然下降”。TensorRT-LLM 的 KV cache reuse 文档明确指出，复用状态只有在前一个请求完成后才可被后续请求重用；如果共享前缀的一批请求几乎同时进来，第一批往往仍要完整付费。这意味着“支持某特性”与“你的流量真的从中获益”是两回事。

### 5.2 代码与 agent 工作负载：尾延迟和长尾输出更棘手

代码补全、多步 agent、tool-heavy workflow 常常有：

- 输出长度长尾更明显；
- 请求间共享前缀不稳定；
- decode 阶段比普通问答更容易成为主瓶颈；
- 大量异步工具调用导致流量形态更抖动。

这类系统通常更需要关注：

- decode 阶段是否被小 batch 拖垮；
- speculative decoding 是否有真实接受率；
- 长输出下 KV cache 膨胀是否导致淘汰过早；
- 调度是否让 GPU 经常处于“忙着切换、没在做价值工作”的状态。

值得注意的是，TensorRT-LLM 的 speculative decoding 文档明确说这类技术尤其适用于小 batch 导致 GPU 利用率不足的情况。这意味着 speculative decoding 不是普适加速按钮，而更像一种“当 decode 细碎到喂不饱 GPU 时”的修补机制。

### 5.3 离线批处理：吞吐比交互感更重要

文档摘要、离线抽取、批量生成常常容忍更高延迟，却极度在意总吞吐和单位成本。此时系统更可能接受：

- 更大的 batch；
- 更强的离线调度；
- 更激进的量化与更高的 GPU memory utilization；
- 甚至接受一定的 CPU offloading，只要整体吞吐与成本更优。

这类系统最容易犯的错，是把在线交互指标硬套过来。对离线批处理来说，`TTFT` 未必是主目标；但如果你把在线配置直接搬过去，也常会莫名浪费硬件。

## 6. 真正改变系统形状的，不是框架名，而是原语组合

### 6.1 KV cache：把历史状态保留下来

KV cache 的意义不是“高级特性”，而是让 decode 免于反复重算历史。TensorRT-LLM 最新文档把 KV cache 定义得很直白：它存储已经计算过的 key-value pairs，在生成阶段复用，并支持跨请求复用、offloading 与优先级淘汰。只要你开始服务长输出或多轮对话，它就是整套系统的地基。

### 6.2 PagedAttention / paged KV：把显存从连续分配改成分页分配

PagedAttention 论文的核心贡献，不是“注意力算法更优雅”，而是把 KV cache 的管理方式改写成分页问题。论文把收益概括为两点：`near-zero waste` 和 `flexible sharing of KV cache within and across requests`。这两点非常关键，因为 serving 系统里真正贵的，往往不是多做一两次算，而是显存被低效组织后，批量和并发都被硬性限制。

### 6.3 Prefix caching / KV reuse：别重复为同一段前缀付费

vLLM 的 prefix caching 文档采用哈希化 block 方案，只缓存完整 block，并把 `cache salts`、LoRA ID、多模态输入哈希一起纳入 cache key。其当前配置文档还把 `sha256` 明确列为默认哈希算法，并提醒非加密哈希虽然更快，但会提高碰撞与多租户泄露风险。这里最重要的启发是：前缀缓存不只是提速结构，还是正确性和隔离结构。

TensorRT-LLM 则从另一个角度补足了边界条件：KV cache reuse 默认开启，但收益取决于共享前缀长度、请求时序、`tokens_per_block` 设定，以及可复用块是否已被 LRU 淘汰。默认 block 是 128 tokens，这本身就是一种工程权衡：更大的块对 kernel 更友好，但会降低复用粒度。

### 6.4 Batching / scheduling：让 GPU 始终在做值钱的事

现代推理系统的难点不再只是“batch 开多大”，而是“不同阶段、不同长度、不同租户的请求如何共享 GPU 时间”。vLLM 把 `continuous batching` 作为核心能力，TensorRT-LLM 则把 inflight batching 与 overlap scheduling 作为性能关键。这些名字背后只有一个朴素目标：让 GPU 持续拿到可并行、可复用、对用户真的有价值的工作。

### 6.5 Speculative decoding：改变 decode 的推进粒度

从原始 speculative decoding 论文到 TensorRT-LLM 的实现文档，这条线都在试图回答同一个问题：如果目标模型每轮只稳妥地产出一个 token，那么 decode 会被“极小步前进”拖慢；能否先便宜地提出一串候选，再一次性验证它们？TensorRT-LLM 的文档把它定义为“一组允许每轮前向生成多个 token 的技术”，并明确说它在 GPU 因小 batch 而利用率不足时尤其有价值。

因此，speculative decoding 不是“神奇快一点”，而是在重写 decode 的时间粒度。

## 7. vLLM 与 TensorRT-LLM：不是谁替代谁，而是两种工程重心

### 7.1 vLLM 更像推理原语实验场

vLLM 的优势在于，把 PagedAttention、continuous batching、prefix caching、spec decode、多种并行策略等原语，做成一个相对灵活、易试验的开源基座。它特别适合回答“某个原语为什么存在，它和别的原语如何配合”。

### 7.2 TensorRT-LLM 更像面向 NVIDIA 栈的极限工程化

TensorRT-LLM 的价值则更偏向硬件亲和、build-time/runtime 参数、KV cache system、speculative decoding、量化、多卡调优与具体 GPU 上的极限性能。它更适合回答“在 NVIDIA 环境里，怎样把这类工作负载压到更低延迟或更高吞吐”。

### 7.3 选择不该是意识形态问题

- 你要快速试验原语、做多模型 serving、保持开源生态灵活性，vLLM 往往更自然；
- 你强依赖 NVIDIA 栈、追求极限性能，且愿意投入更高构建与调优复杂度，TensorRT-LLM 往往更匹配。

不要问“哪个框架更先进”，要问“哪个框架更符合我的硬件、流量形态、维护预算和隔离要求”。

## 8. 一个可迁移的调优决策框架

推理系统优化最怕拍脑袋。更可靠的做法，是先判断自己面对的是哪类结构性问题。

| 症状 | 优先怀疑 | 常见误判 | 更稳妥的判断方式 |
| --- | --- | --- | --- |
| `TTFT` 很高 | `prefill`、共享前缀复用、system prompt 过长 | 以为模型大就是唯一原因 | 先看长前缀占比、复用命中、请求是否并发到来 |
| 并发一上来就 OOM | KV cache 膨胀、分页与淘汰策略差 | 一味加卡 | 先看 block 组织、dtype、eviction、offload 是否合理 |
| 吞吐低且 GPU 不饱和 | 调度、batch 过小、decode 太碎 | 继续调模型参数 | 先看 GPU 是否被有效喂饱，再考虑 speculation |
| 共享 system prompt 却几乎无加速 | 复用时序不成立、块太大、缓存已被淘汰 | 以为缓存实现有 bug | 先确认第一批请求是否先结束、block 是否过粗 |
| 多租户场景担心缓存串味 | cache key、salt、租户隔离策略缺失 | 把性能问题和安全问题分开看 | 直接把缓存视为安全边界的一部分 |
| 多卡扩展性很差 | 并行策略和通信开销 | 卡不够多 | 先判断自己要的是更低延迟、更高吞吐，还是仅仅部署成功 |

这个表真正想训练你的，不是“看到一个症状就开一个开关”，而是学会先找结构性瓶颈，再选原语。

## 9. 一个简短案例：同一个知识助手，为什么会有三种完全不同的 serving 形状

想象一个企业知识助手，system prompt 很长，工具描述固定，用户问题长度波动大。

### 第一阶段：只有少量用户

这时最重要的通常不是极限吞吐，而是把 TTFT 压下来。共享前缀明确，prefix caching 往往很值钱，系统 prompt 的压缩与分层也很值钱。

### 第二阶段：用户开始多起来

这时问题会转向缓存管理。你会发现共享前缀不一定稳定命中，因为很多请求会在前一个请求结束前就并发进入；KV cache 很快从“优化项”变成“容量项”；分页、淘汰、offload 才开始真正重要。

### 第三阶段：系统接入 agent workflow

一旦输出长度变长、请求更碎、工具调用更频繁，瓶颈又会往 decode 与调度方向移动。此时 speculation、batching 与并行策略可能比单纯的前缀缓存更决定体验。

这个案例想说明的不是某个系统一定要按三步走，而是：同一个产品，只要流量形态变了，系统最值钱的优化杠杆就会变。

## 10. 常见误判：为什么很多“优化”最后没有带来真实收益

### 10.1 把所有问题都理解成算力问题

很多系统看起来慢，真实原因却是显存组织差、调度差、前缀复用差。只加卡，不一定能解决结构性浪费。

### 10.2 只看离线 benchmark，不看在线体验

吞吐更高不代表交互更好。很多线下漂亮结果，线上一看只是把 TTFT 和尾延迟交给了用户。

### 10.3 把“支持某特性”误当成“已经获得收益”

打开 prefix caching、speculative decoding、KV reuse，不代表它们真的在你的工作负载上带来净收益。真实收益取决于命中率、时序、接受率、淘汰策略和隔离约束。

### 10.4 忽视缓存的安全语义

一旦做跨请求复用，缓存就不只是性能结构。TensorRT-LLM 的 `cache_salt` 与 vLLM 的多租户 cache salts 都在提醒你：缓存既是优化手段，也是隔离边界。

### 10.5 把框架选择误当成架构选择

框架很重要，但它只是把某些原语实现得更顺手。真正决定成败的，仍然是你是否理解自己的瓶颈结构。

## 11. 这张地图真正想留下什么

推理系统不是一堆高级术语，而是一门关于资源编排的实践科学。它逼你反复回答三件事：

1. 哪些计算是必须做的，哪些是重复做的；
2. 哪些显存是有效占用，哪些只是被糟糕的数据结构浪费掉的；
3. GPU 正在忙什么，而它忙的事是否真的对应用户价值。

只要这三件事想清楚，后面无论遇到 vLLM、TensorRT-LLM、SGLang 或其他系统，你都会更容易抓住本质。

## 来源

- vLLM official docs  
  https://docs.vllm.ai/
- vLLM, *Automatic Prefix Caching*  
  https://docs.vllm.ai/en/latest/design/prefix_caching/
- vLLM, *CacheConfig*  
  https://docs.vllm.ai/en/latest/api/vllm/config/cache/
- *Efficient Memory Management for Large Language Model Serving with PagedAttention*, arXiv:2309.06180  
  https://arxiv.org/abs/2309.06180
- TensorRT-LLM overview  
  https://nvidia.github.io/TensorRT-LLM/latest/overview.html
- TensorRT-LLM, *Performance Tuning Guide*  
  https://nvidia.github.io/TensorRT-LLM/performance/performance-tuning-guide/index.html
- TensorRT-LLM, *KV cache reuse*  
  https://nvidia.github.io/TensorRT-LLM/advanced/kv-cache-reuse.html
- TensorRT-LLM, *KV Cache System*  
  https://nvidia.github.io/TensorRT-LLM/latest/features/kvcache.html
- TensorRT-LLM, *Speculative Sampling / Speculative Decoding*  
  https://nvidia.github.io/TensorRT-LLM/advanced/speculative-decoding.html
- *Fast Inference from Transformers via Speculative Decoding*, arXiv:2211.17192  
  https://arxiv.org/abs/2211.17192
- *Accelerating Large Language Model Decoding with Speculative Sampling*, arXiv:2302.01318  
  https://arxiv.org/abs/2302.01318
