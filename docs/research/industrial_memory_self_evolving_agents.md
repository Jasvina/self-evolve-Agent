# 产业级记忆 - 自我演化智能体的轨迹分析与长期记忆能力探索

- 调研日期：2026-04-28
- 目标仓库：https://github.com/Jasvina/self-evolve-Agent
- 课题关键词：长期记忆、轨迹分析、自我演化智能体、高难算法任务、策略重用、动态优化、伐谋/VRSO

## 1. 执行摘要

本课题不应把“记忆”理解为简单的聊天历史或向量库，而应定义为一种可被智能体在未来任务中稳定调用、可被评估证明有收益、可被版本化维护的工程资产。面向高难算法任务和伐谋已有的长轨迹日志，最有价值的记忆不是完整日志本身，而是从日志中提炼出的：失败模式、关键状态转折、策略适用条件、可复用修复片段、评估反馈、以及经验证的反事实选择。

核心判断：

1. **研究空缺明确**：现有长期记忆工作多围绕聊天、多会话问答或通用个性化；现有自我演化工作多围绕反思、prompt/policy 优化或 agent 代码改写。二者之间缺少一个专门面向“高难算法任务轨迹 -> 可用长期记忆 -> 未来任务收益”的闭环。
2. **伐谋/VRSO 的切入点很强**：本地 `VRSO-longhorizon` 已把失败恢复对象收紧到 `minimal recoverable subtrajectory`，这天然适合作为记忆抽取的最小证据单元，而不是对整条失败轨迹做粗粒度摘要。
3. **产业级记忆的关键标准**：不仅要能存，还要能查准、查得快、能解释、能过期/撤销、能在 verifier 或 benchmark 上证明收益，并且不会因陈旧/错误记忆造成负迁移。
4. **建议的系统形态**：采用“分层记忆 + 证据绑定 + 多路检索 + 在线更新”的架构：短期工作态、episodic 轨迹片段、procedural 策略规则、diagnostic 失败模式、semantic 任务/工具事实、evolutionary agent 版本档案。
5. **第一阶段实验优先级**：先做离线轨迹挖掘与结构化记忆注入，不急于训练大模型权重；用 no-memory、raw trajectory retrieval、reflection-only、structured memory 四类对照证明结构化记忆的边际价值。

## 2. 课题问题重述

给定一个会在高难算法任务上不断尝试、失败、修复、演化的 Agent，我们希望回答三个问题：

- **记什么**：轨迹中的哪些信息值得沉淀为长期记忆？完整状态、失败步骤、修复 span、策略描述、评价信号、还是 agent 版本变更？
- **怎么记**：如何把超长日志压缩成结构化、可检索、可验证、可更新的记忆对象？
- **怎么用**：未来相似任务到来时，如何动态选择相关记忆，并证明它提高了成功率、降低了 token/工具调用成本、或提升了失败恢复效率？

因此，本课题的研究对象不是“更大的上下文窗口”，而是 **可用记忆（usable memory）**。一个记忆对象至少应满足：

- **可定位**：能追溯到原始轨迹、状态、动作、反馈、verifier 证据。
- **可压缩**：比原始日志短，能进入 prompt 或检索结果。
- **可迁移**：能跨相似任务复用，而不是只复述一次具体经历。
- **可检索**：有任务签名、状态签名、失败模式、工具/算法标签等索引。
- **可验证**：注入后能在 held-out 任务或 replay 中体现收益。
- **可维护**：支持版本、置信度、过期、冲突解决和负迁移记录。

## 3. 与伐谋/VRSO 现有基础的衔接

基于本地 `VRSO-longhorizon` 的当前说明，伐谋主线已经具备三类对本课题非常有价值的基础：

