# AI Memory 研究最新进展调研报告（2024–2026）

> **作者**：AI Memory 架构调研小组
> **版本**：v1.0
> **完成日期**：2026 年 4 月
> **关键词**：AI Memory、Agent Memory、长期记忆、检索增强生成（RAG）、知识图谱、强化学习、自演化智能体

## 摘要

随着大语言模型（Large Language Models, LLMs）逐步成为自主智能体（autonomous agent）的认知底座，"记忆（Memory）"已经从一个外挂式的辅助模块，演化为决定智能体能否进行长程推理（long-horizon reasoning）、个性化交互、跨会话演化的核心系统能力。本文系统调研了 2024 年至 2026 年间在 AI Memory 领域具有代表性的 15 篇论文，覆盖**外置文本记忆、知识图谱化记忆、操作系统化记忆、参数化记忆、隐空间生成式记忆、强化学习驱动的记忆管理**五大技术路线。本文以"形式（Forms）—功能（Functions）—动力学（Dynamics）"三元视角抽取每篇论文的技术架构、关键贡献与实验结论，并在此基础上对 AI Memory 这一新兴领域进行**横向对比**与**未来发展方向**的总结。我们的核心发现是：（1）记忆已从静态向量库演化为可读、可写、可演化的"第一类系统资源（first-class primitive）"；（2）图结构与时序建模成为复杂多跳推理与多会话一致性的事实标准；（3）强化学习（RL）正在替代启发式规则，成为下一代记忆管理策略的训练范式；（4）隐空间记忆（latent memory）与生成式记忆（generative memory）有望弥合参数化记忆与外置记忆之间的鸿沟；（5）多模态、多智能体、可信治理是下一阶段必须解决的工程级挑战。

---

## 1 引言

### 1.1 研究背景

LLMs 在生成、理解、推理任务上表现卓越，但其本质仍是**无状态（stateless）**的函数：每次推理仅依赖参数权重和当前有限上下文窗口。一旦交互跨越数千轮、跨越数日数月，标准 RAG 框架（静态文档检索）就无法满足以下需求：

1. **长程一致性**：跨会话、跨任务地维护用户画像、偏好与历史决策；
2. **经验复用**：从过去成功/失败的轨迹中蒸馏可迁移的"程序性记忆（procedural memory）"；
3. **持续学习**：在不重新训练参数的前提下，把新知识无损地集成到智能体的世界模型中；
4. **多模态记忆**：除了对话文本，还需要持久化图像、屏幕截图、工具调用轨迹、环境观测等异质信号。

由此，"AI Memory"已成为继 RAG、Tool Use、Planning 之后，智能体研究中第四块基础拼图。本文调研旨在系统梳理这一拼图的最新进展。

### 1.2 调研范围与方法

- **时间范围**：2024 年 5 月（HippoRAG 提出）至 2026 年 3 月。
- **入选标准**：（1）发表于 arXiv / 顶会（NeurIPS、ICLR、ICML、EMNLP）/ 顶会期刊预印本；（2）有清晰的架构创新或新颖的训练范式；（3）开源代码或工业落地。
- **数据来源**：arXiv、ACL Anthology、OpenReview、官方 GitHub 仓库与论文 PDF。
- **分析框架**：参考 Zhang 等人 *Memory in the Age of AI Agents* (arXiv:2512.13564) 提出的**形式（Forms）—功能（Functions）—动力学（Dynamics）**三维分类法，结合 Yu 等人 *Memory for Autonomous LLM Agents* (arXiv:2603.07670) 的**写–管–读（Write–Manage–Read）**生命周期模型展开评述。

### 1.3 本文组织结构

第 2 节给出 AI Memory 的概念边界与三维分类法。第 3 节按"代表性论文 → 架构 → 关键贡献"逐篇剖析 15 篇论文。第 4 节进行横向对比。第 5 节总结未来发展方向。第 6 节为结论。

---

## 2 概念框架

### 2.1 Agent Memory 与相关概念的边界

参考 *Memory in the Age of AI Agents*（Zhang 等, 2025/2026），我们采用如下界定：

