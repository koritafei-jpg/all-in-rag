# 第一节：AI Memory 最新论文调研报告

## 题目

**从检索增强到可持续记忆：面向 LLM Agent 的 AI Memory 技术架构调研**

## 摘要

大语言模型（Large Language Models, LLMs）在推理、工具调用和交互生成方面取得了显著进展，但固定上下文窗口、静态参数知识和跨会话状态缺失仍限制其成为长期可用的智能体。AI Memory 的研究目标是让 AI 系统能够在多轮、多会话、多模态和开放环境中持续地写入、组织、检索、更新、遗忘并使用信息。本报告围绕 2023 至 2026 年间代表性研究，系统梳理 AI Memory 的技术架构演进：从基于自然语言经验流的 Agent 记忆、基于外部存储的长期会话记忆、操作系统式虚拟上下文、图谱化长期记忆、参数/潜空间自更新记忆，到 2025 年后面向生产部署的多信号、多智能体、多模态记忆层。报告逐篇分析 Generative Agents、Reflexion、MemoryBank、LongMem、MemGPT、LoCoMo、MEMORYLLM、HippoRAG、MemoRAG、LongMemEval、Zep、A-MEM、Mem0、MIRIX 以及最新综述论文的技术架构与成功经验，并总结未来方向：记忆操作自动化、因果与时间一致性、多模态具身记忆、强化学习驱动的记忆管理、可信遗忘、隐私治理和标准化评测。

**关键词**：AI Memory；LLM Agent；长期记忆；检索增强生成；知识图谱；多模态记忆；Agentic Memory；记忆评测

## 1. 引言

传统 RAG 主要解决“给定当前问题，从静态知识库中检索相关文档”的问题，而 AI Memory 进一步关注“系统如何在交互过程中持续形成、组织和使用自身经历”。两者的差异可以概括为：

| 维度 | 传统 RAG | AI Memory |
| :--- | :--- | :--- |
| 信息来源 | 预先构建的静态文档库 | 会话、行动轨迹、用户偏好、环境观察、工具结果、外部知识 |
| 写入路径 | 多为离线 ETL | 在线、持续、带过滤和合并策略 |
| 组织方式 | Chunk + 向量索引为主 | 事件、事实、偏好、计划、反思、知识图谱、潜变量、参数池 |
| 时间建模 | 通常弱时间感知 | 强调跨会话、事实更新、时效、遗忘和版本 |
| 使用目标 | 回答当前问题 | 个性化、长期一致性、自我改进、任务迁移、环境适应 |
| 关键风险 | 检索不准、上下文噪声 | 错记、过度记忆、隐私泄露、冲突事实、记忆污染 |

因此，AI Memory 不是单一模块，而是一个围绕 **write-manage-read** 闭环展开的系统能力：写入时决定“什么值得记住”，管理时决定“如何合并、更新、链接、压缩、遗忘”，读取时决定“在当前目标下召回什么、如何排序、如何注入上下文”。最新综述将 Agent 记忆与普通 LLM 记忆、RAG 和上下文工程区分开来，并从形态、功能与动态三个维度给出更细分类：token-level、parametric、latent memory；factual、experiential、working memory；以及记忆形成、演化与检索过程[^survey_agents_memory]。

## 2. 调研方法与选文范围

本报告选择论文遵循三条标准：

1. **架构代表性**：覆盖外部存储、图谱、潜空间/参数、上下文压缩、多智能体记忆等不同技术路线。
2. **研究影响力与实践价值**：优先选择公开论文、开源实现或被后续系统广泛引用的工作。
3. **最新性**：重点纳入 2024-2026 年论文，并保留 2023 年奠基性 Agent 记忆工作作为技术脉络。

选取的核心论文如下。

| 年份 | 论文/系统 | 主要贡献 |
| :--- | :--- | :--- |
| 2023 | Reflexion | 通过语言化反思形成经验记忆，无需更新模型参数 |
| 2023 | Generative Agents | 观察-记忆流-反思-规划架构，奠定 Agent 经验记忆范式 |
| 2023/2024 | MemoryBank | 面向长期陪伴的外部记忆库，引入遗忘曲线 |
| 2023 | LongMem | 冻结 Backbone + SideNet + KV 记忆库，实现可训练长期记忆 |
| 2023 | MemGPT | 操作系统式虚拟上下文管理，分层内外存 |
| 2024 | LoCoMo | 长期会话记忆评测基准 |
| 2024 | MEMORYLLM | 潜空间固定记忆池，自更新参数化记忆 |
| 2024 | HippoRAG | 海马体启发的图谱化长期记忆检索 |
| 2024/2025 | MemoRAG | 全局记忆增强检索，解决隐式需求和长上下文任务 |
| 2025 | LongMemEval | 面向聊天助手长期交互记忆的综合基准 |
| 2025 | Zep | 时序知识图谱 Agent Memory 生产架构 |
| 2025 | A-MEM | Zettelkasten 启发的 Agentic Memory 动态组织 |
| 2025 | Mem0 | 生产级可扩展长期记忆层，兼顾准确率、延迟和成本 |
| 2025 | MIRIX | 多智能体、多类型、多模态记忆系统 |
| 2025/2026 | 最新综述 | 提出 AI Memory/Agent Memory 分类框架与未来问题 |

