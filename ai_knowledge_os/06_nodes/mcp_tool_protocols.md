# MCP、Tool Protocol 与外部能力接入

> 2026 年的 agent 系统，真正的分水岭已经不只是“模型会不会说”，而是“系统能不能把外部世界接进来，同时不把自己弄坏”。MCP 的价值就在这里：它试图把工具接入、上下文暴露、用户追问、权限协商这些原本散落在胶水代码里的问题，提升成一套可发现、可协商、可审计、可鉴权的协议层。

## 1. 为什么 MCP 不能被理解成 function calling 的别名

很多人第一次接触 MCP，会以为它只是“函数调用的标准化版本”。这个理解太浅。Function calling 解决的是一个局部问题：模型如何按 schema 请求某个动作。MCP 解决的是一个更完整的问题：一个 AI 应用如何长期、系统化、可治理地接入外部能力世界。

只要系统开始连接文件系统、文档库、数据库、SaaS、企业权限体系，问题就会立刻变复杂：

- 工具如何被发现，而不是硬编码在 prompt 里？
- 哪些信息应作为上下文暴露，哪些能力应作为动作暴露？
- 控制权该交给模型、应用，还是用户？
- 远程能力需要用户身份时，授权与 token 流转由谁负责？
- 当能力变化时，client 如何得知并安全地适配？

MCP 的价值，不在于“比 function calling 多几个字段”，而在于它把这些原本写在私有 glue code 里的问题，正式提升成了协议问题。

## 2. 最小心智模型：先把谁在做决定这件事分清楚

MCP 官方架构文档把参与方拆成三类：

- `Host`：真正承载 AI 应用的宿主；
- `Client`：Host 为每个 MCP server 建立的连接组件；
- `Server`：暴露能力的程序，可以提供 tools、resources、prompts。

这个拆分极其重要，因为它强迫你把“谁拥有能力”与“谁决定使用能力”分开。模型不再直接“拥有工具”，而是通过 host 与 client，在协议边界上接触工具、资源和提示模板。这样一来，权限边界、生命周期管理和能力协商才有地方落脚。

如果没有这层拆分，系统很容易把模型、工具实现、权限逻辑和 UI 粘成一团。短期能跑，长期很难维护。

## 3. MCP 的骨架：生命周期、能力协商与传输

### 3.1 它先是协议，再是生态

MCP 规范明确使用 JSON-RPC 2.0 作为消息基础。生命周期规范要求双方通过 `initialize` 完成版本与能力协商，而且初始化请求不能和其他消息混在同一个 JSON-RPC batch 中。这个细节听起来很底层，真正重要之处在于：MCP 把“接入顺序”和“能力声明”都规定成了协议的一部分。

换句话说，MCP 的第一层价值，不是“你能接更多工具”，而是“接入本身开始有秩序”。

### 3.2 它把问题拆成两层

官方架构文档把 MCP 分成 data layer 与 transport layer。前者负责消息、生命周期、原语和能力；后者负责连接、消息传输和授权相关约束。这个分层很像网络协议的思维：不要把“系统暴露什么能力”与“系统通过什么路径接入、如何鉴权”混成一个概念。

### 3.3 能力协商不是形式主义

到 2025-06-18 的稳定规范，MCP 已经把 resources、prompts、tools、sampling、roots、elicitation 等能力都纳入协议结构。真正稳妥的实现不会假设“所有 client 都支持所有能力”，而是把 capability negotiation 当作设计前提。

## 4. MCP 最重要的设计，不是功能列表，而是控制权分层

MCP server 最核心的三类原语，是 `Tools`、`Resources`、`Prompts`。这三者最大的区别，不在数据形态，而在控制权归属。

### 4.1 Tools：模型控制的动作

Tools 规范写得很直接：tools 是可被语言模型调用的外部操作。它们是 model-controlled 的，但出于 trust & safety，规范同时要求应用应当始终允许用户拒绝工具调用，并在 UI 中清楚暴露工具被调用这件事。

这意味着 tool 的本质不是“一个 API”，而是“一项在模型控制下、需要审计与审批意识的动作能力”。

因此，适合做 tool 的东西通常具备两个特征：

- 它本质上是 action，而不是 context；
- 让模型决定何时调用，比由应用静态注入上下文更合理。

例如“发送消息”“创建日程”“写入数据库”“执行查询”都更像 tool。

### 4.2 Resources：应用控制的上下文

Resources 规范把它定义为服务器向客户端暴露上下文数据的标准方式。文档明确说 resources 是 `application-driven` 的，也就是由 host / client 决定如何将这些数据纳入上下文。它们常见于文件、数据库 schema、只读知识、应用配置等场景。

这里最关键的设计判断是：如果某份信息更像“被读取和挑选的上下文”，而不是“让模型决定是否执行的动作”，那它通常就该是 resource，而不是 tool。

把本该是 resource 的东西做成 tool，往往会平白增加模型决策负担与调用不确定性。

### 4.3 Prompts：用户控制的工作流入口

Prompts 规范明确说 prompts 是 `user-controlled` 的，设计目标是让用户显式选择它们来触发某类工作流。它们更像带参数的任务入口，而不是模型会自动触发的函数。

