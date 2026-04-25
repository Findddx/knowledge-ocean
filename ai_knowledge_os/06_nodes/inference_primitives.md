# 推理原语：KV Cache、PagedAttention、Prefix Caching 与 Speculative Decoding

> 这一页不是为了让你记住几个热词，而是为了把一个更根本的问题讲透：为什么一个模型在实验室里看起来很强，一进真实流量就突然变慢、变贵、变脆。只要把这四类原语想明白，你对 serving 的理解就会从“知道功能列表”升级成“知道系统为什么非这么设计不可”。

## 1. 先把问题问对：生成一个 token，到底发生了什么

任何自回归模型都在重复做同一件事：读历史、算下一步、追加新 token、再读更长历史。如果每一轮都从头读取全部上下文并重算注意力，成本会迅速失控。

因此，现代推理优化首先不是在追求“更聪明的模型”，而是在追求“更聪明地管理重复劳动”。真正的问题可以压缩成四句：

1. 已经算过的历史状态，能不能别再算？
2. 必须缓存下来时，能不能别把显存浪费掉？
3. 多个请求共享前缀时，能不能安全复用？
4. decode 阶段每轮只推进一个 token 这件事，能不能被改写？

KV cache、PagedAttention、prefix caching 与 speculative decoding，正是在分别回答这四个问题。

## 2. Prefill 与 decode：为什么同一个请求像在经过两种完全不同的经济体系

TensorRT-LLM 的 Performance Tuning Guide 把推理明确分成 `Context (Prefill) Phase` 与 `Generation Phase`。这两个阶段的差异，决定了后面所有原语为什么存在。

### 2.1 Prefill：先把整段输入“读进去”

在 prefill 阶段，模型一次性处理整段 prompt，建立后续生成所需的注意力状态。长 system prompt、很长的聊天历史、超长文档输入，主要都在这里付费。

### 2.2 Decode：在已有历史上继续前进

进入 decode 之后，系统开始做大量细碎的小步迭代。每轮新增 token 很少，但必须不断读取已缓存状态并保持调度活跃。正因为它是一个高频循环，任何“每轮只省一点点”的改进，都可能在总延迟上变得非常显著。

可以把后面四类原语的角色简单理解成：

- KV cache：避免重复计算历史；
- PagedAttention：避免糟糕地存历史；
- Prefix caching：避免重复处理相同前缀；
- Speculative decoding：避免一次只往前挪最小一步。

## 3. KV cache：把“已经看过的历史”保留下来

### 3.1 它本质上在做什么

KV cache 的思路并不神秘：历史 token 对应的 key/value 已经算出来了，后续生成时就不要再算一遍，而是直接把这些状态缓存下来，供后续 attention 读取。没有 KV cache，decode 的成本会随着历史长度恶化得非常快。

### 3.2 为什么它几乎是现代 serving 的地基

训练里的稀缺资源往往是总算力；serving 里的稀缺资源往往是“如何在可接受延迟里处理不断增长的历史”。KV cache 之所以重要，不是因为它让模型更聪明，而是因为它让系统勉强有机会在真实交互里活下来。

### 3.3 它改变了什么系统设计

一旦引入 KV cache，问题就从“算不算”变成了“怎么存、怎么管、怎么复用、怎么淘汰”。TensorRT-LLM 最新的 KV Cache System 文档把它描述成一个 block pool，并支持跨请求复用、优先级淘汰、offloading，以及 variable attention window、MQA/GQA 等优化。换句话说，缓存不是一个开关，而是一整套资源系统。

### 3.4 边界在哪里

缓存不是免费的。它会把瓶颈从算力转移到显存容量、访存带宽、块组织与淘汰策略上。很多系统“越跑越慢”，根本不是模型变笨了，而是缓存开始吞掉系统的结构性余量。

## 4. PagedAttention：真正难的不是“把历史存下来”，而是“怎么存”

### 4.1 为什么连续分配会成为问题

高并发 serving 下，请求的上下文长度、生成长度、寿命都不一致。如果强迫每个请求占据一整段连续显存，就很容易遇到碎片化、扩缩困难和低复用率。

### 4.2 PagedAttention 的核心贡献是什么

PagedAttention 论文把 KV cache 管理类比成操作系统分页：不是要求每个请求拿到一大段连续显存，而是把缓存拆成固定大小的 page / block，再通过映射关系组织它们。论文摘要里把收益概括为两点：`near-zero waste` 和 `flexible sharing of KV cache within and across requests`。

这两点为什么重要？因为 serving 里很多吞吐上限并不是被 FLOPs 打满，而是先被显存碎片、无效复制和低质量复用掐住。

### 4.3 它改变的是系统形状，不是单个算子

