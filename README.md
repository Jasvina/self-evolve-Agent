# Self-Evolving Agent — 调研资产

围绕 **Agent 自进化（Self-Evolving / Self-Improving Agents）** 这一方向的论文调研、原文备份与阅读笔记。

## 目录结构

```
.
├── README.md
├── docs/
│   └── survey.md         # 主调研文档：分类、代表作、阅读路径
└── papers/               # arXiv 原文 PDF 备份
```

## 核心问题

宣称能"自进化"的 agent，目前主要落在以下几种范式：

1. **记忆检索 / 经验回放**：把过往轨迹合成进上下文，或存进外部记忆库做检索增强（training-free）。
2. **经验挖掘 → Skill 固化**：从历史任务里抽共性、生成可复用 skill，并让 skill 集合本身随时间演化。
3. **RL / 知识蒸馏到小模型**：把大模型处理任务的轨迹蒸馏进小模型，让小模型在特定任务上对齐甚至超过教师。
4. **Self-Edit / 元认知**（容易被忽略的第四类）：agent 自己改自己的 prompt / 源码 / 上下文 playbook，或具备 metacognitive 能力。

详细分类与代表论文见 [docs/survey.md](docs/survey.md)。

## 论文 PDF 列表

| 文件 | arXiv ID | 标题 |
|---|---|---|
| 2507.21046_survey_self_evolving_agents.pdf | 2507.21046 | A Survey of Self-Evolving Agents: What, When, How, and Where to Evolve |
| 2508.07407_comprehensive_survey_self_evolving_ai_agents.pdf | 2508.07407 | A Comprehensive Survey of Self-Evolving AI Agents |
| 2503.12434_survey_optimization_llm_agents.pdf | 2503.12434 | A Survey on the Optimization of LLM-based Agents |
| 2509.02547_landscape_agentic_rl.pdf | 2509.02547 | The Landscape of Agentic Reinforcement Learning for LLMs |
| 2506.06698_contextual_experience_replay.pdf | 2506.06698 | Contextual Experience Replay for Self-Improvement of Language Agents |
| 2505.17612_distilling_llm_agent_into_small_models.pdf | 2505.17612 | Distilling LLM Agent into Small Models with Retrieval and Code Tools |
| 2505.13820_structured_agent_distillation.pdf | 2505.13820 | Structured Agent Distillation for LLM Agents |
| 2510.07841_self_improving_llm_agents_test_time.pdf | 2510.07841 | Self-Improving LLM Agents at Test-Time |