一旦这样理解，prompt 的位置就会很清楚：

- tool 提供动作能力；
- resource 提供上下文材料；
- prompt 负责把这些能力组织成用户可理解、可选择的任务形状。

## 5. 一个可迁移的设计表：什么时候该用 tool、resource、prompt

| 你要暴露的东西 | 更合适的原语 | 谁应拥有主导权 | 常见误判 |
| --- | --- | --- | --- |
| 会改动外部世界的动作 | Tool | 模型，但要有人类兜底 | 把写操作伪装成普通上下文 |
| 稳定的只读知识或上下文材料 | Resource | 应用 / host | 把文档读取强行做成 tool |
| 一类可参数化的任务入口 | Prompt | 用户 | 让模型自动触发本该由用户选择的流程 |
| 需要补充非敏感字段才能继续 | Elicitation | server 发起，用户确认 | 用自由文本聊天代替结构化追问 |

这张表最重要的价值，是帮你在设计阶段先分清控制权，而不是一上来就把一切都塞进 tool。

## 6. Tool 不是“远程函数”那么简单：结果通道同样是协议设计的一部分

MCP 很容易被低估的一点是：tool 不只定义“怎么调用”，还定义“怎么返回”。2025-06-18 的 tools 规范明确允许 tool 结果包含：

- 普通 `content`，例如 text / image / audio；
- `structuredContent`，也就是结构化 JSON 结果；
- `resource_link`，返回一个可供 client 后续读取的资源链接；
- embedded `resource`，直接把资源内容嵌入结果中；
- 可选的 `outputSchema`，帮助 client 对结构化结果做验证。

这件事为什么重要？因为它说明 MCP tool 不是“把一个 RPC 包装给模型”这么简单。它还在定义：

- 结果是给模型看的，还是给用户看的；
- 结果应该继续进入上下文，还是只交给上层 UI；
- 结果是一次性的字符串，还是可被客户端继续获取、展示、缓存、订阅的资源。

从系统设计看，这比 function calling 的普通 schema 更进一步，因为它把“动作结果怎样流入系统后续状态”也协议化了。

## 7. Client-side primitives：MCP 不是单向暴露，而是双向协作

MCP 不只是 server 给 client 能力。它同样定义了 client 侧可暴露的原语。

### 7.1 Sampling：server 可以借用 client 的模型能力

Sampling 允许 server 请求 client 侧 AI 应用代为完成一次模型采样。其意义在于：server 作者不必自己持有模型 SDK 或 API key，却仍然能在必要时借用 host 的模型能力。模型访问的最终控制权仍留在 client 一侧。

### 7.2 Elicitation：把“向用户追问”也协议化

Elicitation 允许 server 在执行过程中结构化地向用户请求额外信息。稳定规范明确规定：

- server 不得用 elicitation 请求敏感信息；
- client 应让用户清楚知道是谁在请求信息；
- 用户应能审阅、修改、拒绝或取消。

这说明 MCP 不只是想标准化工具调用，它还在标准化 agent 运行时里的“追问、确认、补全”流程。

### 7.3 Roots：边界必须被显式暴露

Roots 让 client 向 server 显式声明可访问的文件系统根目录。这背后体现的是一种非常成熟的系统观：server 不应默认看到整个本地环境，边界必须由 host 主动暴露。

## 8. Local vs Remote：同样叫 MCP server，安全语义却完全不同

### 8.1 Local server 更像“受限的本地扩展”

本地 MCP server 常通过 `STDIO` 与 host 通信。优点是延迟低、靠近本地文件与开发环境、网络暴露面小。对代码仓库、文件系统、IDE 集成这类能力，本地 server 往往是自然形态。

但本地不等于天然安全。真正的安全来自最小权限和清晰边界，而不是“反正跑在我机器上”。

### 8.2 Remote server 更像“能力型基础设施”

远程 MCP server 更适合团队共享服务、企业 SaaS、集中权限治理和跨设备访问。它的优势是治理与复用，代价则是鉴权、租户隔离、token 验证、审计和攻击面都必须被系统性处理。

所以 MCP 的核心分界往往不是“本地还是远程”，而是：你接入的是个人扩展，还是一个会代表用户访问敏感资源的系统。

## 9. 授权为什么会成为主战场

MCP 的授权教程把远程授权流程写得很清楚：

1. client 首次连接受保护 server 时，server 返回 `401 Unauthorized`；
2. `WWW-Authenticate` 头里携带 `resource_metadata`，指向 Protected Resource Metadata；
3. client 发现授权服务器、支持的 scopes 等信息；
4. 整个授权流程遵循 OAuth 2.1，并使用 authorization code with PKCE；
5. server 侧需要验证 token、audience 与 scope。

这个流程最值得深想的地方，不是它用了哪套标准，而是：MCP 把“谁能访问什么能力”正式提升成 transport-level concern，而不是散落在每个工具实现里的临时判断。

官方教程也明确区分了本地与远程场景：

- 对 `STDIO` local servers，可以用环境变量或嵌入式第三方凭证；
- 对 remote servers，OAuth 2.1 + PKCE 是推荐正途。