## 3. AI Memory 技术分类框架

### 3.1 按记忆承载形态分类

1. **上下文内记忆（context-resident memory）**  
   将最近交互、摘要、计划或反思直接放入 prompt/context window。优点是实现简单、推理路径透明；缺点是受上下文长度和噪声影响。

2. **外部非参数记忆（external non-parametric memory）**  
   以向量数据库、全文索引、键值库、关系库或图数据库保存记忆。MemoryBank、MemGPT、Mem0、Zep、MIRIX 多属于此类。优点是可更新、可审计、可删除；难点是写入过滤、冲突合并、检索和排序。

3. **图谱记忆（graph memory）**  
   将事件、实体、关系和时间组织为知识图谱或事件图。HippoRAG、Zep、Mem0g 和 MIRIX 均体现该趋势。优点是支持多跳、时间和关系推理；难点是抽取质量、图谱更新成本与错误传播。

4. **潜空间/参数记忆（latent or parametric memory）**  
   在模型内部或附加网络中维护可更新记忆参数，例如 MEMORYLLM 的固定记忆池、LongMem 的 SideNet 和 KV 记忆库。优点是与模型推理深度融合；难点是可解释性、删除权、灾难性遗忘和部署成本。

5. **多智能体记忆（multi-agent memory）**  
   使用多个专职 Agent 管理不同类型记忆，例如 MIRIX 的 Core、Episodic、Semantic、Procedural、Resource、Knowledge Vault 六类记忆。优点是职责分离、扩展性强；缺点是系统复杂度和协调成本较高。

### 3.2 按功能分类

| 功能类型 | 说明 | 代表系统 |
| :--- | :--- | :--- |
| 事实记忆 | 用户信息、世界知识、业务事实 | Mem0、Zep、MIRIX |
| 情景记忆 | 交互事件、观察、行动轨迹 | Generative Agents、MemoryBank、MIRIX |
| 反思记忆 | 从失败、反馈或经历中总结出的经验 | Reflexion、Generative Agents |
| 工作记忆 | 当前任务状态、计划、临时目标 | MemGPT、MIRIX |
| 程序记忆 | 工具使用方式、任务流程、偏好策略 | MIRIX、Agentic 系统 |
| 结构记忆 | 实体关系、事件图、社区摘要 | HippoRAG、Zep、A-MEM |

### 3.3 按记忆生命周期分类

AI Memory 的生命周期可表示为：

```text
Perception/Interaction
        |
        v
Memory Write: 抽取、过滤、去重、事实化、重要性评分
        |
        v
Memory Manage: 合并、链接、更新、冲突检测、遗忘、摘要、时间建模
        |
        v
Memory Read: 查询改写、多路检索、图遍历、重排、上下文构造
        |
        v
Reasoning/Action: 生成答案、执行工具、形成新观察
```

## 4. 代表性论文技术架构分析

### 4.1 Reflexion: Language Agents with Verbal Reinforcement Learning

**核心问题**：传统强化学习需要大量样本和参数更新，语言 Agent 在代码、推理、游戏等任务中很难快速从失败中学习。

**技术架构**：

```text
任务执行 Agent
    -> 环境反馈/测试结果/自评反馈
    -> 反思生成器将反馈转化为自然语言经验
    -> Episodic Memory Buffer 存储反思
    -> 下一轮任务执行时将相关反思注入上下文
```

Reflexion 的关键思想是把奖励信号转化为“语义梯度”。它不更新模型参数，而是让 Agent 用自然语言描述失败原因、改进策略和注意事项，再把这些反思存入 episodic memory。后续尝试中，Agent 读取该记忆作为行动提示。

**成功总结**：

- 在 HumanEval 上报告 91% pass@1，超过当时 GPT-4 基线的 80%[^reflexion]。
- 证明自然语言记忆可以承担轻量级策略更新功能。
- 反思文本可解释，便于调试和人工审查。

**局限**：

- 依赖模型自我评估能力，错误反思会污染记忆。
- 适合短期任务迭代，跨长期、多源事实管理能力较弱。

### 4.2 Generative Agents: Interactive Simulacra of Human Behavior

**核心问题**：如何让 LLM Agent 在模拟世界中表现出长期一致、可信的人类行为。

**技术架构**：

```text
环境观察
  -> Memory Stream: 自然语言记录每条观察
  -> Retrieval: recency + relevance + importance 评分召回
  -> Reflection: 达到重要性阈值后生成高层洞察
  -> Planning: 生成日计划、小时计划、细粒度行动
  -> Action/Dialogue
```