1. **修复对象已明确**：当前研究对象是 long-horizon agent failure recovery，核心对象是最小可修复连续子轨迹（minimal recoverable subtrajectory / repair span）。这比整条轨迹摘要更适合作为记忆的最小单位。
2. **已有轨迹与评估信号**：README 中显示当前 pipeline 已覆盖 anchor localization、dependency-consistent span proposal、verifier-backed minimality，以及 PRM/DSA/SAPO 等实验信号。这些都可成为记忆抽取和筛选特征。
3. **有外部 sanity 环境**：`dependent_tools` 是受控主线，`tau2` 是外部 sanity check；这适合形成“先受控、再外部泛化”的记忆评估路径。

建议把伐谋现有的 `minimal recoverable subtrajectory` 升级为长期记忆系统中的 **证据锚点**：每条记忆都应绑定到至少一个可 replay、可验证、可最小化的 span 或反事实选择，而不是只绑定自然语言总结。

## 4. 相关工作格局

### 4.1 Agent 长期记忆与记忆架构

- **Generative Agents**：提出用自然语言保存完整经历，并通过反思与动态检索支撑后续规划。这说明“经验 -> 高层反思 -> 行为规划”是可行的记忆路径，但其任务主要是社会行为模拟，不是高难算法任务。[arXiv:2304.03442](https://arxiv.org/abs/2304.03442)
- **MemGPT / Letta**：把 LLM 记忆类比为操作系统式的层级内存管理：上下文内核心记忆、可检索的外部记忆、以及由 agent 自主管理的 memory tools。这给产业系统提供了重要启示：记忆不是单一向量库，而是 context management。[arXiv:2310.08560](https://arxiv.org/abs/2310.08560), [Letta docs](https://docs.letta.com/guides/core-concepts/stateful-agents)
- **CoALA**：把语言智能体放进认知架构框架，强调模块化记忆、内部/外部动作空间和决策过程。这对本课题的启发是：记忆系统必须和 action space、decision loop 一起设计，而不是作为外挂 RAG。[arXiv:2309.02427](https://arxiv.org/abs/2309.02427)
- **LangGraph Memory**：明确区分短期 thread-scoped memory 与跨 session 的长期 memory，并把长期 memory 拆成 semantic、episodic、procedural 三类；同时讨论 hot path 写入和 background 写入的工程取舍。[LangGraph docs](https://docs.langchain.com/oss/python/concepts/memory)
- **LongMemEval**：提出长期交互记忆评估的五种能力：信息抽取、多会话推理、时间推理、知识更新、拒答/不确定性，并指出商业助手和长上下文模型在持续交互中仍有明显下降。[arXiv:2410.10813](https://arxiv.org/abs/2410.10813)
- **Mem0**：定位为 production-ready long-term memory，强调动态抽取、合并、检索，并报告低延迟、低 token 成本的工程收益。其 2026 年 README 进一步强调 ADD-only extraction、entity linking 与 semantic/BM25/entity 多信号融合。[arXiv:2504.19413](https://arxiv.org/abs/2504.19413), [mem0 GitHub](https://github.com/mem0ai/mem0)
- **Zep / Graphiti**：用 temporal knowledge graph 管理 agent memory，强调随时间变化的关系、事实有效期、episodic nodes 与 entity/fact edges。这对“策略/工具/任务条件随版本变化”的产业级记忆很重要。[Zep docs](https://help.getzep.com/graph-overview)
- **Memory in the Age of AI Agents**：2025/2026 综述把 agent memory 从 forms、functions、dynamics 三个维度统一，区分 token-level、parametric、latent memory，以及 factual、experiential、working memory，并指出自动化记忆管理、RL、多智能体记忆、可信记忆是前沿方向。[arXiv:2512.13564](https://arxiv.org/abs/2512.13564)

### 4.2 自我演化、反思与轨迹学习

- **Reflexion**：通过语言反馈和 episodic memory buffer，让 agent 不更新权重也能从失败中改进行为；在 coding、reasoning、decision-making 上都有收益。它证明了“失败反馈 -> 反思文本 -> 下次尝试”这条路径的有效性，但记忆结构相对粗粒度。[arXiv:2303.11366](https://arxiv.org/abs/2303.11366)
- **Self-Refine**：让同一个 LLM 生成、反馈、再改写，不需要额外训练即可提升多任务输出质量。这更像 test-time refinement，对长期跨任务记忆的支持不足，但可作为单次轨迹内的 baseline。[arXiv:2303.17651](https://arxiv.org/abs/2303.17651)
- **Tree of Thoughts / LATS**：把问题求解变成多路径搜索、评估和回溯；LATS 进一步把 reasoning、acting、planning 和环境反馈结合，对 programming、web navigation、math 等任务有效。它们提示我们：轨迹记忆可以服务于搜索启发式、value estimation 和回溯决策。[ToT arXiv:2305.10601](https://arxiv.org/abs/2305.10601), [LATS arXiv:2310.04406](https://arxiv.org/abs/2310.04406)
- **Voyager**：通过自动课程、可执行 skill library、环境反馈和自验证，在 Minecraft 中积累可组合技能。对本课题的启发是：长期记忆不只存文本，还可存可执行策略或模板。[arXiv:2305.16291](https://arxiv.org/abs/2305.16291)
- **ExpeL**：从训练任务中自主收集经验并抽取自然语言知识，推理时回忆经验和 insights。它非常接近“经验抽取为可用记忆”的方向，但仍需面向高难算法轨迹进一步细化状态/失败/验证结构。[arXiv:2308.10144](https://arxiv.org/abs/2308.10144)
- **Agent-Pro**：提出 policy-level reflection and optimization，不只反思单个动作，而是从过去轨迹和 belief 中演化策略；这对本课题的 procedural memory 很关键。[ACL 2024](https://aclanthology.org/2024.acl-long.292/)
- **Symbolic Learning Enables Self-Evolving Agents**：把 prompts、tools、pipeline 看作可学习的 symbolic network，用自然语言版 loss/gradient/weights 优化 agent，让 agent 部署后继续自我更新。它给“记忆驱动的 agent 自演化”提供了直接理论邻近。[arXiv:2406.18532](https://arxiv.org/abs/2406.18532)
- **Darwin Gödel Machine**：让 coding agent 迭代修改自身代码并用 coding benchmarks 验证，维护生成 agent 的 archive；这说明“演化档案 + benchmark 验证 + 开放式搜索”是更强的自演化范式。[arXiv:2505.22954](https://arxiv.org/abs/2505.22954)
- **INMS / Memory Sharing**：多 agent 异步共享、过滤、存储与检索 memory pool，推动 collective self-enhancement。对产业级系统而言，共享记忆池可用于跨 agent/跨任务的策略迁移，但必须加入权限、污染控制和质量门禁。[arXiv:2404.09982](https://arxiv.org/abs/2404.09982)

## 5. 关键研究缺口

### 5.1 现有长期记忆偏“会话”，缺少高难任务轨迹语义

多数 memory benchmark 关注多会话事实、时间关系和用户偏好。高难算法任务中的关键记忆更像：

- 哪类状态签名意味着当前策略进入死胡同；
- 哪个中间不变量被破坏；
- 哪种搜索/剪枝/验证策略在相似任务上有效；
- 哪段局部修复 span 是必要且充分的；
- 哪个 PRM/verifier 信号能提前预测后续失败。

这要求从“事实记忆”扩展到“过程记忆 + 失败诊断记忆 + 策略记忆”。

### 5.2 现有反思偏自然语言总结，缺少证据闭环

Reflexion、ExpeL、Agent-Pro 都说明自然语言反思有效，但产业级系统不能只存“我下次应该更小心”。需要把每条反思绑定到：

- 原始任务与版本；
- 轨迹 span；
- 失败/成功 verifier；
- 反事实候选；
- 后续复用收益；
- 失效条件。

否则记忆库会逐渐变成不可验证的经验口号。

### 5.3 现有自我演化偏 agent/prompt/code 优化，缺少“记忆内容”科学

Symbolic Learning 与 DGM 关心如何让 agent 自身演化，但本课题要更细地回答：演化过程里的哪些中间产物应该进入长期记忆？建议把记忆内容拆成五层：

1. **Episodic span memory**：一次任务中某段状态-动作-反馈-修复片段。
2. **Diagnostic memory**：失败模式、触发条件、早期预警信号。
3. **Procedural memory**：可复用策略、prompt 规则、工具调用协议。
4. **Semantic memory**：任务域、工具、评估器、约束事实。
5. **Evolutionary archive memory**：agent 版本、变更、收益、回退条件。

### 5.4 记忆评估仍不充分

仅报告“检索准确率”不够。必须回答：记忆注入后是否真正提升任务结果？是否降低成本？是否减少长轨迹失败？是否在 out-of-distribution 相似任务上泛化？是否会因错误记忆导致负迁移？

## 6. 建议的技术路线

### 6.1 总体闭环

```text
原始轨迹日志
  -> 轨迹规范化
  -> 失败/转折/修复 span 定位
  -> 候选记忆抽取
  -> 证据校验与去重
  -> 多索引存储
  -> 任务时检索与注入
  -> 任务执行/修复
  -> 收益评估
  -> 记忆更新、降权或废弃
```

### 6.2 轨迹规范化

每条轨迹至少应统一到如下字段：

- `task`: 题目、输入输出约束、难度、标签、数据集来源、版本。
- `state`: 当前代码/草稿/工具状态、环境 observation、测试状态、上下文摘要。
- `action`: prompt、工具调用、代码修改、候选动作、选择理由。
- `feedback`: 编译/测试/评估器结果、PRM/RPM 分数、LLM judge、人工标注。
- `branch`: 候选分支、beam/search path、被拒绝动作和拒绝原因。
- `span`: 失败 anchor、修复开始/结束、minimality certificate。
- `agent_version`: prompt、toolset、policy、模型、参数、memory config。

### 6.3 记忆抽取策略

建议同时运行三类抽取器：

1. **规则/结构抽取器**：从日志字段直接抽取任务标签、错误类型、测试失败、工具调用序列、PRM 分数峰谷、span 长度等。
2. **LLM 归纳抽取器**：把局部 span 转为 `situation -> mistake -> repair -> reusable principle -> anti-condition`。
3. **对比抽取器**：比较成功分支与失败分支，抽取“哪个决策差异改变了结果”。这比单条成功轨迹摘要更有价值。

### 6.4 记忆对象 schema 草案

```json
{
  "memory_id": "mem_algo_span_000123",
  "memory_type": "episodic|diagnostic|procedural|semantic|evolutionary",
  "source": {
    "task_id": "dataset/task/version",
    "trajectory_id": "traj_abc",
    "span": {"start": 5, "end": 8, "minimal": true},
    "agent_version": "agent_v12",
    "created_at": "2026-04-28"
  },
  "task_signature": {
    "domain": "algorithm|tool-use|web|swe",
    "tags": ["dp", "state-compression", "off-by-one"],
    "difficulty": "hard",
    "input_constraints": ["n<=2e5", "multiple test cases"]
  },
  "state_signature": {
    "symptoms": ["passes samples but fails hidden edge cases"],
    "failed_tests": ["boundary empty segment", "large n timeout"],
    "prm_signals": {"pre_failure_drop": 0.31, "best_candidate_gap": 0.22}
  },
  "content": {
    "situation": "When the task combines interval DP with modulo constraints...",
    "mistake": "The failed span treated local optimal substructure as independent.",
    "repair": "Introduce state carrying the residue class and verify transition closure.",
    "reusable_strategy": "Before coding, enumerate invariant-carrying state variables and prove transition closure.",
    "anti_conditions": ["Do not apply when constraints allow brute-force enumeration."]
  },
  "evidence": {
    "verifier": "unit+hidden+replay",
    "before": {"pass_rate": 0.18, "cost_tokens": 42000},
    "after": {"pass_rate": 0.73, "cost_tokens": 29000},
    "confidence": 0.82
  },
  "retrieval": {
    "dense_text": "interval DP modulo invariant hidden edge case repair",
    "sparse_terms": ["interval dp", "modulo", "invariant", "hidden test"],
    "graph_entities": ["interval DP", "residue class", "transition closure"],
    "priority": "high"
  },
  "lifecycle": {
    "status": "active|quarantined|deprecated",
    "reuse_count": 7,
    "negative_transfer_count": 1,
    "last_validated_at": "2026-04-28"
  }
}
```

### 6.5 存储与索引设计

建议第一版不要追求复杂数据库，而是分层实现：

- **Artifact store**：原始轨迹、span、patch、评估结果用 content-addressed 文件或对象存储保存。
- **Metadata store**：SQLite/Postgres 保存任务签名、span 边界、版本、指标、生命周期。
- **Vector + sparse index**：dense embedding + BM25/keyword 双路检索，避免纯向量检索错过算法关键词。
- **Graph index（第二阶段）**：用于任务类型、失败模式、策略、工具、agent 版本之间的关系；参考 Zep/Graphiti 的 temporal KG 思路。
- **Prompt memory slots**：把检索结果注入到固定槽位，例如 `Relevant failure patterns`、`Reusable strategies`、`Do-not-repeat mistakes`、`Verified repair templates`。

### 6.6 检索与注入策略

任务执行时不要把所有相关记忆直接塞入上下文。建议使用两阶段：

1. **召回阶段**：基于任务描述、约束、当前状态、失败信号、agent 版本进行多路召回。
2. **筛选阶段**：由小模型/规则/verifier 对候选记忆排序，剔除过期、冲突、低证据或明显不适用的记忆。

注入时应区分：

- **任务前注入**：procedural/semantic memory，帮助初始解法规划。
- **失败后注入**：diagnostic/episodic memory，帮助定位相似 failure。
- **搜索中注入**：value/prior memory，帮助 beam/MCTS/repair search 选分支。
- **演化后写回**：把本次结果更新到 memory evidence，而不是无条件新增。

## 7. 系统性评估方案

### 7.1 数据与任务分层

建议三层数据集：

1. **受控层**：伐谋 `dependent_tools`，用于验证 span 抽取、minimality、记忆收益的因果性。
2. **外部 replay 层**：`tau2` 或类似长程工具任务，用于测试记忆在真实环境中的鲁棒性。
3. **高难算法层**：HumanEval/MBPP 只适合做 sanity；更建议加入 APPS、CodeContests/Codeforces 风格题、或内部 hard algorithm tasks，以覆盖复杂状态、边界条件和多轮修复。

### 7.2 Baseline

至少设置以下对照：

- `B0 no-memory`：没有长期记忆，仅使用当前上下文。
- `B1 full-context/raw-log`：检索原始相似轨迹或长摘要。
- `B2 reflection-only`：类似 Reflexion，只注入自然语言反思。
- `B3 few-shot episodic`：注入相似成功/失败案例。
- `B4 structured-memory`：注入本课题 schema 化记忆。
- `B5 oracle-memory`：人工/规则选取最相关记忆，估计上界。

### 7.3 指标

任务结果指标：

- `Pass@1 / Pass@k`
- 首次成功所需 token、工具调用、wall time
- 失败恢复率、平均修复 span 长度
- hidden tests / replay verifier 通过率

记忆质量指标：

- 记忆抽取 precision/recall：是否覆盖关键失败/策略点
- retrieval hit@k：是否召回真正有用的记忆
- memory utility：注入该记忆后的边际收益
- negative transfer rate：错误记忆导致性能下降的比例
- staleness：agent/tool/task 版本变化后记忆失效率

演化指标：

- agent 版本随迭代的能力曲线
- memory bank 增长速度 vs 有效记忆比例
- 不同 memory type 的 ablation 收益
- 跨任务、跨难度、跨 benchmark 的迁移收益

### 7.4 推荐实验矩阵

| 实验 | 目的 | 方法 | 成功标准 |
| --- | --- | --- | --- |
| E0 schema audit | 验证日志是否足够支撑记忆抽取 | 对已有轨迹做字段覆盖率统计 | 关键字段覆盖率 > 90% |
| E1 span-to-memory | 验证 minimal span 是否比整轨迹摘要更好 | span memory vs full trajectory summary | structured span 在 hit@k 和 Pass@1 上更优 |
| E2 failure diagnostic | 验证失败模式记忆是否能提前预警 | 根据状态/PRM 信号预测失败 anchor | anchor/失败类型 top-k 提升 |
| E3 retrieval injection | 验证记忆注入收益 | no-memory vs raw retrieval vs structured memory | Pass@1 提升且成本不过度增加 |
| E4 negative transfer | 检验错误/过期记忆危害 | 人为加入相似但错误记忆 | 筛选器能降权或拒绝 |
| E5 self-evolution loop | 验证长期迭代收益 | 每轮任务后更新 memory bank | 多轮性能曲线持续提升或稳定 |
| E6 cross-benchmark | 验证泛化 | dependent_tools 训练，tau2/算法任务测试 | 至少部分 memory type 正迁移 |

## 8. 论文/项目创新点可表述方式

建议把创新点收紧为三个：

1. **Memory object innovation**：提出面向高难任务自我演化的 `verifier-grounded trajectory memory`，以最小可修复子轨迹为证据锚点。
2. **Memory pipeline innovation**：从状态轨迹、行为选择、PRM/verifier 反馈中抽取 diagnostic、episodic、procedural、evolutionary 多层记忆，并支持检索、注入、更新、降权。
3. **Evaluation innovation**：不只评估记忆检索，而是通过未来任务收益、失败恢复效率、负迁移率和跨 benchmark 泛化评估“可用记忆”。

不建议把创新点写成“我们用了向量数据库/RAG”或“我们让 agent 有长期记忆”。这些表述太泛，且容易被现有 LangGraph/Mem0/Zep/Letta 工作覆盖。

## 9. 第一版实施路线图

### Phase 0：仓库与文档固化（1-2 天）

- 固化 research report、schema 草案、实验 protocol。
- 明确第一批轨迹数据来源和字段覆盖率。
- 定义 memory object 的 JSON schema 与 validation script。

### Phase 1：离线轨迹挖掘（1-2 周）

- 从伐谋现有日志中抽取任务、状态、动作、反馈、span。
- 生成 `episodic span memory` 与 `diagnostic memory`。
- 对每条记忆绑定原始证据和 verifier 结果。
- 先不做在线写入，避免系统复杂度过高。

### Phase 2：记忆检索与注入（2-3 周）

- 实现 BM25 + embedding 双路检索。
- 实现 memory ranker：任务相似度、状态相似度、证据置信度、版本兼容性。
- 在受控任务上做 B0-B4 baseline 对照。

### Phase 3：自我演化闭环（3-5 周）

- 每次任务后根据结果更新 memory evidence。
- 引入 negative transfer logging。
- 加入 procedural memory 更新：把多次成功的 span memory 归纳为策略规则。
- 评估多轮任务序列上的能力曲线。

### Phase 4：产业级可靠性（长期）

- 记忆权限、隔离、审计、回滚。
- 记忆污染检测与 quarantine。
- 跨 agent 共享池与版本化治理。
- 高成本任务下的 latency/token budget 优化。

## 10. 风险与应对

| 风险 | 表现 | 应对 |
| --- | --- | --- |
| 记忆过度泛化 | 错误策略被复用于不适用任务 | 强制记录 anti-conditions 与 negative transfer |
| 记忆污染 | 失败经验或 hallucinated reflection 进入高优先级记忆 | 记忆写入必须绑定 verifier 或 replay 证据 |
| 检索不准 | 相似文本但不同算法本质被召回 | 加入结构标签、失败模式、约束签名、稀疏关键词 |
| 成本膨胀 | 记忆抽取/检索/注入增加 token 和延迟 | background extraction；top-k 限制；只注入压缩 memory |
| 版本失效 | agent prompt/tool 变化后旧记忆不再适用 | 记忆绑定 agent/tool/environment version |
| 评估不闭环 | 只展示案例，没有系统指标 | 固化 B0-B5 baseline 与 E0-E6 实验矩阵 |

## 11. 推荐近期产出

1. `schema/memory_item.schema.json`：正式 JSON schema。
2. `scripts/extract_memories_from_trajectory.py`：从轨迹 JSONL 生成候选 memory。
3. `scripts/evaluate_memory_injection.py`：跑 no-memory/raw/structured 三组对照。
4. `docs/memory_taxonomy.md`：定义本课题的 memory types、写入规则、过期规则。
5. `docs/experiment_protocol.md`：固定数据切分、指标、baseline、统计口径。

## 12. 参考资料

- Park et al., **Generative Agents: Interactive Simulacra of Human Behavior**, 2023. https://arxiv.org/abs/2304.03442
- Shinn et al., **Reflexion: Language Agents with Verbal Reinforcement Learning**, 2023. https://arxiv.org/abs/2303.11366
- Madaan et al., **Self-Refine: Iterative Refinement with Self-Feedback**, 2023. https://arxiv.org/abs/2303.17651
- Yao et al., **Tree of Thoughts: Deliberate Problem Solving with Large Language Models**, 2023. https://arxiv.org/abs/2305.10601
- Wang et al., **Voyager: An Open-Ended Embodied Agent with Large Language Models**, 2023. https://arxiv.org/abs/2305.16291
- Zhao et al., **ExpeL: LLM Agents Are Experiential Learners**, 2023/2024. https://arxiv.org/abs/2308.10144
- Sumers et al., **Cognitive Architectures for Language Agents**, 2023/2024. https://arxiv.org/abs/2309.02427
- Packer et al., **MemGPT: Towards LLMs as Operating Systems**, 2023/2024. https://arxiv.org/abs/2310.08560
- Zhou et al., **Language Agent Tree Search Unifies Reasoning Acting and Planning in Language Models**, 2023/2024. https://arxiv.org/abs/2310.04406
- Zhang et al., **Agent-Pro: Learning to Evolve via Policy-Level Reflection and Optimization**, ACL 2024. https://aclanthology.org/2024.acl-long.292/
- Gao and Zhang, **INMS: Memory Sharing for Large Language Model based Agents**, 2024/2026. https://arxiv.org/abs/2404.09982
- Zhou et al., **Symbolic Learning Enables Self-Evolving Agents**, 2024. https://arxiv.org/abs/2406.18532
- Wu et al., **LongMemEval: Benchmarking Chat Assistants on Long-Term Interactive Memory**, ICLR 2025. https://arxiv.org/abs/2410.10813
- Chhikara et al., **Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory**, 2025. https://arxiv.org/abs/2504.19413
- Zhang et al., **Darwin Godel Machine: Open-Ended Evolution of Self-Improving Agents**, 2025/2026. https://arxiv.org/abs/2505.22954
- Hu et al., **Memory in the Age of AI Agents**, 2025/2026. https://arxiv.org/abs/2512.13564
- LangGraph Memory documentation. https://docs.langchain.com/oss/python/concepts/memory
- Letta Stateful Agents documentation. https://docs.letta.com/guides/core-concepts/stateful-agents
- Zep Graph Overview documentation. https://help.getzep.com/graph-overview
- Mem0 GitHub README. https://github.com/mem0ai/mem0