这背后的工程哲学很明确：本地场景更看重开发便利与最小暴露面，远程场景则必须把身份、作用域、租户与审计当成系统级问题。

## 10. 一个案例：为知识库协作助手设计 MCP 接口时，为什么控制权划分比“能不能调起来”更重要

假设你在做一个知识库协作助手，它既能读资料，又能修改文档，还能触发预定义工作流。

### 10.1 文档目录、页面内容、数据库 schema

这些更适合作为 resources 暴露。因为它们本质上是上下文材料，应用应决定什么时候展示、检索、筛选、注入。

### 10.2 创建页面、更新数据库、发送通知

这些更适合作为 tools。因为它们会改动外部世界，而且应当保留 human-in-the-loop 的审批空间。

### 10.3 “起草周报”“整理会议纪要”“生成研究摘要”

这些更适合作为 prompts。因为用户需要显式地以某种任务形状来启动流程，而不是让模型在后台自己猜该不该触发。

### 10.4 缺少某个结构化参数

如果 server 需要额外的非敏感字段才能继续，例如周报时间范围、目标项目名，这时 elicitation 比自由文本追问更可控，因为它把字段、来源和确认动作都显式化了。

这个案例真正想说明的是：MCP 设计得好不好，首先取决于控制权有没有分清，而不是接口数量够不够多。

## 11. 最常见的几个误区

### 11.1 把所有东西都做成 tool

如果文档、目录树、只读 schema、检索结果全都做成 tool，模型会承担过多无意义选择。很多东西本来就更像 resource。

### 11.2 以为有了 MCP 就天然安全

协议帮你把能力接入标准化了，但不会自动替你做好最小权限、scope 设计、token 验证、密钥管理和日志脱敏。MCP 只是让你有机会把这些问题设计对。

### 11.3 忽视结果通道的设计

如果 tool 只返回一大段自由文本，上层系统就很难继续做结构化处理。很多情况下，你真正需要的是 `structuredContent`、`outputSchema` 或 `resource_link`。

### 11.4 在本地 / 远程之间使用错误的默认值

个人开发流程常常更适合本地 server；组织级 SaaS 和企业数据更适合远程 server。用错部署模型，往往很快就会在安全和可维护性上付出代价。

### 11.5 忽视用户可见性

工具是否暴露给模型、何时被调用、何时需要审批，这些都应当是用户可见的。一个看不见、关不掉、审计不了的工具层，长期一定会制造不信任。

## 12. 对 agent builder 来说，MCP 的真正位置是什么

如果把 agent 看成“模型 + 外部能力 + 环境反馈”，MCP 并不直接替代其中任何一层。它更像一副接插件骨架：

- 让外部能力不必绑死在单个应用内部；
- 让上下文暴露和动作调用不必混在 prompt 里；
- 让授权、追问、结果结构开始像软件架构，而不是像 prompt 黑魔法。

学到最后你会发现，MCP 最珍贵的不是语法，而是一种更成熟的系统观：

- 不是所有能力都该由模型控制；
- 不是所有信息都该以自由文本进入上下文；
- 不是所有写操作都该默认静默执行；
- 不是所有授权都该在后台偷偷完成。

## 13. 练习题

1. 一个只读知识库接口为什么更适合做 resource，而不是 tool？
2. 当一个 tool 的结果还需要被上层系统继续消费时，为什么 `structuredContent` 和 `outputSchema` 很重要？
3. 远程 MCP server 为何必须把授权当成系统级问题，而不是每个工具自己做一次 token 判断？

## 14. 一句话总结

如果说 function calling 解决的是“模型怎样触发某个动作”，那么 MCP 解决的是“一个 AI 应用如何长期、系统化、可治理地接入外部能力世界”。

## 来源

- Model Context Protocol homepage  
  https://modelcontextprotocol.io/
- MCP architecture overview  
  https://modelcontextprotocol.io/docs/learn/architecture
- MCP server concepts  
  https://modelcontextprotocol.io/docs/learn/server-concepts
- MCP specification overview, version `2025-06-18`  
  https://modelcontextprotocol.io/specification/2025-06-18
- MCP lifecycle specification, version `2025-06-18`  
  https://modelcontextprotocol.io/specification/2025-06-18/basic/lifecycle
- MCP tools specification, version `2025-06-18`  
  https://modelcontextprotocol.io/specification/2025-06-18/server/tools
- MCP resources specification, version `2025-06-18`  
  https://modelcontextprotocol.io/specification/2025-06-18/server/resources
- MCP prompts specification, version `2025-06-18`  
  https://modelcontextprotocol.io/specification/2025-06-18/server/prompts
- MCP elicitation specification, version `2025-06-18`  
  https://modelcontextprotocol.io/specification/2025-06-18/client/elicitation
- MCP roots specification, version `2025-06-18`  
  https://modelcontextprotocol.io/specification/2025-06-18/client/roots
- MCP authorization tutorial  
  https://modelcontextprotocol.io/docs/tutorials/security/authorization
- OpenAI, *Safety in building agents*  
  https://developers.openai.com/api/docs/guides/agent-builder-safety
