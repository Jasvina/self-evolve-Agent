# Self-Evolving Agent — 调研资产

围绕 **Agent 自进化（Self-Evolving / Self-Improving Agents）** 这一方向的论文调研、原文备份与阅读笔记。

最近一次更新：**2026-05-14**（补齐 2026 年 2–5 月的新论文）。

## 目录结构

```
.
├── README.md
├── docs/
│   └── survey.md         # 主调研文档：分类、代表作、阅读路径、2026 趋势
└── papers/               # arXiv 原文 PDF 备份
```

## 核心问题

宣称能"自进化"的 agent，目前主要落在以下四种范式：

1. **记忆检索 / 经验回放**：把过往轨迹合成进上下文，或存进外部记忆库做检索增强（training-free）。
2. **经验挖掘 → Skill 固化**：从历史任务里抽共性、生成可复用 skill，并让 skill 集合本身随时间演化。
3. **RL / 知识蒸馏到小模型**：把大模型处理任务的轨迹蒸馏进小模型，让小模型在特定任务上对齐甚至超过教师。
4. **Self-Edit / 元认知**：agent 自己改自己的 prompt / 源码 / 上下文 playbook，或具备 metacognitive 能力。

2026 的关键演进：agent ↔ 数据 / 环境 / skill 库的**双向共进化**；**reward-free 内禀进化**；可执行的**层次化 skill 库**。详见 [docs/survey.md](docs/survey.md)。

## 论文 PDF 列表

### 综述
| 文件 | arXiv ID | 标题 |
|---|---|---|
| 2507.21046_survey_self_evolving_agents.pdf | 2507.21046 | A Survey of Self-Evolving Agents (TMLR 2026) |
| 2508.07407_comprehensive_survey_self_evolving_ai_agents.pdf | 2508.07407 | A Comprehensive Survey of Self-Evolving AI Agents |
| 2503.12434_survey_optimization_llm_agents.pdf | 2503.12434 | A Survey on the Optimization of LLM-based Agents |
| 2509.02547_landscape_agentic_rl.pdf | 2509.02547 | The Landscape of Agentic RL for LLMs |

### 经验回放
| 文件 | arXiv ID | 标题 |
|---|---|---|
| 2506.06698_contextual_experience_replay.pdf | 2506.06698 | Contextual Experience Replay |
| 2510.16079_evolver_experience_lifecycle.pdf | 2510.16079 | EvolveR: Experience-Driven Lifecycle (ICLR 2026) |

### Skill 固化（2026 集中爆发）
| 文件 | arXiv ID | 标题 |
|---|---|---|
| 2602.02474_memskill.pdf | 2602.02474 | MemSkill: Learning and Evolving Memory Skills |
| 2602.08234_skillrl_recursive_skill_rl.pdf | 2602.08234 | SkillRL: Recursive Skill-Augmented RL |
| 2605.06614_skillos_skill_curation.pdf | 2605.06614 | SkillOS: Learning Skill Curation |

> EvoSkills (arXiv:2604.01687) 多次下载超时，未存档。详情见 `docs/survey.md` §3.3。

### RL / 蒸馏到小模型
| 文件 | arXiv ID | 标题 |
|---|---|---|
| 2505.17612_distilling_llm_agent_into_small_models.pdf | 2505.17612 | Distilling LLM Agent into Small Models (NeurIPS 2025) |
| 2505.13820_structured_agent_distillation.pdf | 2505.13820 | Structured Agent Distillation |
| 2510.07841_self_improving_llm_agents_test_time.pdf | 2510.07841 | Self-Improving LLM Agents at Test-Time |
| 2604.15840_coevolve_agent_data_mutual_evolution.pdf | 2604.15840 | CoEvolve: Agent-Data Mutual Evolution |
| 2604.18131_reward_free_self_evolution_world_knowledge.pdf | 2604.18131 | Reward-Free Self-Evolution via World Knowledge |
| 2605.08083_autotts_llms_improving_llms.pdf | 2605.08083 | AutoTTS: LLMs Improving LLMs |

### 未存 PDF 但调研里提到的论文（题目供你检索）

- Self-Generated In-Context Examples (Sarukkai et al., NeurIPS 2025)
- ExpeL: LLM Agents Are Experiential Learners (Zhao et al., 2024)
- MUSE (Yang et al., 2025) — 分层记忆 + Plan-Execute-Reflect-Memorize
- ERL: Experiential Reflective Learning (ICLR 2026 MemAgents Workshop)
- SPEAR (arXiv:2509.22601) — 朴素 replay 的 entropy collapse 风险
- Your Agent May Misevolve (arXiv:2509.26354, ICLR 2026)
- Alignment Tipping Process (arXiv:2510.04860)
- SICA: Self-Improving Coding Agent (Robeyns et al., 2025)
- Intrinsic Metacognitive Learning (Liu et al., ICML 2025 position)
- OneManCompany: From Skills to Talent (arXiv:2604.22446)
- Self-Healing Framework for LLM-Based Agents (arXiv:2605.06737)
- Search, Do not Guess (arXiv:2604.04651) — 小模型蒸馏防参数化幻觉
- On-Policy Distillation (Thinking Machines blog, 2025)
- AgentArk: Multi-Agent → Single-Agent 蒸馏
- Agentic Context Engineering (ACE)
- SkillWeaver (Zheng et al., 2025)
- Voyager (2023) — skill library 开山作
- StuLife: Benchmark for ELL (2026)

## 相关资源
- [Awesome-Self-Evolving-Agents (XMUDeepLIT)](https://github.com/XMUDeepLIT/Awesome-Self-Evolving-Agents)
- [Awesome-Self-Evolving-Agents (EvoAgentX)](https://github.com/EvoAgentX/Awesome-Self-Evolving-Agents)
- ICLR 2026 Lifelong Agent (LLA) Workshop / MemAgents Workshop