| 概念 | 关注点 | 典型代表 |
| --- | --- | --- |
| **LLM Memory** | 模型权重内编码的世界知识 | 预训练 + 微调 |
| **RAG** | 静态文档检索与上下文拼接 | LangChain RAG、HippoRAG |
| **Context Engineering** | 上下文窗口的动态构造 | Long-context、Prompt Caching |
| **Agent Memory** | 跨任务/跨会话持久化、可写、可演化 | MemGPT、Mem0、A-MEM、MemoryOS |

Agent Memory 的核心特征是**跨会话持久化（persistence）**与**自演化（evolvability）**，而 RAG 通常面向静态语料。

### 2.2 三维分类法

**1. 形式（Forms）：记忆载体**

- **Token-level Memory（显式记忆）**：以自然语言文本/三元组形式存储，可直接读取，如 Mem0、MemoryBank、A-MEM。
- **Parametric Memory（参数化记忆）**：编码进模型权重或 LoRA 模块，例如持续预训练、Memory Tokens。
- **Latent Memory（隐空间记忆）**：以隐藏状态/连续向量为载体，例如 KV Cache 压缩、MemGen 的 latent token。

**2. 功能（Functions）：记忆作用**

- **Factual Memory**：事实性知识（用户姓名、偏好、领域知识）；
- **Experiential Memory**：经验/技能（成功/失败的推理策略、程序性技能）；
- **Working Memory**：当前任务相关的活跃上下文。

**3. 动力学（Dynamics）：记忆生命周期**

- **Formation（写入/抽取）**：从交互中识别、抽取、压缩信息；
- **Evolution（更新/巩固/遗忘）**：合并、修订、遗忘与对抗冲突；
- **Retrieval（读取/激活）**：根据当前查询激活相关记忆。

---

## 3 代表性论文剖析

下文按"**论文标题（年份）**"为单位组织，每篇论文给出：①问题动机；②技术架构；③关键贡献；④实验结论。

### 3.1 HippoRAG：海马体启发的长期记忆 RAG（NeurIPS 2024）

**论文**：Gutiérrez 等, *HippoRAG: Neurobiologically Inspired Long-Term Memory for Large Language Models* (arXiv:2405.14831, NeurIPS 2024)

**问题动机**：标准 RAG 仅基于稠密向量近似最近邻检索，难以胜任跨文档的多跳关联与"知识整合"。生物学上，海马体充当**索引引擎**，新皮层负责存储；本文以这一神经生物学理论为蓝本设计 RAG。

**技术架构**：

1. **离线索引（Indexing as Hippocampus）**：用 LLM 从语料中抽取 (subject, predicate, object) 三元组，构建知识图谱（KG）。
2. **在线检索（Retrieval as Pattern Completion）**：将查询解析为概念实体，注入 KG，通过 **Personalized PageRank（PPR）**做能量扩散；高分节点对应的 passages 即为答案。
3. **新皮层（Neocortex）**：BM25 / 稠密向量提供局部相似度能量。

**关键贡献**：

- 首次将"海马体索引理论"工程化为可落地的 RAG 框架；
- 单步检索即可超越迭代式 IRCoT，**多跳 QA 提升高达 20%**，速度提升 6–13 倍，成本降低 10–30 倍。

**实验**：在 MuSiQue、2Wiki、HotpotQA 多跳 QA 上全面领先。

---

### 3.2 HippoRAG 2：从 RAG 到 Memory（ICML 2025）

**论文**：Gutiérrez 等, *From RAG to Memory: Non-Parametric Continual Learning for Large Language Models* (arXiv:2502.14802, ICML 2025)

**核心改进**：

- **稠密–稀疏融合**：在 KG 中同时引入 phrase 节点与 passage 节点，平衡事实问答与关联推理；
- **Recognition Memory**：增加 LLM 过滤器，剔除噪声三元组；
- **在线 LLM 闭环**：LLM 同时承担抽取、过滤、阅读三种角色。

**实验**：在 MuSiQue 上 F1 由 44.8 提升到 51.9；在 2Wiki 上 Recall@5 由 76.5% 提升到 90.4%。相比 GraphRAG/RAPTOR 等结构化 RAG，**保留了简单 QA 上的性能不退化**。

**意义**：将 RAG 重新定位为"非参数化持续学习（non-parametric continual learning）"的载体，是 RAG → Memory 范式过渡的标志性工作。

