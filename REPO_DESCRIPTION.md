# 仓库描述文案

供 GitHub 仓库描述框、推广材料、以及别处引用时使用。

## GitHub 描述框（精简）

**英文（推荐，64 字符）**：

```
A contract-verified skill orchestration blueprint with structured breakpoints
```

**中文（36 字）**：

```
面向 Agent 技能合集的契约校验编排蓝本，含结构化断点与安全语义
```

> GitHub 描述框建议 ≤ 120 字符、首屏可见约 70 字符，上面两条都在安全范围内。

## 详细版（用于 README 开头或推广文案）

skill-compat-layer 是一份面向 Agent Skill 合集的**接口兼容层蓝本**。

它提供五份数据契约、适配器约定、四级断点判定、九种结构化人工介入机制，以及安全熔断语义 ——
让原本各自独立的技能能够按流水线串联执行，并在接口对不上时**结构化地"喊人"介入**，而不是静默出错。

基于 [aipoch/medical-research-skills](https://github.com/aipoch/medical-research-skills)（601 个技能）验证，
但**设计本身与具体技能库无关**，可复用于任意技能合集。

## 一句话

中文：

> 技能下载成本趋近于零，使用成本却没有 —— 我们给这堆技能加一层契约校验与编排接口，
> 让它们串起来跑的时候不再**无声地断**。

English:

> Downloading skills is nearly free; using them is not. This is a contract-and-orchestration layer
> that keeps a pile of heterogeneous agent skills from breaking **silently** when you chain them.

## 关键词（用于 Topics / SEO）

```
agent-skills, skill-orchestration, compatibility-layer, data-contracts,
workflow-specification, human-in-the-loop, circuit-breaker, blueprint
```