Memory Stream 是一个按时间追加的自然语言经验日志。检索同时考虑最近性、语义相关性和重要性。反思模块周期性地从底层观察中抽象出高层洞察，并再次写回记忆流，形成“观察-反思”层次结构。

**成功总结**：

- 25 个 Agent 在沙盒小镇中可以形成可信个体行为和涌现式社交行为，例如自发传播情人节派对邀请[^generative_agents]。
- 消融实验显示观察、规划、反思均对可信度有显著贡献。
- 奠定了后续 Agent Memory 的经典范式：经验记录、重要性评分、反思合成、动态检索。

**局限**：

- 主要依赖 prompt 和 LLM 评分，成本较高。
- 记忆冲突、错误反思和长期规模化索引没有系统解决。

### 4.3 MemoryBank: Enhancing Large Language Models with Long-Term Memory

**核心问题**：长期陪伴、心理咨询、秘书等场景需要模型持续记住用户经历和人格特征。

**技术架构**：

```text
多轮对话
  -> 会话事件抽取与摘要
  -> 用户画像/全局摘要更新
  -> 向量化索引与相关记忆召回
  -> Ebbinghaus 遗忘曲线控制记忆强度
  -> 个性化响应生成
```

MemoryBank 将会话记忆组织为事件摘要和用户画像，并用向量检索召回相关记忆。其特色是引入艾宾浩斯遗忘曲线，根据时间间隔和记忆重要性衰减记忆强度；被再次召回的记忆会得到强化。

**成功总结**：

- 在 SiliconFriend 长期陪伴场景中展示了记忆召回、用户人格理解和更具共情的响应[^memorybank]。
- 将“遗忘”引入 LLM 长期记忆管理，避免无限保留低价值信息。
- 可适配闭源模型和开源模型，工程门槛较低。

**局限**：

- 遗忘曲线是启发式规则，未必适配所有业务场景。
- 对事实冲突和隐私删除的支持有限。

### 4.4 LongMem: Augmenting Language Models with Long-Term Memory

**核心问题**：固定长度 Transformer 难以利用长历史上下文，直接扩展上下文成本高。

**技术架构**：

```text
冻结 Backbone LLM
    -> 将历史片段编码为 attention key-value
    -> Cache Memory Bank 保存历史 KV
SideNet
    -> 生成当前查询
    -> 从 Memory Bank 检索相关 KV
    -> Joint Attention 融合记忆与当前隐状态
```

LongMem 使用解耦设计：原始 Backbone LLM 冻结并作为记忆编码器，适配训练一个残差 SideNet 作为记忆检索器和读取器。历史上下文以 KV 形式缓存到非可微记忆库，当前输入通过 SideNet 检索并融合历史 KV。

**成功总结**：

- 支持约 65K token 长记忆，用于长上下文建模和 memory-augmented in-context learning[^longmem]。
- 冻结 Backbone 降低训练成本，SideNet 解决记忆陈旧问题。
- 将“长期记忆”从纯检索系统推进到模型内部注意力融合层面。

**局限**：

- 需要训练和定制模型结构，难以直接套用闭源 API。
- KV 记忆可解释性和精确删除能力弱于外部数据库。

### 4.5 MemGPT: Towards LLMs as Operating Systems

**核心问题**：上下文窗口有限，但真实任务需要访问远超窗口大小的对话和文档。

**技术架构**：

```text
Main Context (类似 RAM)
  - System Instructions
  - Working Context
  - FIFO Queue

External Context (类似 Disk)
  - Archival Storage
  - Recall Storage

LLM Processor
  -> 通过函数调用执行 search/store/update/summarize
  -> Queue Manager 触发警告、驱逐和分页
```

MemGPT 借鉴操作系统虚拟内存，将上下文分为主上下文和外部上下文。主上下文是模型当前可见的 token 空间，外部上下文是超出窗口的持久存储。LLM 通过函数调用主动管理自己的记忆，例如把重要信息写入外部存储、检索相关历史、更新工作记忆。

**成功总结**：

- 提出“virtual context management”概念，为固定上下文模型提供近似无限上下文的使用体验[^memgpt]。
- 在文档分析和多会话聊天中验证了分层记忆管理的有效性。
- 将 OS 中的分页、层级存储、控制流中断引入 LLM 系统设计。

**局限**：

- 记忆操作由 LLM 自主决策，稳定性依赖 prompt 和工具调用质量。
- 多次分页检索会带来延迟，复杂任务中可能出现策略漂移。

### 4.6 LoCoMo: Evaluating Very Long-Term Conversational Memory of LLM Agents

**核心问题**：已有长期对话评测通常只有少量会话，难以衡量真正跨会话、时间和因果记忆能力。

**技术架构/评测设计**：

```text
Persona + Temporal Event Graph
  -> LLM Agent 生成长对话
  -> 人工校验和编辑
  -> LoCoMo 数据集
      - QA
      - Event Summarization
      - Multimodal Dialogue Generation
```