---

### 3.3 A-MEM：智能体化的记忆系统（2025）

**论文**：Xu 等, *A-MEM: Agentic Memory for LLM Agents* (arXiv:2502.12110)

**问题动机**：MemoryBank、MemGPT 这类记忆系统采用**静态、预定义的存储/检索操作**，难以适应任务多样性。

**技术架构**：受卡片盒笔记法（**Zettelkasten**）启发：

1. **原子化笔记**：每条记忆由 (上下文描述、关键词、标签) 等结构化属性组成；
2. **动态链接**：新记忆生成后，由 LLM 自主分析与已有记忆的语义相关性，建立链接；
3. **记忆演化**：新记忆的写入会反向触发对历史记忆描述的更新，使整个网络持续精炼。

**关键贡献**：

- 首次将"智能体（agentic）"思想引入记忆管理本身——**记忆模块由 LLM 动态决策结构与链接**；
- 在 LoCoMo 上，多跳推理 F1 至少 2× 于现有 SOTA；
- 每次记忆操作仅 ~1,200 tokens，相对 LoCoMo / MemGPT 的 16,900 tokens 节省 85–93%。

---

### 3.4 Zep：时序知识图谱式 Agent Memory（2025）

**论文**：Rasmussen 等, *Zep: A Temporal Knowledge Graph Architecture for Agent Memory* (arXiv:2501.13956)，配套开源 Graphiti 引擎。

**技术架构**：三层有向图 G=(N, E, φ)：

1. **Episode Subgraph**：原始对话/事件节点，保留完整 provenance；
2. **Semantic Entity Subgraph**：实体与关系节点，支持去重；
3. **Community Subgraph**：通过 label propagation 形成主题社区，提供高层概要。

**双时间建模**：每条边同时记录"事实生效时间"与"系统知晓时间"，支持时间点查询、边失效与历史回溯。

**检索**：Hybrid Search（cosine + BM25 + KG BFS）+ 多阶段重排（RRF, MMR, LLM cross-encoder）。

**实验**：DMR 94.8% vs MemGPT 93.4%；LongMemEval 准确率 +18.5%，延迟降低 90%。

**意义**：是首个把"时间维度作为一等公民"嵌入 Agent Memory 设计的开源工业级方案。

---

### 3.5 Mem0 / Mem0g：生产级长期记忆（2025）

**论文**：Chhikara 等, *Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory* (arXiv:2504.19413)

**技术架构**：

- **Extraction Phase**：以"最近消息 + 滚动摘要 + 当前消息对"为 prompt，让 LLM 抽取候选事实；摘要由后台异步刷新，主链路不阻塞。
- **Update Phase**：候选事实与向量库中 top-s 相似条目比对，通过 function-calling 触发 **ADD / UPDATE / DELETE / NOOP** 四元操作。
- **Mem0g**：在此基础上扩展为带类型标签的有向图（Neo4j 实现），支持实体、关系、冲突解决。

**关键贡献**：

- LOCOMO 总分相比 OpenAI Memory **+26%**；
- p95 延迟下降 **91%**（17.12 s → 1.44 s）；token 节省 **90%**；
- 是 2025 年最具工程影响力的开源 Memory Layer 之一，已被广泛集成到生产对话系统。

---

### 3.6 MemOS：面向 LLM 的"记忆操作系统"（2025）

**论文**：Li 等, *MemOS: An Operating System for Memory-Augmented Generation in LLMs* (arXiv:2505.22101)

**技术架构**：将记忆抽象为**第一类系统资源**，提出 **MemCube** 标准化记忆单元，统一管理三种记忆：

- **Parametric Memory**（权重内）；
- **Activation Memory**（运行时上下文/KV Cache）；
- **Plaintext Memory**（外置文本/图谱）。

**三层架构**：Interface Layer（API、解析）→ Operation Layer（MemScheduler、MemOperator、MemGovernance）→ Infrastructure Layer（多模态、跨平台、安全治理）。

**关键贡献**：首次将"**操作系统抽象**"系统性地引入 LLM 记忆管理，支持版本控制、迁移、跨平台共享与多用户治理；为后续 **EverMemOS** 等"自组织记忆 OS"奠定基础。

