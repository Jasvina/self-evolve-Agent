# Self-Evolving Agent 调研笔记

> 截至 2026-05。围绕"自进化 / 自改进"的 LLM Agent 的最新综述与代表作。
> 论文原文 PDF 见 [../papers/](../papers/)。

## 0. TL;DR

- LLM 部署后参数静态，但任务、环境、知识在变 — 这是"自进化"的核心动机。
- 2025–2026 出现了多篇系统综述，把这条线归并到统一框架下：可演化的对象（model / memory / tools / architecture）× 何时演化（test-time / offline / lifelong）× 如何演化（in-context / SFT / RL / self-edit）。
- 工程上能落地的范式可大致归为 4 类：① 经验回放 ② Skill 固化 ③ 蒸馏到小模型 ④ Self-Edit / 元认知。

---

## 1. 必读综述

### 1.1 A Survey of Self-Evolving Agents: What, When, How, and Where to Evolve on the Path to ASI

- **arXiv**: [2507.21046](https://arxiv.org/abs/2507.21046)（2025-07 首版，2026-01 v4，TMLR 2026）
- **本地**: `papers/2507.21046_survey_self_evolving_agents.pdf`
- **价值**: 目前最系统、覆盖最全的一篇。把"自进化"拆成四维度：
  - **What** to evolve: model parameters / memory / tools / architecture
  - **When** to evolve: test-time / offline batch / lifelong
  - **How** to evolve: in-context learning / SFT / RLHF / Self-modification
  - **Where** to apply: single-agent / multi-agent / agent-environment co-evolution
- **对你三分类的映射**:
  - 经验回放 → 在 *Memory* + *Test-time in-context* 这一格
  - Skill 固化 → 在 *Memory / Tools* + *Offline + Lifelong* 这一格
  - 蒸馏到小模型 → 在 *Model* + *Offline SFT/RL* 这一格
- **配套资源**: [Awesome-Self-Evolving-Agents (XMUDeepLIT)](https://github.com/XMUDeepLIT/Awesome-Self-Evolving-Agents)

### 1.2 A Comprehensive Survey of Self-Evolving AI Agents — Bridging Foundation Models and Lifelong Agentic Systems

- **arXiv**: [2508.07407](https://arxiv.org/abs/2508.07407)（Fang et al., 2025）
- **本地**: `papers/2508.07407_comprehensive_survey_self_evolving_ai_agents.pdf`
- **价值**: 提出统一反馈环框架 `System Inputs → Agent System → Environment → Optimizer`；提出 Asimov 风格的三层原则 **Endure（安全适应）/ Excel（性能不退化）/ Evolve（自主演化）**。
- **配套资源**: [EvoAgentX 框架 + Awesome list](https://github.com/EvoAgentX/Awesome-Self-Evolving-Agents)

### 1.3 辅助综述

| 综述 | arXiv | 切入角度 |
|---|---|---|
| A Survey on the Optimization of LLM-based Agents | [2503.12434](https://arxiv.org/abs/2503.12434) | Optimization 视角；轨迹构造、反馈式自修正、多 agent 协作训练 |
| The Landscape of Agentic Reinforcement Learning for LLMs | [2509.02547](https://arxiv.org/abs/2509.02547) | 500+ 篇 agentic RL 综述；planning / tool use / memory / self-improvement 各自成章 |
| Agent Skills from the Perspective of Procedural Memory | [TechRxiv](https://www.techrxiv.org/users/1016212/articles/1376445) | 专门讲 skill induction & procedural memory 一支 |
| A Systematic Survey of Self-Evolving Agents: From Model-Centric to Environment-Driven Co-Evolution | TechRxiv 2026-02 | 把"模型中心进化"与"环境驱动共进化"对立起来讨论 |

---

## 2. 范式 ① 记忆检索 / 经验回放（Training-Free）

不动模型参数，只改"上下文"或"外部记忆"。最容易落地，也是 2025 年最热的子方向。

### 2.1 Contextual Experience Replay (CER)
- **arXiv**: [2506.06698](https://arxiv.org/abs/2506.06698)
- **本地**: `papers/2506.06698_contextual_experience_replay.pdf`
- **要点**: 把过往轨迹（环境动态 + 决策模式）累积、合成进动态记忆缓冲，在新任务里检索注入上下文。完全 training-free。
- **适用**: Web 导航、长程交互任务。

### 2.2 Self-Generated In-Context Examples（NeurIPS 2025, Sarukkai et al.）
- **要点**: agent 一旦成功解一个任务，就把完整成功轨迹存下；下次直接 few-shot 进 prompt。ALFWorld 73% → 89%（部分设定 → 93%），不调权重就能逼近大模型。
- **取舍**: 实现极简，但上下文长度有限，需要 curation 策略。

### 2.3 ExpeL（Zhao et al., 2024）
- **要点**: 轨迹蒸馏成可编辑的自然语言 *insights*，按相关性检索注入。后续大量工作的 baseline。

### 2.4 EvolveR（ICLR 2026）
- **OpenReview**: <https://openreview.net/forum?id=sooLoD9VSf>
- **要点**: 完整的"经验生命周期"：
  - **Offline Self-Distillation**: 把轨迹综合成一个 *abstract reusable strategic principles* 的库；
  - **Online Interaction**: 主动检索 principles 指导决策；
  - 与 RL 更新结合，形成闭环。

### 2.5 MUSE（Yang et al., 2025）
- **要点**: Plan-Execute-Reflect-Memorize 四阶段，建分层记忆（strategic / procedural / tool-use）。

### 2.6 反例 / 警示
- **SPEAR**（arXiv:2509.22601）: 朴素 experience replay 会触发 **entropy collapse** 和 **exploration shrinkage**，尤其在 1.5B–7B 小模型上。引入 curriculum + covariance clipping 才能缓解。
- **Misevolution / "Your agent may misevolve"** (ICLR 2026): 自进化的安全风险。

---

## 3. 范式 ② Skill 固化（Skill Induction & Library Evolution）

从 Voyager（2023）的固定 skill library 出发，2025 年的关键演进是 **skill 集合本身也在演化**。

### 3.1 MemSkill（arXiv:2602.02474）
- **要点**: skill 选择策略 + skill 集本身 *联合优化*。设计器周期性 review 失败案例，提出新的 skill 或精炼旧 skill。
- **评测**: LoCoMo / LongMemEval / HotpotQA / ALFWorld。

### 3.2 SkillWeaver（Zheng et al., 2025）
- **要点**: 在 web agent 上自动发现并精炼可复用 skill；和 MemSkill 是同一思路的 web 落地。

### 3.3 Voyager（2023, 开山之作）
- **要点**: 在 Minecraft 中按"自然语言函数"积累 skill library，被几乎所有后续工作引为 baseline。

### 3.4 Agentic Context Engineering (ACE)
- **要点**: 把"上下文"当成一份不断累积的 *playbook*；模块化增量更新，可用于离线 prompt 优化或在线记忆适配。这条线把 skill 看作"可执行的 playbook 片段"。

---

## 4. 范式 ③ RL / 蒸馏到小模型（让小模型对齐大模型行为）

把强模型的完整 agentic 轨迹（不止 CoT，还包括工具调用）蒸馏进 sLM。

### 4.1 Distilling LLM Agent into Small Models with Retrieval and Code Tools
- **arXiv**: [2505.17612](https://arxiv.org/abs/2505.17612)（Kang et al., NeurIPS 2025）
- **本地**: `papers/2505.17612_distilling_llm_agent_into_small_models.pdf`
- **代码**: <https://github.com/Nardien/agent-distillation>
- **要点**: 0.5B / 1.5B / 3B 学生模型，配 retrieval + code tool，达到 1.5B / 3B / 7B CoT 蒸馏模型水平。两个核心 trick：
  - **First-thought prefix**: 让 teacher 生成更高质量的轨迹起点
  - **Self-consistent action**: 提升 test-time 鲁棒性

### 4.2 Structured Agent Distillation
- **arXiv**: [2505.13820](https://arxiv.org/abs/2505.13820)
- **本地**: `papers/2505.13820_structured_agent_distillation.pdf`
- **要点**: 把轨迹切成 `[REASON]` / `[ACT]` 两类 span，对各自施加 span-specific loss；比 token-level 蒸馏更稳。ALFWorld / HotpotQA-ReAct / WebShop 上一致优于 baseline。
- **核心论点**: token-level 蒸馏让学生学会表面动作但学不到背后 rationale。

### 4.3 Search, Do not Guess（arXiv:2604.04651）
- **要点**: 朴素蒸馏会把"参数化幻觉"也学过去（小模型从教师轨迹里学到"我记得 X"这种伪知识）。解法：always-search 策略 + 阈值过滤，只保留真的调用工具的轨迹。

### 4.4 On-Policy Distillation（Thinking Machines Lab, 2025）
- **链接**: <https://thinkingmachines.ai/blog/on-policy-distillation/>
- **要点**: 在线策略蒸馏，复现 Qwen3 的推理性能但成本远低于纯 RL；强调 on-policy 比 off-policy SFT 在持续学习场景下遗忘更少。

### 4.5 AgentArk（arXiv:2602.03955）
- **要点**: 把 multi-agent debate 蒸馏成单 agent 的 single forward pass。三招组合：R-SFT（共识训练） + 轨迹多样化扩增 + Process-aware reward model。

### 4.6 Self-Improving LLM Agents at Test-Time
- **arXiv**: [2510.07841](https://arxiv.org/abs/2510.07841)
- **本地**: `papers/2510.07841_self_improving_llm_agents_test_time.pdf`
- **要点**: 两套机制 —
  - **TT-SI**: 模型对自己 uncertain 的样本生成补充训练数据，自训自学；
  - **TT-D**: 由更强模型生成类似样本做蒸馏。
- TT-SI 平均 +5.48% 精度，且训练样本仅为常规方法的 1/68。

---

## 5. 范式 ④ Self-Edit / 元认知（你列表里漏掉的一类）

不是改记忆、也不是改权重，而是改 *agent 自身的代码 / prompt / heuristic*。

### 5.1 SICA: Self-Improving Coding Agent（Robeyns et al., 2025）
- **要点**: agent 评估自己在 benchmark 上的表现 → 用 LLM 提议修改自己的源码 / prompt → 应用并 re-evaluate → 保留有改进的。SWE-Bench 类任务上 17–53% 提升，部分场景成本/时间也下降。

### 5.2 Intrinsic Metacognitive Learning（Liu et al., ICML 2025 position）
- **要点**: 主张当前所有"反思类"工作只是 *self-improvement 1.0* — 任务级。真正的 self-improving agent 需要：
  - **Metacognitive knowledge**: 准确自评估
  - **Metacognitive planning**: 决定下一步学什么、怎么学
  - **Metacognitive evaluation**: 判断学习是否真的有效
- 这是当前 SOTA 的清晰差距。

### 5.3 Misevolution risks（ICLR 2026, "Your Agent May Misevolve"）
- **要点**: 自进化引入的新型安全风险（恶意 skill 累积、错误 insight 放大、对抗性轨迹注入等）。做自进化 agent 早晚要看。

---

## 6. 评测 benchmark（不绕过）

| Benchmark | 关注点 |
|---|---|
| **MemBench** (Tan et al., 2025) | factual vs reflective memory，三维：effectiveness / efficiency / capacity |
| **MemoryAgentBench** (Hu et al., 2025) | 四类记忆能力：accurate retrieval / test-time learning / long-range understanding / **selective forgetting**（当前系统普遍跪在最后一项） |
| **Evo-Memory** | 流式 test-time memory evolution 基准 |
| **ALFWorld / WebShop / HotPotQA-ReAct** | 经验回放 + skill induction + 蒸馏类工作的共同试金石 |

---

## 7. 建议阅读顺序

1. **第 1 天**：读 `2507.21046` 第 3–5 节，建立分类骨架。
2. **第 2 天**：读 `2508.07407` 的统一反馈环框架，对照自己的三分类做加法。
3. **第 3–4 天**：每个范式精读 2 篇代表作：
   - 经验回放：`2506.06698` (CER) + EvolveR (OpenReview)
   - Skill：MemSkill + ACE
   - 蒸馏：`2505.17612` + `2505.13820`
   - Self-Edit：SICA 论文 + Liu et al. ICML 2025 position paper
4. **第 5 天**：扫 [Awesome-Self-Evolving-Agents](https://github.com/XMUDeepLIT/Awesome-Self-Evolving-Agents) README 表格补漏，定向跟踪 ICLR 2026 / NeurIPS 2025 新论文。

---

## 8. 待跟踪

- ICLR 2026 评审中关于自进化安全（"misevolve"）的几篇论文最终版
- EvoAgentX 框架（[GitHub](https://github.com/EvoAgentX/Awesome-Self-Evolving-Agents)）的实际复现质量
- On-policy distillation 是否会替代 PPO / GRPO 成为蒸馏 agent 的默认方案
