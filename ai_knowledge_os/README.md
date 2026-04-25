# AI Knowledge OS

这是一套面向长期学习、研究、实操与技术判断的 AI 知识操作系统源文件。

目标不是堆资料，而是把资料压缩成可维护、可迁移、可行动的知识结构：

- 本地目录负责保存稳定源文件、模板、操作规则。
- Notion 负责数据库化呈现、视图切换、筛选与回顾。
- 飞书 Wiki 负责层级导航、长文阅读、中文知识入口。

当前核心章节已经不是单纯提纲，而是逐步补成了更像书稿的长文层：

- 地图页会解释问题压力、结构关系、误判和进入路径。
- 方法论页会加入案例、诊断表、练习题、模板和回写规则。
- 概念节点会讲清机制、边界、选型逻辑和工程判断。
- 雷达与 sourcebook 会帮助区分结构性变化、资料读法和路线判断。

## 目录

- [AGENTS.md](AGENTS.md)
  未来让 Codex 继续扩建本知识库时应遵守的规则。
- [00_system/knowledge_os_blueprint.md](00_system/knowledge_os_blueprint.md)
  整体架构、模块划分、Notion 与飞书分工。
- [00_system/knowledge_ingestion_protocol.md](00_system/knowledge_ingestion_protocol.md)
  论文、项目、课程与工具的固定摄入流程。
- [00_system/chatgpt_pro_workflows.md](00_system/chatgpt_pro_workflows.md)
  ChatGPT / Codex 的学习、研究与执行工作流。
- [01_maps/ai_domain_map.md](01_maps/ai_domain_map.md)
  AI 全景知识树与顶层模块。
- [01_maps/agents_benchmarks_map.md](01_maps/agents_benchmarks_map.md)
  Agents、多智能体与 benchmark 的结构地图。
- [01_maps/inference_systems_map.md](01_maps/inference_systems_map.md)
  推理系统、GPU 优化与 serving 主线。
- [01_maps/multimodal_production_map.md](01_maps/multimodal_production_map.md)
  图像、视频、音频与创作流程地图。
- [02_sources/official_learning_stack.md](02_sources/official_learning_stack.md)
  以官方资料为主的学习源索引。
- [02_sources/agents_systems_benchmark_sourcebook.md](02_sources/agents_systems_benchmark_sourcebook.md)
  Agent、systems 与 benchmark 的官方资料索引。
- [03_execution/project_lab_roadmap.md](03_execution/project_lab_roadmap.md)
  实操项目和阶段推进路线。
- [03_execution/agent_workflow_eval_playbook.md](03_execution/agent_workflow_eval_playbook.md)
  Agent workflow、trace 与 eval 的执行指南。
- [03_execution/benchmark_selection_guide.md](03_execution/benchmark_selection_guide.md)
  Benchmark 对比与选型方法。
- [03_execution/creative_production_workflow.md](03_execution/creative_production_workflow.md)
  AI 视频 / 音乐 production workflow。
- [04_radar/frontier_radar.md](04_radar/frontier_radar.md)
  用于每周/每月更新前沿的雷达框架。
- [04_radar/frontier_signals_2026_q2.md](04_radar/frontier_signals_2026_q2.md)
  本轮值得重点追踪的结构性变化。
- [05_notion/notion_ai_lake_playbook.md](05_notion/notion_ai_lake_playbook.md)
  Notion 数据库的结构与视图使用建议。
- [05_notion/seed_cards.md](05_notion/seed_cards.md)
  首批适合写入 Notion 数据库的知识卡种子。
- [06_nodes/context_engineering_long_context.md](06_nodes/context_engineering_long_context.md)
  长上下文、记忆设计与 context engineering 的系统化拆解。
- [06_nodes/post_training_primitives.md](06_nodes/post_training_primitives.md)
  SFT、DPO、RFT、GRPO 这组后训练原语的边界与选型。
- [06_nodes/inference_primitives.md](06_nodes/inference_primitives.md)
  KV cache、PagedAttention、prefix caching、speculative decoding 的推理原语。
- [06_nodes/mcp_tool_protocols.md](06_nodes/mcp_tool_protocols.md)
  MCP、工具协议、资源协议和远程接入的结构化理解。

## 维护原则

- 稳定知识进蓝图、地图、协议页。
- 快速变化知识进雷达页与数据库卡片。
- 每个新增资料都要回写到至少一个知识节点，而不是只留在聊天记录里。
- 优先保留可复用框架、关键问题、项目路径和权威来源。
- 如果一篇文档只有定义和条目，没有案例、误判、边界或可执行方法，它仍然算半成品。