---

### 3.7 MemoryOS：层级化短中长期记忆 OS（EMNLP 2025 Oral）

**论文**：Kang 等, *Memory OS of AI Agent* (arXiv:2506.06326, EMNLP 2025)

**技术架构**：

- **三级存储**：Short-Term（FIFO 对话链）、Mid-Term（按主题的分段分页结构）、Long-term Persona Memory（用户画像+知识库）。
- **更新策略**：STM→MTM 采用基于对话链的 FIFO；MTM→LPM 采用**热度（热度 = 检索次数 × 交互长度 × 时间衰减）**驱动的分页迁移与淘汰。
- **检索**：分层多源检索 + 个性化生成。

**实验**：LoCoMo 上 F1 平均提升 **49.11%**、BLEU-1 提升 **46.18%**（相对基线，GPT-4o-mini）。

---

### 3.8 MIRIX：多智能体多模态记忆系统（2025）

**论文**：Wang & Chen, *MIRIX: Multi-Agent Memory System for LLM-Based Agents* (arXiv:2507.07957)

**技术架构**：

- **六种记忆类型**：Core、Episodic、Semantic、Procedural、Resource、Knowledge Vault；每类记忆由专属 Agent 管理。
- **Meta Memory Manager**：协调路由、写入与检索。
- **多模态原生**：支持高分辨率截屏（ScreenshotVQA）、文本对话（LOCOMO）。

**实验**：

- ScreenshotVQA（约 20,000 张高清截图/序列）：相对 RAG 基线**准确率 +35%、存储节省 99.9%**；
- LOCOMO：85.4% 准确率，**SOTA**。

**意义**：是"记忆即多智能体协作"思想的代表作品；同时验证了**多模态长期记忆**的可行性。

---

### 3.9 Memp：智能体程序性记忆（2025）

**论文**：Fang 等, *Memp: Exploring Agent Procedural Memory* (arXiv:2508.06433)

**核心思想**：把过去成功的轨迹蒸馏成两个层次：

- **Step-by-step Instructions**：细粒度执行指令；
- **Script-like Abstractions**：高层任务脚本/技能。

**关键技术**：Build / Retrieval / Update 全生命周期协议；动态淘汰与修正机制。

**实验**：在 TravelPlanner、ALFWorld 上随着记忆库精炼，成功率与效率持续提升；**强模型蒸馏的程序性记忆迁移到弱模型仍带来显著增益**——这暗示了程序性记忆作为"模型无关 IP"的潜在价值。

---

### 3.10 Memory-R1：用 RL 学习记忆管理（2025）

**论文**：Yan 等, *Memory-R1: Enhancing LLM Agents to Manage and Utilize Memories via RL* (arXiv:2508.19828)

**问题动机**：现有 ADD/UPDATE/DELETE 操作多由启发式规则触发，**学不到全局最优策略**。

**技术架构**：

- **Memory Manager**：RL 微调，输出 ADD / UPDATE / DELETE / NOOP；
- **Answer Agent**：RL 微调，先做 RAG 取 top-60 候选，再用 Memory Distillation 策略筛选；
- 训练采用 **PPO / GRPO**，仅 152 个 QA 对即可。

**实验（LoCoMo / LLaMA-3.1-8B）**：相对 Mem0 提升 F1 **+48%**、BLEU-1 **+69%**、LLM-Judge **+37%**；3B–14B 多个尺度均稳定有效。

**意义**：开创"**记忆管理即策略**"的训练范式，用 outcome-driven RL 替代启发式记忆操作。

---

### 3.11 多记忆系统 MMS：认知心理学启发（2025）

**论文**：Gao 等, *Multiple Memory Systems for Enhancing the Long-term Memory of Agent* (arXiv:2508.15294)

**核心架构**：将短期记忆通过四种"认知加工"角度并行抽取为长期记忆碎片：

- 关键词；
- 多种认知视角；
- 情景记忆；
- 语义记忆。

每个碎片对偶生成**Retrieval Unit**（用于查询匹配）与**Contextual Unit**（用于回答增强）。这一对偶设计源自"编码特异性原则（encoding specificity）"。