LoCoMo 使用机器-人工管线生成并校验长期对话。ACL 版本包含 10 个高质量长对话，每个约 600 turns、16K tokens、最多 32 个 session，并设计 QA、事件摘要和多模态对话生成任务[^locomo]。

**成功总结**：

- 指出长上下文模型和 RAG 在长期对话中仍显著落后于人类，尤其是时间和因果动态。
- 成为 Mem0、MIRIX 等后续生产记忆系统常用评测基准。
- 将 AI Memory 评测从静态 recall 推向跨会话任务。

**局限**：

- 数据规模较小，主题和真实用户复杂性仍有限。
- 主要评测显式事实和事件，隐式约束/价值观记忆覆盖不足。

### 4.7 MEMORYLLM: Towards Self-Updatable Large Language Models

**核心问题**：部署后的 LLM 参数通常静态，外部 RAG 又无法真正改变模型内部知识。

**技术架构**：

```text
Transformer Backbone
  + Latent Memory Pool (固定大小、可自更新参数)
      -> 注入文本知识
      -> 更新记忆池
      -> 后续推理直接读取潜空间记忆
```

MEMORYLLM 在 Transformer 潜空间中加入固定大小记忆池，使模型可以通过文本知识进行自更新。该记忆池类似可写参数区域，用于吸收新知识并保留较早注入的信息。

**成功总结**：

- 在模型编辑、长期信息保留和长上下文任务中验证自更新能力[^memoryllm]。
- 报告近百万次记忆更新后仍未出现明显性能退化。
- 代表“参数/潜空间记忆”路线，为未来可持续更新 LLM 提供方向。

**局限**：

- 删除、审计和隐私合规难度较高。
- 需要定制模型，难以快速应用于 API 型 Agent。

### 4.8 HippoRAG: Neurobiologically Inspired Long-Term Memory for Large Language Models

**核心问题**：传统 RAG 难以整合分散在多文档中的多跳知识，迭代检索成本高。

**技术架构**：

```text
文档语料
  -> LLM/OpenIE 抽取实体与事实三元组
  -> 构建知识图谱
  -> 查询实体识别
  -> Personalized PageRank 在图上扩散
  -> 召回相关段落/事实
  -> LLM 生成答案
```

HippoRAG 受海马体索引理论启发，用知识图谱模拟新皮层知识结构，用 Personalized PageRank 模拟海马体基于线索的关联检索。它将多跳检索转化为图上的单步扩散。

**成功总结**：

- 在多跳 QA 上较现有 RAG 方法最高提升约 20%[^hipporag]。
- 单步检索达到或超过 IRCoT 等迭代检索效果，同时成本低 10-30 倍、速度快 6-13 倍。
- 证明图谱化记忆适合复杂关系和跨文档知识整合。

**局限**：

- 依赖信息抽取质量，抽取错误会影响图结构。
- 图构建和更新成本高于普通向量索引。

### 4.9 MemoRAG: Boosting Long Context Processing with Global Memory-Enhanced Retrieval Augmentation

**核心问题**：传统 RAG 假设查询明确、知识结构良好，但长上下文任务中用户需求可能隐式、模糊或需要全局理解。

**技术架构**：

```text
长上下文/数据库
  -> 轻量长程模型建立 Global Memory
  -> 面向任务生成 draft answer / clues
  -> 检索器根据 clues 定位证据
  -> 强表达模型基于证据生成最终答案
  -> RLGF 强化记忆模块的 clue 生成能力
```

MemoRAG 采用双系统架构：轻量但长程的模型负责形成全局记忆和生成检索线索；昂贵但表达力强的模型负责最终答案。其记忆模块可用 KV compression 实现，并通过 Generation quality Feedback 强化。

**成功总结**：

- 在多种长上下文任务中优于标准 RAG，尤其适合传统 RAG 难以处理的复杂任务[^memorag]。
- 将“先记住全局，再生成检索线索”作为新型 RAG 数据接口。
- 提升了对隐式信息需求和非结构化知识的处理能力。

**局限**：

- 双模型架构增加系统复杂度。
- draft clue 质量决定检索上限，错误线索可能造成偏航。

### 4.10 LongMemEval: Benchmarking Chat Assistants on Long-Term Interactive Memory

**核心问题**：聊天助手的长期记忆能力缺少覆盖多能力、可扩展历史长度的评测。

**技术架构/评测设计**：

```text
可扩展聊天历史
  -> 500 个精标问题
  -> 五类能力评测：
      1. 信息抽取
      2. 多会话推理
      3. 时间推理
      4. 知识更新
      5. 拒答/不可答
  -> 索引-检索-阅读三阶段设计分析
```

LongMemEval 包含 LongMemEval_S（约 115K tokens）和 LongMemEval_M（约 500 sessions）等设置，强调长期互动中的检索、更新、时间和拒答能力[^longmemeval]。

**成功总结**：

