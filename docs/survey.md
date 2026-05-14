# Self-Evolving Agent 调研笔记

> 最近一次更新: 2026-05。围绕"自进化 / 自改进"的 LLM Agent 的最新综述与代表作。
> 论文原文 PDF 见 [../papers/](../papers/)。

## 0. TL

- LLM 部署后参数静态，但任务、环境、知识在变 — 这是"自进化"的核心动机。
- 2025–2026 出现了多篇系统综述，把这条线归并到统一框架下：可演化的对象（model / memory / tools / architecture）× 何时演化（test-time / offline / lifelong）× 如何演化（in-context / SFT / RL / self-edit）。
- 工程上能落地的范式大致 4 类：① 经验回放 ② Skill 固化 ③ 蒸馏到小模型 ④ Self-Edit / 元认知。
- **2026 的趋势**：从"agent 单向自进化"转向 **agent ↔ 数据 / 环境 / skill 库的双向共进化**；从"人造 reward 监督"转向 **reward-free 内禀进化**；从"raw trajectory 存档"转向 **可执行的层次化 skill 库**。

---

## 1. 必读综述

### 1.1 A Survey of Self-Evolving Agents: What, When, How, and Where to Evolve on the Path to ASI

- **arXiv**: [2507.21046](https://arxiv.org/abs/2507.21046)（2025-07 首版，2026-01 v4，TMLR 2026）
- **本地**: `papers/2507.21046_survey_self_evolving_agents.pdf`（77 页）
- **价值**: 目前最系统、覆盖最全的一篇。把"自进化"拆成四维度：
  - **What** to evolve: model / memory / tools / architecture
  - **When** to evolve: test-time / offline batch / lifelong
  - **How** to evolve: in-context / SFT / RLHF / Self-modification
  - **Where** to apply: single-agent / multi-agent / agent-environment co-evolution
- **配套**: [Awesome-Self-Evolving-Agents (XMUDeepLIT)](https://github.com/XMUDeepLIT/Awesome-Self-Evolving-Agents)

### 1.2 A Comprehensive Survey of Self-Evolving AI Agents — Bridging Foundation Models and Lifelong Agentic Systems

- **arXiv**: [2508.07407](https://arxiv.org/abs/2508.07407)（Fang et al., 2025）
- **本地**: `papers/2508.07407_comprehensive_survey_self_evolving_ai_agents.pdf`（55 页）
- **价值**: 提出统一反馈环框架 `System Inputs → Agent System → Environment → Optimizer`；提出三层原则 **Endure / Excel / Evolve**。
- **配套**: [EvoAgentX 框架](https://github.com/EvoAgentX/Awesome-Self-Evolving-Agents)

### 1.3 辅助综述

| 综述 | arXiv | 切入角度 |
|---|---|---|
| A Survey on the Optimization of LLM-based Agents | [2503.12434](https://arxiv.org/abs/2503.12434) | Optimization 视角；轨迹构造、反馈式自修正、多 agent 协作训练 |
| The Landscape of Agentic Reinforcement Learning for LLMs | [2509.02547](https://arxiv.org/abs/2509.02547) | 500+ 篇 agentic RL 综述；planning / tool use / memory / self-improvement 各自成章 |
| Agent Skills from the Perspective of Procedural Memory | [TechRxiv](https://www.techrxiv.org/users/1016212/articles/1376445) | 专门讲 skill induction & procedural memory 一支 |
| A Systematic Survey of Self-Evolving Agents: Model-Centric to Environment-Driven Co-Evolution | TechRxiv 2026-02 | "Model-Centric / Environment-Centric / Model-Env Co-Evolution"三分法 |

---

## 2. 范式 ① 记忆检索 / 经验回放（Training-Free）

不动模型参数，只改"上下文"或"外部记忆"。最容易落地。

### 2.1 Contextual Experience Replay (CER)
- **arXiv**: [2506.06698](https://arxiv.org/abs/2506.06698) · **本地**: `papers/2506.06698_contextual_experience_replay.pdf`
- **要点**: 把过往轨迹（环境动态 + 决策模式）累积、合成进动态记忆缓冲，新任务里检索注入上下文。training-free。

### 2.2 EvolveR: Self-Evolving LLM Agents through an Experience-Driven Lifecycle ⭐ 2026
- **arXiv**: [2510.16079](https://arxiv.org/abs/2510.16079)（2025-10 首版，2026-02 ICLR 2026）
- **本地**: `papers/2510.16079_evolver_experience_lifecycle.pdf`
- **代码**: <https://github.com/Edaizi/EvolveR>
- **要点**: 论文给的范式四分法本身就是个很好的 mental model：
  - Stateless Execution（标准 agent，扔掉经验）
  - Learning by Raw Trajectories（存原始轨迹再检索 → ExpeL 一支）
  - Learning via External Scribing（靠外部 teacher 蒸馏 insight）
  - **EvolveR**: 完整闭环 — agent 自己离线把轨迹蒸馏成抽象 *strategic principles*，在线检索调用 + RL 更新策略。

### 2.3 Self-Generated In-Context Examples（NeurIPS 2025, Sarukkai et al.）
- **要点**: 成功轨迹存下做 few-shot。ALFWorld 73% → 89%（部分 → 93%）。

### 2.4 ExpeL（Zhao et al., 2024）
- **要点**: 轨迹蒸馏成自然语言 *insights*，按相关性检索注入。后续 baseline 之一。

### 2.5 MUSE（Yang et al., 2025）
- **要点**: Plan-Execute-Reflect-Memorize 四阶段 + 分层记忆（strategic / procedural / tool-use）。

### 2.6 ERL — Experiential Reflective Learning（ICLR 2026 MemAgents Workshop）⭐ 2026
- **要点**: 构建"启发式池"作为经验记忆；在新环境下 +7.8% 相对 ReAct，56.1% 总体成功率。

### 2.7 反例 / 警示
- **SPEAR**（arXiv:2509.22601）: 朴素 replay 会引发 entropy collapse / exploration shrinkage。
- **Misevolution**（arXiv:2509.26354, ICLR 2026）: 自进化过程的紧急安全风险。
- **Alignment Tipping Process**（arXiv:2510.04860）: 自进化把 agent 推离对齐的机理分析。

---

## 3. 范式 ② Skill 固化（Skill Induction & Library Evolution）

**2026 这一支爆发了**：从 Voyager 固定 skill library，演进到"skill 集合本身 + 选择策略 + verifier 三者联合进化"。

### 3.1 MemSkill: Learning and Evolving Memory Skills ⭐ 2026
- **arXiv**: [2602.02474](https://arxiv.org/abs/2602.02474)（2026-02）
- **本地**: `papers/2602.02474_memskill.pdf`
- **要点**: 把"记忆操作"（extract / consolidate / prune）当成可学习的 *memory skill*。controller 选 skill，executor 执行，designer 周期性 review 失败案例 → 提出新 skill / 精炼旧 skill。
- **评测**: LoCoMo / LongMemEval / HotpotQA / ALFWorld。

### 3.2 SkillRL: Recursive Skill-Augmented RL ⭐ 2026
- **arXiv**: [2602.08234](https://arxiv.org/abs/2602.08234)（2026-02）
- **本地**: `papers/2602.08234_skillrl_recursive_skill_rl.pdf`
- **要点**: 三件套：
  - *Experience-based distillation* 建分层 **SkillBank**
  - 自适应检索（区分通用 vs 任务特定启发式）
  - **Recursive evolution**: skill 库与 RL 策略**共同进化**
- **意义**: 把 skill induction 第一次正式嵌进 RL 训练循环。

### 3.3 EvoSkills: Co-Evolutionary Verification ⭐ 2026
- **arXiv**: [2604.01687](https://arxiv.org/abs/2604.01687)（2026-04）
- **本地**: `papers/2604.01687_evoskills_co_evolutionary_verification.pdf`
- **要点**: Skill Generator + **Surrogate Verifier** 共同进化。Verifier 在没有 ground-truth test 的前提下给出可用反馈，agent 由此构造多文件 skill 包。
- **评测**: SkillsBench；在 Claude Code / Codex 上都是五个 baseline 中最高 pass rate；可泛化到另外 6 个 LLM。

### 3.4 SkillOS: Learning Skill Curation ⭐ 2026
- **arXiv**: [2605.06614](https://arxiv.org/abs/2605.06614)（2026-05）
- **本地**: `papers/2605.06614_skillos_skill_curation.pdf`
- **要点**: 把"skill curation（如何挑、合并、淘汰 skill）"本身当成 RL 的学习目标。SkillRepo 中的 skill 演化成结构化 Markdown，编码高阶 meta-skill。

### 3.5 SkillWeaver / Voyager（baseline）
- Voyager (2023): 固定 skill library 开山作。
- SkillWeaver (Zheng et al., 2025): web agent 自动发现 + 精炼可复用 skill。

---

## 4. 范式 ③ RL / 蒸馏到小模型（让小模型对齐大模型行为）

### 4.1 Distilling LLM Agent into Small Models with Retrieval and Code Tools
- **arXiv**: [2505.17612](https://arxiv.org/abs/2505.17612)（NeurIPS 2025）· **本地**: `papers/2505.17612_distilling_llm_agent_into_small_models.pdf`
- **代码**: <https://github.com/Nardien/agent-distillation>
- **要点**: 0.5B / 1.5B / 3B 学生 + retrieval + code tool，达到 1.5B / 3B / 7B CoT 蒸馏水平。两 trick：first-thought prefix + self-consistent action。

### 4.2 Structured Agent Distillation
- **arXiv**: [2505.13820](https://arxiv.org/abs/2505.13820) · **本地**: `papers/2505.13820_structured_agent_distillation.pdf`
- **要点**: 轨迹切 `[REASON]` / `[ACT]` 两类 span，分别施加 loss。优于 token-level 蒸馏。

### 4.3 CoEvolve: Agent-Data Mutual Evolution ⭐ 2026
- **arXiv**: [2604.15840](https://arxiv.org/abs/2604.15840)（2026-04）
- **本地**: `papers/2604.15840_coevolve_agent_data_mutual_evolution.pdf`
- **要点**: 解决"static 数据分布跟不上 agent 行为演化"的问题。从 rollout 提取 *forgetting* 和 *uncertainty* 信号识别失败模式 → LLM 合成新任务 → 环境验证 → 更新数据分布。
- **结果**: AppWorld + BFCL 在 Qwen2.5-7B / Qwen3-4B / Qwen3-30B-A3B 上分别 +19.43% / +15.58% / +18.14% 绝对提升。

### 4.4 Reward-Free Self-Evolution via World Knowledge Exploration ⭐ 2026
- **arXiv**: [2604.18131](https://arxiv.org/abs/2604.18131)（2026-04）
- **本地**: `papers/2604.18131_reward_free_self_evolution_world_knowledge.pdf`
- **要点**: 主张当前 agent 的 "self-evolution" 仍依赖外部 reward — 没有人类信号就停摆。设计一个 outcome-based reward：以"自生成 world knowledge 是否提升下游成功率"为目标，训练 agent 在没见过的环境里**主动探索 + 自总结**，且这套 reward 只在训练阶段用。
- **意义**: 朝"内禀元进化"迈了一步，呼应 Liu et al. ICML 2025 的 metacognitive learning 主张。

### 4.5 AutoTTS: LLMs Improving LLMs ⭐ 2026
- **arXiv**: [2605.08083](https://arxiv.org/abs/2605.08083)（2026-05）
- **本地**: `papers/2605.08083_autotts_llms_improving_llms.pdf`
- **要点**: Test-time scaling (TTS) 通常靠人工设计策略。AutoTTS 改成"环境驱动"框架，让 TTS 策略**被自动发现**。数学推理 benchmark 上发现的策略迁移到新 benchmark / 新模型规模仍有效；整套发现仅花 \$39.9 / 160 分钟。

### 4.6 其他
- **Self-Improving LLM Agents at Test-Time** [2510.07841](https://arxiv.org/abs/2510.07841)（本地有）: TT-SI / TT-D，+5.48%，训练样本 1/68。
- **Search, Do not Guess**（arXiv:2604.04651, 2026-04）: 解决朴素蒸馏让小模型继承"参数化幻觉"的问题；强制 always-search 策略 + 阈值过滤轨迹。
- **On-Policy Distillation**（[Thinking Machines blog, 2025](https://thinkingmachines.ai/blog/on-policy-distillation/)）: on-policy 蒸馏复现 Qwen3 推理性能，成本远低于纯 RL。
- **AgentArk**: multi-agent debate → single-pass agent 蒸馏。

---

## 5. 范式 ④ Self-Edit / 元认知 / 多 agent 自组织

### 5.1 SICA: Self-Improving Coding Agent（Robeyns et al., 2025）
- agent 自评 → 用 LLM 改自己的源码/prompt/heuristic → 应用 → 再评。SWE-Bench 类任务 +17–53%。

### 5.2 Intrinsic Metacognitive Learning（Liu et al., ICML 2025 position）
- 当前所有"反思类"工作只是 self-improvement 1.0；真正自进化需要 metacognitive knowledge / planning / evaluation 三件套。

### 5.3 OneManCompany (OMC): From Skills to Talent ⭐ 2026
- **arXiv**: [2604.22446](https://arxiv.org/abs/2604.22446)
- **要点**: multi-agent 系统不是"叠 skill"，而是模仿公司**自我组织** — 自己招聘、重构、审查工作。基本单元是 "role"，不是 model 或 skill。

### 5.4 Self-Healing Framework for Reliable LLM-Based Agents ⭐ 2026
- **arXiv**: [2605.06737](https://arxiv.org/abs/2605.06737)
- **要点**: 集成 failure detection + reliability assessment + 自动恢复（adaptive replanning + corrective prompting）。比"自进化"更偏"自修复"，但属于 lifelong agent 的同一谱系。

### 5.5 Agentic Context Engineering (ACE)
- 上下文当 *playbook*，模块化增量更新。

---

## 6. 评测 benchmark

| Benchmark | 关注点 |
|---|---|
| **MemBench** (Tan et al., 2025) | factual vs reflective memory；effectiveness / efficiency / capacity 三维 |
| **MemoryAgentBench** (Hu et al., 2025) | 四类记忆能力，**selective forgetting** 普遍跪 |
| **StuLife** ⭐ 2026 | Experience-Driven Lifelong Learning (ELL) 基准，强调长程记忆 + 自驱探索 + 内化 |
| **SkillsBench** ⭐ 2026 | EvoSkills 提出；评估多文件 skill 包的 pass rate |
| **Evo-Memory** | 流式 test-time memory evolution 基准 |
| **ALFWorld / WebShop / HotPotQA-ReAct / AppWorld / BFCL** | 经典试金石 |

---

## 7. 建议阅读顺序（更新版）

1. **D1 骨架**: `2507.21046` §3–§5 + `2508.07407` 反馈环框架（2 篇综述）。
2. **D2 经验回放支**: `2506.06698` CER + `2510.16079` EvolveR（看清"从原始轨迹检索 → principle 蒸馏"的演进）。
3. **D3 Skill 支（2026 集中爆发）**: 读 SkillRL → MemSkill → EvoSkills → SkillOS 一条线，能完整看出 "skill 库 + 选择策略 + verifier 三者协同进化" 的成型过程。
4. **D4 蒸馏 & RL 支**: `2505.17612` + `2505.13820` 打底，CoEvolve（2604.15840）看数据共进化，Reward-Free Self-Evolution（2604.18131）看 reward-free 方向。
5. **D5 元认知 + 安全**: SICA + Liu et al. ICML 2025 + Misevolution / Alignment Tipping 两篇风险论文。
6. **持续跟踪**: ICLR 2026 Lifelong Agent (LLA) workshop 和 MemAgents workshop；Awesome list 表格补遗。

---

## 8. 关键趋势观察（2025 vs 2026）

| 维度 | 2025 主流 | 2026 演进 |
|---|---|---|
| 记忆形态 | raw trajectory + 自然语言 insight | **层次化 skill 库 / 可执行 markdown** |
| 进化对象 | agent 单向自进化 | **agent ↔ 数据 / 环境 / skill 库共进化** |
| 监督形态 | 外部 reward / human label | 出现 **reward-free intrinsic motivation** 路线 |
| RL 与 skill 关系 | 解耦：RL 训 policy，skill 库分离 | **SkillRL / SkillOS：skill 与 RL 训练在一个循环里** |
| 验证 | benchmark 黑盒打分 | **Surrogate Verifier 共同进化**（EvoSkills） |
| 单 vs 多 agent | 主要单 agent | OMC 等"组织级"自组织 multi-agent 系统出现 |
| 安全 | 偶有提及 | misevolve / alignment tipping 成单独研究方向 |