**实验**：在 LoCoMo 上 Recall@N、F1、BLEU-1 全面优于 MemoryBank、A-MEM、NaiveRAG，特别是在多跳与开放域问题上。

---

### 3.12 ReasoningBank + MaTTS：用记忆驱动 Test-Time Scaling（ICLR 2026）

**论文**：Ouyang 等, *ReasoningBank: Scaling Agent Self-Evolving with Reasoning Memory* (arXiv:2509.25140, ICLR 2026)

**技术架构**：

- **ReasoningBank**：把成功**与失败**轨迹蒸馏为 (title, description, content) 三元结构的可迁移推理策略；区别于"只记成功"的旧范式。
- **MaTTS（Memory-aware Test-Time Scaling）**：在 parallel 与 sequential 两种模式下增加每个任务的探索预算，多样化轨迹反过来为记忆提供"对比信号（contrastive signals）"。

**实验**：WebArena、Mind2Web、SWE-Bench-Verified 上相对 Synapse、AWM 等基线**成功率提升至多 34.2%、交互步数减少 16%**。

**意义**：提出**记忆驱动的经验扩展（experience scaling）**作为"测试时计算扩展"之外的新维度。

---

### 3.13 MemGen：生成式隐空间记忆（ICLR 2026）

**论文**：Zhang 等, *MemGen: Weaving Generative Latent Memory for Self-Evolving Agents* (arXiv:2509.24704, ICLR 2026)

**问题动机**：参数化记忆需改写权重；外置记忆需检索；二者都没有捕捉到人脑"推理-记忆-再推理"的连续耦合。

**技术架构**：

- **Memory Trigger**（RL 训练）：监控隐藏状态，决定何时调用记忆；
- **Memory Weaver**：以当前状态为提示，**生成一段隐空间 token 序列（latent memory）**，直接注入推理流。

**关键贡献**：

- 在 8 个基准上相对 ExpeL、AWM 提升至多 **38.22%**，相对 GRPO 提升 **13.44%**；
- **无监督的情况下**自发涌现出 planning memory、procedural memory、working memory 三种功能性记忆；
- 推理核心权重不变，避免灾难性遗忘。

**意义**：提出"**记忆即生成（memory-as-generation）**"——把记忆从"外部数据库"还原为"隐空间生成过程"，开创隐空间生成记忆这一新方向。

---

### 3.14 LightMem：轻量化高效记忆（ICLR 2026）

**论文**：Fang 等, *LightMem: Lightweight and Efficient Memory-Augmented Generation* (arXiv:2510.18866, ICLR 2026)

**架构灵感**：Atkinson–Shiffrin 三阶段人脑记忆模型。

- **Sensory Memory**：轻量压缩 + 主题分组，过滤噪声；
- **Short-Term Memory**：主题级摘要、结构化访问；
- **Long-Term Memory + Sleep-time Update**：离线"睡眠时刻"批量整合，**与在线推理解耦**。

**实验（LongMemEval / LoCoMo）**：相对强基线**准确率提升 7.7–29.3%**；token 消耗降低 38–117 倍；API 调用降低 30–310 倍。

**意义**：明确把"**性能–效率折中**"作为一等设计目标，并通过"睡眠期更新"巧妙解决在线写入开销问题。

---

### 3.15 MemSearcher：端到端 RL 训练的搜索-推理-记忆管理（2025）

**论文**：Yuan 等, *MemSearcher: Training LLMs to Reason, Search and Manage Memory via End-to-End Reinforcement Learning* (arXiv:2511.02805)

**技术架构**：

- 每一轮 LLM 仅看到 **(用户问题, 紧凑记忆)** 两个简短输入；
- LLM 同时承担：①推理；②搜索动作；③记忆管理（更新紧凑记忆）；
- 采用 **multi-context GRPO**：同一轨迹下不同上下文的多组样本共享 trajectory-level advantage，端到端联合优化。

**实验**：7 个公开基准上相对 ReAct 基线 **Qwen2.5-3B +11%、7B +12%**；3B 版本即可超过原 7B 基线，体现"信息完整性 vs 效率"折中后的双赢。

**意义**：把 ReAct 范式中"**记忆即上下文堆叠**"重构为"**记忆即压缩状态**"，并以 RL 端到端联合训练，是 search agent 记忆设计的范式转变。