- 商业聊天助手和长上下文 LLM 在持续交互历史上出现约 30% 准确率下降。
- 提出 session decomposition、fact-augmented key expansion、time-aware query expansion 等优化方向。
- 为工程系统提供了索引、检索、阅读三阶段分析框架。

**局限**：

- 仍以文本聊天历史为主，对多模态和真实工具行动覆盖有限。
- 评测结果受问题构造和自动指标影响。

### 4.11 Zep: A Temporal Knowledge Graph Architecture for Agent Memory

**核心问题**：企业 Agent 需要动态整合会话和业务数据，并保持事实时间、历史版本和低延迟检索。

**技术架构**：

```text
会话数据 + 结构化业务数据
  -> Graphiti 时序知识图谱
      - Episodic Memory: 原始事件/对话
      - Semantic Memory: 实体、事实、关系
      - Temporal Edges: 事实有效时间与系统记录时间
      - Communities: 社区摘要
  -> Hybrid Search: 向量 + BM25 + 图遍历
  -> Reranking: RRF/MMR/距离/LLM 重排
  -> Context Constructor
```

Zep 的核心是 Graphiti：一个动态、双时间（bi-temporal）知识图谱引擎。它记录事实在现实世界中的有效时间，以及系统何时知道该事实，支持历史查询、事实失效和来源追踪。

**成功总结**：

- 在 DMR 上超过 MemGPT（94.8% vs 93.4%），在 LongMemEval 上报告最高 18.5% 准确率提升和 90% 延迟降低[^zep]。
- 将生产系统关注的准确率、延迟、可追溯和可更新统一到图谱记忆架构中。
- 强调 enterprise memory 中静态文档 RAG 不足，需要持续融合业务事实。

**局限**：

- 图谱抽取和维护复杂，对领域 schema 和抽取模型质量敏感。
- 商业系统与论文评测之间仍需更多独立复现。

### 4.12 A-MEM: Agentic Memory for LLM Agents

**核心问题**：现有记忆系统多为固定结构和固定操作，缺乏任务自适应组织能力。

**技术架构**：

```text
新记忆输入
  -> 生成 Zettelkasten 风格原子笔记
      - 上下文描述
      - 关键词
      - 标签
      - embedding
  -> 检索历史记忆
  -> 建立语义/属性链接
  -> 触发历史记忆属性和上下文演化
  -> 形成动态知识网络
```

A-MEM 借鉴 Zettelkasten 卡片盒笔记法，将每条记忆转化为结构化笔记，并由 Agent 自主决定链接、索引和演化方式。与静态图数据库不同，它强调“记忆操作本身也由 Agent 进行”。

**成功总结**：

- 在六个 foundation model 上相对现有 SOTA 取得更好长期任务表现[^amem]。
- 把记忆管理从固定 pipeline 推向 agentic、可自适应组织。
- 适合复杂任务中不断重写、扩充和重解释历史经验的场景。

**局限**：

- Agentic 管理带来不可预测性，评估和调试难度增加。
- 记忆演化可能改写历史语义，需要版本控制和审计机制。

### 4.13 Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory

**核心问题**：真实生产场景要求长期记忆同时满足准确率、低延迟、低 token 成本和可扩展部署。

**技术架构**：

```text
用户/Agent 对话
  -> 动态抽取 salient memories
  -> 合并、更新、去重、存储
  -> 多信号检索相关记忆
  -> 可选 Graph Memory (Mem0g) 表达实体关系
  -> 注入上下文生成响应
```

Mem0 的核心是可扩展记忆层：从对话中抽取重要事实，动态合并和检索，并提供图记忆变体 Mem0g 用于复杂关系建模。其工程目标明确：在长期对话基准上提高准确率，同时降低全上下文方法的延迟和成本。

**成功总结**：

- 在 LoCoMo 上相对 OpenAI Memory 的 LLM-as-a-Judge 指标提升 26%，Mem0g 在整体分数上较基础版进一步提升约 2%[^mem0]。
- 相比 full-context 方法，p95 延迟降低 91%，token 成本节省超过 90%。
- 体现 AI Memory 从研究原型走向生产基础设施的趋势。

**局限**：

- 抽取、更新和删除策略需要在不同业务中重新校准。
- 用户隐私、错误记忆纠正和多租户隔离是生产部署关键问题。

### 4.14 MIRIX: Multi-Agent Memory System for LLM-Based Agents

**核心问题**：单一平面记忆难以覆盖真实个人助手中的文本、视觉、程序、资源和隐私需求。

**技术架构**：

```text
Meta Memory Manager
  -> Core Memory Agent: 人设、用户核心偏好
  -> Episodic Memory Agent: 事件和经历
  -> Semantic Memory Agent: 抽象事实和概念
  -> Procedural Memory Agent: 流程和技能
  -> Resource Memory Agent: 文件、截图、外部资源
  -> Knowledge Vault Agent: 高价值知识和隐私信息
  -> 多模态输入：文本、图像、语音、屏幕活动
```