很多人第一次读 PagedAttention，会把注意力放在 attention kernel 本身。更深的看法是：它改变了推理系统的内存模型。只要缓存不再依赖连续分配，块复用、块回收、跨请求共享、有限窗口裁剪和 LRU 风格淘汰才真正变得可管理。

所以 PagedAttention 最重要的启发不是“多了一个 clever trick”，而是“推理系统终于开始像操作系统一样认真对待内存管理”。

## 5. Prefix caching：不是所有 prefill 都值得重做

### 5.1 它在复用什么

许多真实请求并不是彼此独立的。相同的 system prompt、相同的工具描述、相同的长文档开头、相同的会话历史前缀，都会导致多个请求前半段拥有完全一致的 token 序列。对这类请求，再跑一次完整 prefill，通常就是纯浪费。

Prefix caching 的目标，就是复用这些共享前缀对应的 KV 状态。

### 5.2 vLLM 的设计告诉了我们什么

vLLM 的 Automatic Prefix Caching 文档明确说，它采用 `hash-based approach`，并用父块哈希、块内 token、额外哈希信息共同决定 block 身份。文档还特别强调三点：

- 只缓存 `full blocks`；
- `extra hashes` 可包含 LoRA IDs、多模态输入哈希与 `cache salts`；
- 当前配置里 `enable_prefix_caching` 默认开启，`sha256` 是默认哈希算法，而非加密哈希虽然更快，但会提高碰撞与多租户风险。

把这三点连起来看，就会发现 prefix caching 从来不是“命中率越高越好”这么简单。它还必须回答：

1. 复用粒度到什么程度；
2. 哈希碰撞如何避免；
3. 多租户或多配置场景下如何避免错误复用。

### 5.3 TensorRT-LLM 又补上了什么现实边界

TensorRT-LLM 的 KV cache reuse 文档把很多实现边界说得更直接：

- 共享前缀确实可以显著降低 TTFT；
- 但可复用状态只有在前一个请求终止后才会进入可复用集合；
- 大 batch、长输出会提高内存压力，从而降低复用发生的概率；
- block 越大，kernel 可能更高效，但复用粒度会更粗；
- 内存不够时，可复用块会被 LRU 风格策略驱逐；
- 使用 p-tuning 时，仅靠 input ids 可能导致错误复用，需要 `extra_ids` 区分 prompt embedding。

这里最重要的结论是：缓存复用不是“支持即可获益”，而是一个同时受时序、块大小、工作负载与隔离策略支配的系统行为。

### 5.4 一个经常被误会的事实

很多团队看到“共享 system prompt 很多”，就以为 prefix caching 必然能大幅提速。实际上，如果一批请求几乎同时进入，第一批请求往往仍要完整支付 prefill 成本，因为复用状态还没来得及变成可共享状态。这不是实现 bug，而是机制边界。

## 6. Speculative decoding：能不能一轮前向换来不止一个 token

### 6.1 它为什么出现

decode 的根本痛点在于：系统往往要做大量相似的小步前向，而每轮只换来一个 token。原始 speculative decoding 论文与 speculative sampling 论文都在试图改写这件事：先用更便宜的方法产生一串候选 token，再让目标模型一次性验证。

### 6.2 它的机制其实很朴素

不管是 draft-target、N-Gram、Medusa、ReDrafter 还是 EAGLE，本质都在做两件事：

1. 便宜地提出若干未来 token 候选；
2. 用更贵的目标模型一次性验证并接受其中一部分。

TensorRT-LLM 的官方定义很直接：speculative decoding 是“一组允许每轮前向生成多个 token 的技术”。文档还明确指出，它在 GPU 因 batch 较小而利用率不足时尤其有价值。

### 6.3 它什么时候最可能有效

TensorRT-LLM 的 speculative decoding 文档给了两个非常关键的条件：

- 同时处理多个 draft token 的代价，最好不要显著高于处理一个 token；
- draft token 应当有足够高的验证通过率。

换成工程语言就是：如果候选接受率太差，或者系统根本不缺 decode 吞吐，只是卡在 prefill 与缓存组织上，那么 speculative decoding 的收益就会很有限。

### 6.4 它为什么会改变系统设计

Speculative decoding 的深层意义，不只是快一点，而是它第一次明确地告诉系统设计者：你可以重新设计 decode 的时间粒度。优化不再只有“少做重复算”，也可以是“每轮前向推进更多有效工作”。

## 7. 这四类原语是怎么互相配合的

单独理解每个概念还不够，真正重要的是看它们如何组合。

### 7.1 KV cache 是底座

没有 KV cache，后面的分页、跨请求复用和 speculative decoding 几乎都失去意义。它回答的是“历史状态是否值得保存”。