---

### 3.16 综述 & 元工作（2025–2026）

为了完整起见，本文同时纳入两篇重要综述作为元参考：

1. **Memory in the Age of AI Agents**（Zhang 等, arXiv:2512.13564, 2025/2026）：提出 Forms / Functions / Dynamics 三维统一框架，区分 token-level / parametric / latent 三种载体，区分 factual / experiential / working 三种功能；附带配套论文清单仓库 `Shichun-Liu/Agent-Memory-Paper-List`。
2. **Memory for Autonomous LLM Agents**（Yu 等, arXiv:2603.07670, 2026）：以 **write–manage–read** 闭环为骨架，将机制族归纳为五类：context-resident compression、retrieval-augmented stores、reflective self-improvement、hierarchical virtual context、policy-learned management；并系统梳理了基准的演进轨迹。

二者共同确立了"AI Memory"作为一个**独立子领域**的概念坐标。

---

## 4 横向对比

### 4.1 技术架构对比矩阵

| 系统 | 形式（载体） | 主要功能 | 写入策略 | 更新/巩固 | 检索机制 | 训练范式 | 代表性指标 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| HippoRAG / 2 | Token + KG | Factual + Sense-making | LLM 三元组抽取 | 静态 + LLM 过滤 | PPR + 稠密/稀疏 | 训练-free | MuSiQue F1 51.9 |
| A-MEM | Token Note + 动态图 | Factual + Experiential | LLM 自主生成笔记 | 触发式相互更新 | 笔记网络 + Top-k | 训练-free | LoCoMo 多跳 SOTA |
| Zep / Graphiti | 三层时序图 | All | Episode-by-episode | 边失效 + 双时间 | Hybrid + 多阶段重排 | 训练-free | DMR 94.8% |
| Mem0 / Mem0g | 向量 + 可选 KG | Factual | LLM 抽取 | ADD/UPDATE/DELETE/NOOP | 向量近邻 + 图遍历 | 训练-free | LOCOMO +26% |
| MemOS | Parametric+Activation+Plaintext | All（系统级） | MemCube 单元 | 调度+治理+迁移 | API 化访问 | 训练-free | 框架级 |
| MemoryOS | 三级层次 | All | LLM + FIFO | 热度淘汰、分页迁移 | 分层多源 | 训练-free | LoCoMo F1 +49.11% |
| MIRIX | 6 类专用记忆 + 多智能体 | All（含多模态） | 各 Agent 写入 | 元 Manager 协调 | 类型路由 | 训练-free | LoCoMo 85.4% |
| Memp | 程序性记忆 | Experiential | 蒸馏轨迹 | 动态淘汰 | 任务相似度 | 训练-free | TravelPlanner 增益 |
| Memory-R1 | 向量库 | Factual + Experiential | RL Manager | RL 学习 ADD 等 | RAG + Distillation | PPO/GRPO | F1 +48% |
| MMS | 多碎片 + 对偶单元 | Factual + Experiential | 多视角抽取 | — | 检索-上下文对偶 | 训练-free | LoCoMo 多跳领先 |
| ReasoningBank + MaTTS | Token 推理策略 | Experiential | 成功+失败蒸馏 | 增量合并 | 嵌入 + 主题 | 训练-free + Test-Time Scaling | WebArena +34.2% |
| MemGen | Latent token | Working + Procedural + Planning（涌现） | 触发-编织 | RL 学习触发 | 隐空间内嵌 | RL（Trigger+Weaver） | 8 基准 +38.22% |
| LightMem | Token 三阶段 | All | 感觉记忆压缩 | Sleep-time | 嵌入/上下文/混合 | 训练-free | tokens ↓117× |
| MemSearcher | 紧凑 Token Memory | Working | LLM 端到端写 | RL 学习压缩 | 内嵌于上下文 | 多上下文 GRPO | +11/12% |

### 4.2 共性观察