MIRIX 将记忆类型拆分为六类，并由多个专职 Agent 管理，Meta Memory Manager 负责路由、协调和检索。它特别强调多模态个人助手场景，例如持续监控屏幕、压缩高分辨率视觉信息并构建本地私有记忆。

**成功总结**：

- 在 ScreenshotVQA 上比 RAG baseline 高 35% 准确率，同时减少 99.9% 存储需求；在 LoCoMo 上达到 85.4% SOTA 表现[^mirix]。
- 将 AI Memory 扩展到真实多模态生活流，而不仅是文本会话。
- 通过记忆类型分工提升系统可维护性和扩展性。

**局限**：

- 多 Agent 协调成本高，错误路由会导致召回失败。
- 持续屏幕记忆带来极高隐私风险，需要本地化、权限和数据最小化设计。

### 4.15 最新综述：From Human Memory to AI Memory 与 Memory in the Age of AI Agents

2025 年的《From Human Memory to AI Memory》从人类记忆出发，按对象、形式和时间三个维度组织 LLM 时代记忆机制，强调 AI Memory 与人类感觉记忆、短期记忆、长期记忆、情景记忆、语义记忆之间的映射关系[^human_to_ai_memory]。

2025/2026 年的《Memory in the Age of AI Agents》进一步指出，传统“短期/长期”二分法不足以描述 Agent 记忆，提出从形态（token-level、parametric、latent）、功能（factual、experiential、working）与动态（形成、演化、检索）理解 Agent Memory，并总结记忆自动化、强化学习、多模态、多智能体和可信问题[^survey_agents_memory]。

这些综述的共同贡献是把 AI Memory 从零散模块提升为 Agent 架构的一等公民（first-class primitive）：记忆不再只是 RAG 附属品，而是决定 Agent 个性化、持续学习、长期规划和安全治理的基础设施。

## 5. 横向对比：不同架构路线的适用场景

| 路线 | 代表论文 | 优势 | 风险/成本 | 适合场景 |
| :--- | :--- | :--- | :--- | :--- |
| 自然语言经验流 | Generative Agents、Reflexion | 简单、可解释、快速迭代 | 容易冗余和污染 | 游戏 NPC、任务反思、原型 Agent |
| 外部向量/文档记忆 | MemoryBank、Mem0 | 工程成熟、易删除、易部署 | 时间与关系建模弱 | 客服、个人助手、长期会话 |
| 操作系统式分层记忆 | MemGPT | 支持虚拟上下文和自主管理 | 延迟和策略稳定性问题 | 长文档分析、多会话 Agent |
| 图谱/时序图记忆 | HippoRAG、Zep、Mem0g | 多跳、关系、时间推理强 | 抽取和维护成本高 | 企业知识、复杂用户画像、事件追踪 |
| 潜空间/参数记忆 | LongMem、MEMORYLLM | 与模型推理深度融合 | 删除和审计困难 | 自研模型、离线训练、知识注入 |
| 全局记忆增强 RAG | MemoRAG | 支持隐式需求和长上下文 | 双模型系统复杂 | 长上下文问答、报告分析 |
| Agentic/Multi-Agent Memory | A-MEM、MIRIX | 自适应、多类型、多模态 | 协调、评估和隐私复杂 | 高级个人助手、具身/多模态 Agent |

## 6. 成功经验总结

### 6.1 记忆不是“保存全部历史”，而是“选择性压缩与结构化”

Mem0 的 token 成本节省、MemoryBank 的遗忘曲线、MemoRAG 的全局记忆、MIRIX 的视觉数据压缩都说明：长期记忆的核心不是无限存储，而是在写入时选择、抽象和压缩。优秀系统通常在 write path 做三件事：

1. 抽取稳定事实、偏好、事件和技能；
2. 删除或弱化低价值、重复、过期信息；
3. 保存可追溯证据，避免摘要失真。

### 6.2 图结构显著提升多跳和时间推理

HippoRAG、Zep、Mem0g 表明，当问题涉及“谁与谁相关”“事件如何演化”“某事实何时有效”时，单纯向量相似度不足。图谱通过实体、关系、时间边和社区摘要提供了更强的结构归纳偏置。

### 6.3 反思记忆能把失败转化为可复用经验

Reflexion 和 Generative Agents 证明，语言化反思可以作为轻量级学习机制。它不需要训练参数，却能在后续任务中提供明确的策略提示。对于代码 Agent、工具 Agent 和开放环境 Agent，反思记忆是低成本自我改进的重要手段。

### 6.4 分层架构比单一记忆库更可扩展

MemGPT 的 main/external context、MIRIX 的六类记忆、Zep 的 episodic/semantic graph 都体现分层思想。不同记忆具有不同读写频率、隐私等级、生命周期和检索方式，混在一个向量库中会导致召回噪声和维护困难。

### 6.5 评测正在从“能否回忆事实”走向“能否长期使用记忆”