### 7.2 PagedAttention 是组织层

它不直接减少要保存的内容，但会让这些缓存更可管理、更少碎片、更容易复用。它回答的是“这些状态该怎么保存”。

### 7.3 Prefix caching 是跨请求复用层

它把 prefill 的收益从“单请求内部”扩展到“请求之间”。它回答的是“谁能共享这些已经保存的状态”。

### 7.4 Speculative decoding 是 decode 提速层

它不改变缓存本身，但改变消费缓存的节奏。它回答的是“如何更快地利用这些状态向前生成”。

如果一定要压缩成一句话：

- KV cache 决定“存不存”；
- PagedAttention 决定“怎么存”；
- Prefix caching 决定“谁能复用”；
- Speculative decoding 决定“怎么更快往前走”。

## 8. 一个案例：为什么同样有共享前缀，系统 A 获益明显，系统 B 几乎没有收益

想象两个都使用长 system prompt 的知识助手。

### 系统 A：请求较稀疏

用户请求间隔较长，上一条请求常常先结束，下一条才进来。此时共享前缀很容易变成可复用状态，TTFT 可以明显下降。

### 系统 B：高并发同时突发

很多请求在同一时间进入。虽然它们共享完全相同的 system prompt，但第一波请求还没结束，缓存还没变成可复用块，后续请求也就很难即时受益。与此同时，大 batch 和长输出还会增加内存压力，让可复用块更早被淘汰。

这个案例想说明的是：所谓“有很多共享前缀”，只是 prefix caching 值得考虑的必要条件，不是充分条件。时序、内存压力和块大小，同样决定收益。

## 9. 一个更可迁移的诊断表

| 症状 | 先怀疑什么 | 为什么 |
| --- | --- | --- |
| `TTFT` 很高 | `prefill`、prefix caching、system prompt 过长 | 首 token 常先被整段输入处理拖慢 |
| 并发一上来就 OOM | KV cache 膨胀、分页与淘汰策略、cache dtype | 缓存占用与碎片问题在并发下同步放大 |
| 吞吐低但 GPU 不饱和 | batch 太小、调度差、speculative decoding 未发挥 | 可能不是模型算不过来，而是 GPU 没被有效喂饱 |
| 相同前缀几乎无加速 | 请求时序不对、block 太大、复用块已淘汰 | 复用不是自动即时发生的 |
| 多租户担心缓存串味 | cache salt、cache key 设计、隔离边界缺失 | 性能机制已经触及安全边界 |

## 10. 常见反例：什么时候不该对某个原语抱过高期待

### 10.1 不要把 prefix caching 当成实时共享魔法

它对共享前缀很有用，但并不是“同一批同时到来的请求都能立即占便宜”。

### 10.2 不要把 speculative decoding 当成普适加速

如果瓶颈主要在 prefill、缓存管理或超长输入，speculation 往往不是第一优先级。

### 10.3 不要把 PagedAttention 理解成“更快的 attention kernel”

它真正改变的是内存模型，而不是单轮 attention 的某个局部算子。

### 10.4 不要把 KV cache 理解成“简单缓存”

一旦进入真实 serving，它会牵出 block、大页、小页、eviction、offload、复用、安全隔离等整套系统问题。

## 11. 练习题

1. 一个代码 agent 的 `TTFT` 很低，但长输出阶段吞吐差、GPU 利用率偏低。你会优先怀疑哪一层，为什么？
2. 一个企业知识助手共享 2,000 token 的 system prompt，却几乎看不到 prefix caching 收益。列出三个不依赖“实现有 bug”的解释。
3. 如果多租户环境里担心 prompt theft，你会怎样重新理解 prefix caching 的价值与风险？

## 12. 读完这一页后，真正该记住什么

推理原语最迷惑人的地方，在于它们听起来像一堆“高级特性”，其实只是朴素工程原则的不同体现：

- KV cache：别反复重算历史。
- PagedAttention：别用糟糕的方式占内存。
- Prefix caching：同样的前缀别重复预填充。
- Speculative decoding：别每轮只推进最小步。

真正的高手，不是背会这些词，而是能在看见某种 workload 时立刻判断：它更像重复计算问题、显存组织问题、跨请求复用问题，还是 decode 推进粒度问题。

## 来源

- *Efficient Memory Management for Large Language Model Serving with PagedAttention*, arXiv:2309.06180  
  https://arxiv.org/abs/2309.06180
- vLLM official docs  
  https://docs.vllm.ai/
- vLLM, *Automatic Prefix Caching*  
  https://docs.vllm.ai/en/latest/design/prefix_caching/
- vLLM, *CacheConfig*  
  https://docs.vllm.ai/en/latest/api/vllm/config/cache/
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