1. **Token-level + 图结构** 仍是当前最主流的"可解释 + 可检索"组合；
2. **时间一致性** 已成为生产系统的硬要求（Zep、MemoryOS、LightMem 都显式建模）；
3. **写入路径越来越"智能体化"**：从启发式 → LLM 自主决策 → RL 学习；
4. **效率成为一等公民**：A-MEM、Mem0、LightMem 均以 token/延迟/API 调用为关键指标；
5. **失败经验同样重要**：ReasoningBank 把"失败轨迹 + 自我评判"作为对比信号引入。

---

## 5 未来发展方向

基于上述梳理，结合 *Memory in the Age of AI Agents* 与 *Memory for Autonomous LLM Agents* 提出的开放问题，本文总结出**七个值得长期投入**的研究方向：

### 5.1 记忆自动化（Memory Automation）

当前许多系统仍依赖人工设计的"写入路径"与"提示模板"。未来工作应让记忆系统**自我决策**：何时写、写什么、写到哪、如何遗忘。Memory-R1、MemSearcher、MemGen 是这一方向的早期代表，需进一步在多任务/多模态场景验证。

### 5.2 强化学习与记忆的深度融合

RL 已在 Memory-R1（外置）、MemGen（隐空间）、MemSearcher（紧凑上下文）、ReasoningBank（轨迹蒸馏）四种范式中得到验证。下一步包括：

- **多目标奖励**：兼顾准确性、效率、隐私、可信；
- **离线 RL 与世界模型**：从用户日志中离线学习记忆策略；
- **元 RL**：跨用户、跨任务迁移记忆管理策略。

### 5.3 多模态与具身记忆（Multimodal & Embodied Memory）

MIRIX、MemVerse 等已经开始处理屏幕截图、视频帧；而具身智能体（机器人、自动驾驶）需要把**感知–动作–环境状态**作为一等记忆对象。需要新的：

- 多模态压缩与去重；
- 时空一致的 4D 记忆图谱；
- 与世界模型/具身基础模型协同的 retrieval。

### 5.4 多智能体共享记忆与协作

未来 Agent 不会孤立运行。共享记忆引发：

- 命名空间与权限边界；
- 跨智能体冲突解决；
- 联邦/差分隐私下的记忆同步；
- 公私记忆划分（公共知识 vs 用户私域）。

MemOS 的发布订阅式 MemCube 给出初步答案。

### 5.5 隐空间 / 生成式 / 混合记忆

MemGen 表明 latent memory 能**自发涌现**类人记忆功能。未来可能出现：

- "**显式 + 隐式 + 参数**"的混合记忆栈；
- 类似人脑"**清醒-睡眠**"双相整合（LightMem 已小步实践）；
- 与 Long-context、KV Cache 压缩、State Space Models（SSM）深度融合。

### 5.6 持续学习与忘却（Continual Learning & Forgetting）

人类记忆的核心机制之一是**主动遗忘**。当前多数系统的"遗忘"仍是热度淘汰或显式 DELETE。未来需要：

- **学习型遗忘策略**（learned forgetting）；
- **基于因果与可信度的剪枝**；
- **可控记忆消除（Right to be forgotten / GDPR 合规）**。

### 5.7 可信治理：安全、隐私、对齐与基准

记忆是个人化、敏感的。Trustworthy Memory 涉及：

- **Provenance（来源可追溯）**：Zep 的双时间、MemOS 的版本控制是开端；
- **Hallucinated Memory 防御**：对抗注入、虚假回忆；
- **记忆对齐**：与用户长期价值观一致；
- **基准建设**：从 LoCoMo / LongMemEval / DMR 走向更复杂的多智能体、多模态、多会话实战基准（如 WebArena、ScreenshotVQA、SWE-Bench-Verified）。

---

## 6 结论

本文系统调研了 2024–2026 年间 AI Memory 领域的 15+ 篇代表性论文，归纳出以下五点结论：

1. **范式跃迁**：AI Memory 已从"外挂的向量库"跃迁为"第一类系统资源"，并衍生出 OS（MemOS、MemoryOS）、多智能体（MIRIX）、隐空间（MemGen）等新形态；
2. **图与时序**：Zep、A-MEM、HippoRAG 2 等表明，**图结构 + 时序建模**是支撑多跳与多会话推理的事实标准；
3. **学习取代启发式**：Memory-R1、MemSearcher、MemGen、ReasoningBank 共同表明，记忆管理正从规则驱动迈入**RL/自演化**阶段；
4. **效率与性能并重**：LightMem、Mem0、A-MEM 在保持或提升性能的同时显著降低 token / 延迟 / API 调用，"工程美学"逐渐成为评估的一等指标；
5. **认知心理学回归**：Atkinson–Shiffrin、Zettelkasten、海马体索引、编码特异性等心理学/神经科学原理正在以**可工程化**的方式重新进入主流。