LoCoMo 和 LongMemEval 将长期记忆评测扩展到多会话推理、时间推理、知识更新、拒答和多模态生成。最新趋势是评估 Agent 在长期任务中是否能用记忆作出正确决策，而不仅是回答“某条信息是什么”。

### 6.6 生产级记忆必须同时优化准确率、延迟、成本和治理

Mem0 和 Zep 的论文都强调低延迟、低 token 成本和部署可行性。实际系统中，记忆检索如果每次引入多轮 LLM 调用，即使准确率提升也难以落地。生产级 AI Memory 需要：

- 写入路径异步化和批处理；
- 多路召回与轻量重排；
- 时间和用户作用域过滤；
- 可删除、可导出、可解释的审计接口；
- 隐私隔离和加密存储。

## 7. 未来发展方向

### 7.1 记忆操作自动化：从规则 pipeline 到策略学习

当前许多系统仍使用启发式规则决定写入、合并和遗忘。未来需要学习型 memory policy，根据任务收益、成本、隐私风险和长期表现自动决定：

- 哪些内容进入长期记忆；
- 何时更新旧记忆而非新增；
- 何时遗忘、压缩或降权；
- 当前任务应召回多少、召回哪类记忆。

强化学习和离线轨迹学习可能成为关键方法，但需要解决奖励稀疏和错误记忆长期影响的问题。

### 7.2 因果和时间一致性记忆

长期 Agent 不仅要知道事实，还要知道事实的因果来源、有效时间和更新历史。未来架构应支持：

- bi-temporal facts：事实发生时间与系统知晓时间分离；
- contradiction resolution：检测并解决新旧事实冲突；
- provenance tracking：回答时可追溯到原始证据；
- causal retrieval：不是只检索相似内容，而是检索导致当前状态的关键事件链。

### 7.3 多模态与具身记忆

MIRIX、LoCoMo 的多模态任务表明，真实 Agent 会从屏幕、语音、图像、视频、传感器和工具日志中形成记忆。未来重点包括：

- 视觉事件压缩和长期索引；
- 跨模态实体对齐；
- 场景、位置、对象和行动的统一表示；
- 对高隐私多模态数据的本地优先处理。

### 7.4 从检索记忆到推理记忆

下一阶段的记忆系统应不仅返回“相关片段”，还应返回“可执行推理状态”。例如：

- 用户长期目标和当前约束；
- 历史失败模式和规避策略；
- 多步任务的未完成子目标；
- 可复用工具调用流程；
- 与当前问题相关的假设、证据和反证。

这要求记忆与规划器、工具执行器和验证器深度耦合。

### 7.5 可控遗忘与隐私治理

AI Memory 越强，隐私风险越高。未来系统必须内建：

- 用户可见和可编辑的记忆面板；
- 精确删除与“被遗忘权”；
- 敏感信息识别和默认不写入；
- 记忆访问权限、租户隔离和加密；
- 记忆使用日志和可审计解释。

可控遗忘会成为 AI Memory 与普通 RAG 的重要分水岭。

### 7.6 标准化基准与线上评估

现有基准仍难完全覆盖真实场景。未来评测需要同时衡量：

- 写入质量：是否记住该记住的内容；
- 检索质量：是否在正确时机召回正确记忆；
- 使用质量：模型是否真正基于记忆作出更好决策；
- 更新质量：新事实是否覆盖旧事实；
- 遗忘质量：过期和敏感信息是否被正确处理；
- 成本指标：延迟、token、存储、LLM 调用次数；
- 安全指标：隐私泄露、错误记忆放大、越权召回。

### 7.7 参数记忆与外部记忆融合

外部记忆易治理，参数/潜空间记忆推理效率高。未来可能出现混合架构：

```text
高频稳定知识 -> 参数/潜空间记忆
用户个性化事实 -> 外部可删除记忆
复杂关系和时间事件 -> 时序知识图谱
短期任务状态 -> 工作上下文
失败经验和策略 -> 反思/程序记忆
```

关键挑战是不同记忆层之间的一致性、迁移、删除和优先级仲裁。

## 8. 面向工程落地的参考架构

结合上述论文，一个生产级 AI Memory 系统可采用如下架构：

```text
Input Layer
  - 用户消息、Agent 行动、工具结果、文件、图像、屏幕事件

Memory Writer
  - 敏感信息过滤
  - 事件/事实/偏好/流程抽取
  - 重要性评分
  - 去重与冲突检测

Memory Stores
  - Working Memory: 当前任务状态
  - Episodic Store: 原始会话和事件
  - Semantic Store: 事实和用户画像
  - Temporal KG: 实体、关系、时间、来源
  - Reflection Store: 失败经验和策略
  - Resource Store: 文件、图片、截图、工具日志

Memory Manager
  - 合并、版本、遗忘、权限、加密、审计

Memory Reader
  - Query rewriting
  - BM25 + Vector + Entity + Graph 多路召回
  - 时间过滤和用户作用域过滤
  - RRF/MMR/Cross-Encoder/LLM 重排
  - Context construction

Agent Runtime
  - Reasoning
  - Planning
  - Tool use
  - Response generation
  - New observations written back
```