未来 2–3 年，AI Memory 的研究将围绕"**自动化、强化学习化、多模态化、可信化**"四个轴心展开；并与 Long Context、Continual Learning、World Model、Embodied AI 等方向交叉融合，最终把 LLM 智能体推向真正"会记、会忘、会成长"的下一阶段。

---

## 参考文献（按本文出现顺序）

1. Zhang, G., et al. *Memory in the Age of AI Agents*. arXiv:2512.13564, 2025/2026.
2. Yu, et al. *Memory for Autonomous LLM Agents: Mechanisms, Evaluation, and Emerging Frontiers*. arXiv:2603.07670, 2026.
3. Gutiérrez, B. J., et al. *HippoRAG: Neurobiologically Inspired Long-Term Memory for Large Language Models*. arXiv:2405.14831, NeurIPS 2024.
4. Gutiérrez, B. J., et al. *From RAG to Memory: Non-Parametric Continual Learning for Large Language Models*. arXiv:2502.14802, ICML 2025.
5. Xu, W., et al. *A-MEM: Agentic Memory for LLM Agents*. arXiv:2502.12110, 2025.
6. Rasmussen, P., et al. *Zep: A Temporal Knowledge Graph Architecture for Agent Memory*. arXiv:2501.13956, 2025.
7. Chhikara, P., Khant, D., Aryan, S., Singh, T., Yadav, D. *Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory*. arXiv:2504.19413, 2025.
8. Li, Z., et al. *MemOS: An Operating System for Memory-Augmented Generation in Large Language Models*. arXiv:2505.22101, 2025.
9. Kang, J., Ji, M., Zhao, Z., Bai, T. *Memory OS of AI Agent*. arXiv:2506.06326 / EMNLP 2025 (Oral).
10. Wang, Y., Chen, X. *MIRIX: Multi-Agent Memory System for LLM-Based Agents*. arXiv:2507.07957, 2025.
11. Fang, J., et al. *Memp: Exploring Agent Procedural Memory*. arXiv:2508.06433, 2025.
12. Yan, S., Bi, et al. *Memory-R1: Enhancing Large Language Model Agents to Manage and Utilize Memories via Reinforcement Learning*. arXiv:2508.19828, 2025/2026.
13. Gao, K., et al. *Multiple Memory Systems for Enhancing the Long-term Memory of Agent*. arXiv:2508.15294, 2025.
14. Ouyang, S., et al. *ReasoningBank: Scaling Agent Self-Evolving with Reasoning Memory*. arXiv:2509.25140, ICLR 2026.
15. Zhang, G., Fu, M., Yan, S. *MemGen: Weaving Generative Latent Memory for Self-Evolving Agents*. arXiv:2509.24704, ICLR 2026.
16. Fang, J., Deng, X., et al. *LightMem: Lightweight and Efficient Memory-Augmented Generation*. arXiv:2510.18866, ICLR 2026.
17. Yuan, Q., et al. *MemSearcher: Training LLMs to Reason, Search and Manage Memory via End-to-End Reinforcement Learning*. arXiv:2511.02805, 2025.
18. Packer, C., et al. *MemGPT: Towards LLMs as Operating Systems*. arXiv:2310.08560, 2024.
19. Zhong, et al. *MemoryBank: Enhancing Large Language Models with Long-Term Memory*. AAAI 2024.
20. *MemVerse: Multimodal Memory for Lifelong Learning Agents*. arXiv:2512.03627, 2025.

---

> **写作说明**：本文为面向开发者与研究人员的技术调研报告，旨在帮助读者快速建立 AI Memory 领域的全貌认知，并辅助选型与下一步研究规划。建议结合 *all-in-rag* 项目第七章「基于知识图谱的 RAG」一同阅读，以理解 Memory 与 RAG 在工程落地中的衔接关系。