该架构吸收了 Mem0 的生产记忆层、Zep 的时序图谱、MemGPT 的分层上下文、A-MEM 的动态链接、MIRIX 的多类型记忆和 Reflexion 的经验反馈机制。

## 9. 结论

AI Memory 正从“给 LLM 加一个向量库”演进为 Agent 系统的核心基础设施。2023 年的 Generative Agents、Reflexion、MemoryBank 和 MemGPT 建立了经验流、反思、外部记忆和虚拟上下文的基础范式；2024 年的 MEMORYLLM、HippoRAG、LoCoMo 推动了参数记忆、图谱记忆和长期评测；2025 年后的 MemoRAG、LongMemEval、Zep、A-MEM、Mem0、MIRIX 则把重点推进到生产可用、时序一致、动态组织、多模态和多 Agent 协作。

未来高质量 AI Memory 系统应同时满足四个标准：**记得准、用得对、忘得掉、管得住**。其中，“记得准”依赖多信号抽取和检索，“用得对”依赖与推理/规划闭环整合，“忘得掉”依赖显式生命周期和隐私机制，“管得住”依赖审计、权限、版本和评测。对 AI Memory 架构师而言，关键不是选择单一论文路线，而是根据场景组合不同记忆层：外部存储保障治理，图谱支持关系和时间推理，反思记忆支持自我改进，潜空间记忆提升高频知识效率，多智能体记忆支撑复杂多模态个人助手。

## 参考文献

[^reflexion]: Shinn, N., et al. *Reflexion: Language Agents with Verbal Reinforcement Learning*. arXiv:2303.11366, 2023. https://arxiv.org/abs/2303.11366

[^generative_agents]: Park, J. S., et al. *Generative Agents: Interactive Simulacra of Human Behavior*. arXiv:2304.03442, 2023. https://arxiv.org/abs/2304.03442

[^memorybank]: Zhong, W., et al. *MemoryBank: Enhancing Large Language Models with Long-Term Memory*. AAAI 2024 / arXiv:2305.10250. https://arxiv.org/abs/2305.10250

[^longmem]: Wang, W., et al. *Augmenting Language Models with Long-Term Memory*. NeurIPS 2023 / arXiv:2306.07174. https://arxiv.org/abs/2306.07174

[^memgpt]: Packer, C., et al. *MemGPT: Towards LLMs as Operating Systems*. arXiv:2310.08560, 2023. https://arxiv.org/abs/2310.08560

[^locomo]: Maharana, A., et al. *Evaluating Very Long-Term Conversational Memory of LLM Agents*. ACL 2024 / arXiv:2402.17753. https://arxiv.org/abs/2402.17753

[^memoryllm]: Wang, Y., et al. *MEMORYLLM: Towards Self-Updatable Large Language Models*. ICML 2024 / arXiv:2402.04624. https://arxiv.org/abs/2402.04624

[^hipporag]: Gutiérrez, B. J., et al. *HippoRAG: Neurobiologically Inspired Long-Term Memory for Large Language Models*. NeurIPS 2024 / arXiv:2405.14831. https://arxiv.org/abs/2405.14831

[^memorag]: Qian, H., et al. *MemoRAG: Boosting Long Context Processing with Global Memory-Enhanced Retrieval Augmentation*. WWW 2025 / arXiv:2409.05591. https://arxiv.org/abs/2409.05591

[^longmemeval]: Wu, D., et al. *LongMemEval: Benchmarking Chat Assistants on Long-Term Interactive Memory*. ICLR 2025 / arXiv:2410.10813. https://arxiv.org/abs/2410.10813

[^zep]: *Zep: A Temporal Knowledge Graph Architecture for Agent Memory*. arXiv:2501.13956, 2025. https://arxiv.org/abs/2501.13956

[^amem]: Xu, W., et al. *A-MEM: Agentic Memory for LLM Agents*. NeurIPS 2025 / arXiv:2502.12110. https://arxiv.org/abs/2502.12110

[^mem0]: Chhikara, P., et al. *Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory*. arXiv:2504.19413, 2025. https://arxiv.org/abs/2504.19413

[^mirix]: Wang, Y., and Chen, X. *MIRIX: Multi-Agent Memory System for LLM-Based Agents*. arXiv:2507.07957, 2025. https://arxiv.org/abs/2507.07957

[^human_to_ai_memory]: Wu, Y., et al. *From Human Memory to AI Memory: A Survey on Memory Mechanisms in the Era of LLMs*. arXiv:2504.15965, 2025. https://arxiv.org/abs/2504.15965

[^survey_agents_memory]: Zhang, G., et al. *Memory in the Age of AI Agents*. arXiv:2512.13564, 2025/2026. https://arxiv.org/abs/2512.13564
