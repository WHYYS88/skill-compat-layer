# Skill 合集的接口兼容层 · 蓝本 / A Compatibility-Layer Blueprint for Skill Collections

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache--2.0-blue.svg)](LICENSE)
[![Contracts: JSON Schema](https://img.shields.io/badge/contracts-JSON%20Schema-green)](workflow/schemas/)
[![Examples: 6 validated](https://img.shields.io/badge/examples-6%20validated-brightgreen)](workflow/examples/)

> **大量 Agent Skill 放在一起时，接口往往对不上。这是一份把它们串起来、让接口对得上的蓝本。**
>
> 这个问题很普遍：**规模稍大的 skill 合集里，接口不正确的情况几乎一定会出现** —— 有的只声明了名字
> 没声明输入格式，有的干脆没有可执行入口，有的两个技能之间的数据接不上。单个技能读起来都没问题，
> **串起来跑就断**，而且断得没有声音。这不是谁做错了：技能库的价值本来就偏「知识」而非「可执行流程」。
> 缺的是中间那层接口。
>
> 下载成本已经趋近于零，使用成本没有。**钱不是门槛，时间和精力才是** —— 而等一个统一接口标准，
> 近几年看不到头。本仓库给的就是这一层：**五份数据契约 + 适配器约定 + 四级断点判定 + 九种结构化人工介入**，
> 外加**安全语义**、一份可以直接改的[流程编排定义](workflow/orchestration.yaml)，
> 以及 [6 份通过校验的真实产物实例](workflow/examples/)。
>
> 它基于 **[aipoch/medical-research-skills]**（601 个技能，MIT，Copyright (c) 2026 AIpoch）做成，
> 但**这层设计本身与具体技能库无关** —— 换一个 skill 合集，同样的思路照样成立。后续怎么改，交给使用的人。

[aipoch/medical-research-skills]: https://github.com/aipoch/medical-research-skills

[中文](#中文) · [English](#english)

---

## 中文

### 0. 快速开始

1. **拿到上游技能库**（本仓库不自带技能）：

   ```bash
   git clone https://github.com/aipoch/medical-research-skills.git
   ```

2. **读规格**：从 [workflow/overview.md](workflow/overview.md) 开始，再按需看其余几份。
3. **对着实例做**：[workflow/examples/](workflow/examples/) 里有六份真实产物，可以逐字段对照。
4. **要跑通自己的**：照 [workflow/orchestration.yaml](workflow/orchestration.yaml) 定义阶段，写自己的适配器。

> 本仓库是**规格与实例**，不含实现代码 —— 执行器由使用者按自己的技术栈实现。

### 1. 要解决的问题

技能合集有个共同特点：**单看每一个都挺像样，合起来却接不上。**
以上游 **[aipoch/medical-research-skills]**（601 个技能，MIT，Copyright (c) 2026 AIpoch）为例，
逐条读它的 SKILL.md，得到这样一组分布 —— 这组数字在同类合集里很有代表性：

| 事实 | 数量 |
| --- | --- |
| 技能总数 | 601 |
| **实测验证可执行**（`verified=true`） | **1** |
| 纯提示词技能（无可执行入口） | **201** |
| 未声明输入格式 | **401** |
| 入口未文档化 | **202** |

**这不是谁做错了。** 技能库的价值本来就偏**知识**而非**可执行流程**，
作者写技能时想的是「人读着能把事办了」，不是「机器能无人值守地串起来」。
所以缺口是结构性的：**差一层把知识变成流程的接口层。**

所以这套工作流的定位是：

> **一个知道什么时候该停下来问人的编排层，而不是一个全自动产品。**

所以我们做的不是「接好这一个库」，而是把这一层**抽象成可复用的蓝本**，做三件事：
定义**五份数据契约**让阶段之间能对上话；规定**适配器**怎么把上游产物翻译成下游输入；
规定接口对不上时如何**结构化地喊人**。

> 对大多数人来说，**兼容性接口这一层的意义最大** —— 具体连哪些技能是可以换的，
> 但「契约长什么样、阶段怎么接、断点怎么判」这套结构，换一个技能合集照样成立。

### 2. 为什么是现在做这个

**囤积的成本已经趋近于零，使用的成本没有。**

一个几百上千个技能的合集，下载下来占不了多少空间，所以**最简单的做法就是先全下来**。但下完之后大多数人会卡在同一处：**这些到底哪个能用、按什么顺序串、串不上怎么办。**

学怎么用、怎么从里面筛出自己需要的那几个 —— **这套学习的金钱成本趋近于 0**（资料全在手上，也不需要买什么）。**真正要花的是时间和精力**，而这两样不是人人都有的。

这个问题不是我们的主观感受。**Tencent 的 GraSP 论文**（[arXiv:2604.17870](https://arxiv.org/abs/2604.17870)，2026）
把根因讲得很清楚：技能的可获得性早已不是问题，**瓶颈在「技能怎么编排」** ——
而检索与执行之间缺了一层，没人回答"这些技能怎么互相依赖、正确的执行顺序是什么"。

**要编排，前提是每个环节都说得清自己的输入输出。** 这正是存量合集最缺的东西。

至于等一个统一接口标准 —— 这条路很常见，但近几年看不到头：每个平台都在推自家的技能格式，没有哪一方有强制力；就算明天出了标准，已经存在的技能也不会自动改造。

而且**标准本身也解决不了这个问题**。标准规范的是"能连上"，兼容层解决的是"数据对得上"，是两层：

| | 标准能做的 | 兼容层能做的 |
| --- | --- | --- |
| 传输格式 | ✅ 统一 | — |
| 字段叫什么 | ⚠️ 可以规定，但落不了地 | ✅ 按契约映射 |
| 上游缺字段 | ❌ 无解 | ✅ 抛结构化断点，告诉人要补什么 |
| 存量技能 | ❌ 不会自动改造 | ✅ 适配器包住，原件不动 |

所以与其等，不如先做这一层。它还有个别的好处：**跑出来的断点数据，本身就是"标准该长什么样"的实证材料。**

> 这跟技术史上几次是同一类事：npm、PyPI、Docker Hub 都曾经"下载很容易、用起来很难"，
> 而解法从来不是让人少装，是**给这一堆东西一个能转起来的结构**。

> 与 GraSP 的关系：它研究"零件齐备时如何编排"（动态编译 + 自动修复），
> 本仓库做的是它之前那一步 —— **把契约补上，让零件可编排**。两者不冲突，可以叠加。
> 详见 [workflow/prior-art.md](workflow/prior-art.md)。

需要说清楚的边界：**本仓库给出的这层结构，目前只打通了一条路**
（模拟数据 → 清洗 → 分析 → 出图 → 报告，4 种图型、1 个原生技能），
不是 601 个技能都能跑。详见 [§7 能力与限制](#7-能力与限制诚实版) 与 [capability.md](workflow/capability.md)。
**但这一条路是真的通的，产物在 [workflow/examples/](workflow/examples/)。**

### 3. 五阶段流水线

```mermaid
graph LR
    S1[SimulatedDataset] -->|S1 生成| S2[CleanDataset]
    S2 -->|S2 处理| S3[AnalysisResult]
    S3 -->|S3 分析| S4[FigureArtifact]
    S4 -->|S4 出图| S5[ReportBundle]
    S5 -.->|幂等 + 回滚| S1
    style S1 fill:#4472C4,stroke:#333,color:#fff
    style S2 fill:#5B9BD5,stroke:#333,color:#fff
    style S3 fill:#70AD47,stroke:#333,color:#fff
    style S4 fill:#FFC000,stroke:#333,color:#000
    style S5 fill:#ED7D31,stroke:#333,color:#fff
```

```
S1 生成          S2 处理          S3 分析           S4 出图            S5 报告
SimulatedDataset → CleanDataset → AnalysisResult → FigureArtifact → ReportBundle
      │                │               │                │                │
      └── 每个交接点：契约校验 + 四级判定 + 失败回滚 + 必要时结构化人工介入 ──┘
```

五份契约共用一个 **13 字段统一信封**，全部带 `provenance` 血缘，可沿 `inputs` 回溯到最初数据。

### 4. 用这套规格能做什么

规格本身不是终点，它是让你少走弯路的起点。照着它，你可以搭出：

| 你想要的 | 规格里已经给了什么 |
| --- | --- |
| **一条跑得通的流水线** | 五阶段顺序、每阶段输入输出、失败回滚要清什么、**以及怎么验证上游没被弄坏** |
| **能互相对上话的阶段** | 五份契约的字段级定义 + 9 个 JSON Schema（可直接拿去做校验） |
| **出了问题能找到根因** | 统一血缘：从任一产物沿 \`inputs\` 回溯到最初的模拟数据 |
| **不靠人盯的重复执行** | 幂等语义：同输入产出逐字节相同；内容没变就不重写文件（含实现要点与陷阱） |
| **会主动喊人的系统** | 四级判定尺度 + 九种原因码 + 断点四要素（卡在哪 / 要什么格式 / 怎么补 / 证据） |
| **坏数据不外泄的兜底** | 安全接口：三态策略、故障分类、四类触发点，命中即刻切断 |
| **接自己的技能与工具** | 适配器实现契约：统一签名、必需行为、技能门禁，以及调外部脚本的五个坑 |
| **对外说清楚边界** | 能力与限制声明怎么写（含"601 个里只有 1 个能跑"这种难看的数字） |

具体到落地场景：

- **给技能库补一层执行层** —— 你有一堆 SKILL.md 或工具脚本，想让它们按顺序跑起来，
  而不是每次靠人拼命令。规格里的适配器契约与门禁机制就是为这个准备的。
- **审查别人交来的分析结果** —— 拿 Schema 校验，用四级判定标出哪些字段其实不可用，
  让问题在进入结论之前就暴露。
- **把断点做成工单** —— 退出码 2 + 结构化 JSON，上游系统可以直接捕获、派单、回收，
  不用人去日志里捞。
- **评估一套研究流程是否可靠** —— 血缘链 + 幂等 + 熔断语义，让"可复现"有可验证的依据。

想要一份可以直接对照的样板，看 **[workflow/examples/](workflow/examples/)**：
五份契约实例加一个真实触发的人工介入请求，全部通过仓库自带的 Schema 校验。

> 集成这类工作的难点**从来不是写转换函数**，而是把"阶段之间怎么对话、出了问题怎么办"
> 这层语义定清楚。这套规格就是把那一层写出来了。

### 5. 规格文档

| 文档 | 内容 |
| --- | --- |
| [workflow/orchestration.yaml](workflow/orchestration.yaml) | **流程编排定义**：阶段顺序、输入输出声明、门禁、失败回滚、断点触发点（声明式，拿去就能改） |\n| [workflow/overview.md](workflow/overview.md) | 流水线、统一信封、血缘、幂等语义、回滚与验证 |
| [workflow/contracts.md](workflow/contracts.md) | 五份契约的载荷字段与硬约束 |
| [workflow/criteria.md](workflow/criteria.md) | 四级断点判定尺度与逐列留痕 |
| [workflow/interventions.md](workflow/interventions.md) | 九种人工介入原因码、四要素、退出码约定 |
| [workflow/safety.md](workflow/safety.md) | 安全接口（熔断器）：什么故障必须即刻切断 |
| [workflow/adapters.md](workflow/adapters.md) | 适配器实现契约与执行外部技能的工程注意点 |
| [workflow/capability.md](workflow/capability.md) | **能力与限制声明**（含做不到什么） |
| [workflow/schemas/](workflow/schemas/) | 9 个 JSON Schema（机器可读的契约定义） |
| [workflow/prior-art.md](workflow/prior-art.md) | 相关工作与先技术：GraSP 论文要点、与它的区别、MCP 故障分类学等 |
| **[workflow/examples/](workflow/examples/)** | **五份契约实例 + 一个断点请求实例**（真实跑出来的产物，可直接对照） |
| [CONTRIBUTING.md](CONTRIBUTING.md) | 什么样的贡献有价值、自查清单、提交信息规范 |
| [SECURITY.md](SECURITY.md) | 什么算安全问题（含"规格的安全承诺是否兑现"）与上报方式 |
| [REPO_DESCRIPTION.md](REPO_DESCRIPTION.md) | 仓库描述与推广文案（引用时用这份，别自己拼） |

### 6. 核心设计要点

**硬约束写进模型，不是写在文档里**

信封、血缘、断点这些结构是通用的；**但每条硬约束的具体内容，取决于你的技能库面向什么场景** ——这部分由每个蓝本自己定，不是通则。

本仓库的做法是**把它塞进模型校验**，而不是写在文档里靠自觉。三类约束作为示例：

| 约束 | 本仓库为什么这样定 |
| --- | --- |
| 数据必须标记为合成、且不含真实患者数据 | 场景是医疗研究，合规风险高，所以让模型层直接拒绝 |
| 源里没有的数值一律留 `TODO`，绝不编造 | 学术场景下编造数值比缺值严重得多 |
| 路径字段必须是工作区相对路径，禁绝对路径与 `..` | 产物要跨机器复现，绝对路径会失效 |

> 换个领域，这三条大概率要换掉（比如做代码生成的技能库，"禁止编造"的含义就完全不同）。
> **可复用的是"把约束写进校验"这个做法，不是这三条约束本身。**

**断点是一等公民**

接口对不上时抛结构化请求，含四要素：**卡在哪** / **需要什么格式** / **建议怎么做** / **证据**。
只打日志不算断点。九种原因码见 [interventions.md](workflow/interventions.md)。

**安全接口**

回答一个很多编排器没回答的问题：**哪些故障属于"再跑下去就是错的"？**
三态策略 `off` / `default` / `strict`，命中即刻切断。
详见 [safety.md](workflow/safety.md)。

**幂等**

同输入产出逐字节相同，内容未变不重写文件。
本蓝本早期版本在落盘内容里写了一处"本次执行的时刻"，结果每次都判为内容有变、每次都重写 —— **幂等声明形同虚设**。这是实测发现的一个反例。
详见 [overview.md §5](workflow/overview.md)。

### 7. 能力与限制（诚实版）

| 能做什么 | 做不到什么 |
| --- | --- |
| 五阶段端到端，含失败回滚与幂等 | **601 个技能里只有 1 个实测可执行** |
| 四级判定 + 九种原因码的结构化人工介入 | 201 个纯提示词技能**永远无法**接入自动流水线 |
| 安全接口：致命故障即刻切断 | 图型覆盖有限 |
| 出图两条通道：本地兜底 / 上游原生技能 | 平台实测仅 Windows x64 + CPython 3.10 |

完整声明见 [workflow/capability.md](workflow/capability.md)。

### 8. 本仓库包含 / 不包含

**包含**：工作流规格文档 + 契约 JSON Schema。

**不包含**（刻意的）：

| 内容 | 原因 |
| --- | --- |
| 实现代码（契约模型、适配器、编排器、安全接口、出图网关等） | 这一层留给使用者按自己的技术栈实现 |
| 601 个上游 SKILL.md 快照 | 第三方作品，按上游许可需自行获取 |
| 技能审计注册表 | 属于实现侧的运行数据 |
| R 包与运行时依赖 | 应由使用者按自身环境安装 |

### 9. 环境变量

实现本规格时**全部可选**，都有合理默认值。模板见 [.env.example](.env.example)：

| 变量 | 作用 |
| --- | --- |
| `AITOCI_ROOT` | 项目根（默认从文件位置自推导） |
| `AITOCI_OUTPUTS` | 产物目录 |
| `AITOCI_SKILL_SPEC` | 技能注册表路径 |
| `AITOCI_WORKFLOW` | 流水线定义路径 |
| `AITOCI_BREAKER_POLICY` | 安全接口策略（`off` / `default` / `strict`） |
| `AITOCI_DETERMINISTIC` | `=1` 时字节级可复现 |
| `R_LIBS_USER` | R 包库位置（仅原生通道需要） |

### 10. 上游与致谢

本项目**不自带技能**，它是一层给上游技能库用的集成规格。没有上游，这套规格没有意义。

| 项 | 内容 |
| --- | --- |
| **原仓库（真正的上游）** | **[aipoch/medical-research-skills]** |
| 原仓库地址 | https://github.com/aipoch/medical-research-skills |
| 原仓库许可 | **MIT License**（LICENSE 文件在上游根目录） |
| 版权 | Copyright (c) 2026 AIpoch |
| 本项目与它的关系 | 派生自对上游的事实性审计与统计；**不复制**其正文，**不分发**其技能快照或脚本 |

> 本项目的审计基于该仓库 **commit \`686e09dfdb\`**（2026-09-17）快照。

**具体派生在哪：**

- [workflow/capability.md](workflow/capability.md) 里的能力分布数字（601 / 201 / 401 / 202 等）
  来自对上游逐条 SKILL.md 的统计；
- 本规格中对"技能入口常未文档化""大量技能未声明输入格式"等判断，都是读上游原文得出的；
- 规格本身（契约设计、判定尺度、断点语义、安全接口）由本项目独立设计，不派生自上游。

**想用这套规格，你需要先拿到上游：**

\`\`\`bash
git clone https://github.com/aipoch/medical-research-skills.git
\`\`\`

上游的 MIT 许可要求保留其版权与许可声明 —— 这一点已在 [NOTICE.md](NOTICE.md) 中体现，
本仓库不复述上游正文，只做事实性引用。

### 11. 参与

这个仓库是规格类项目，欢迎的贡献主要是三类：**把规格写清楚、把实例补全、把踩过的坑记下来**。

- 想提改动 → [CONTRIBUTING.md](CONTRIBUTING.md)（含自查清单）
- 发现错误 → 用仓库自带的 [Issue 模板](.github/ISSUE_TEMPLATE/)
- 安全问题 → **别开公开 Issue**，走 [SECURITY.md](SECURITY.md)

> 一般性的建议：**本蓝本刻意保持克制，少一条约束往往比多一条好。**
> 如果你的场景需要额外约束，那更适合作为"你那个蓝本"的一部分，而不是塞进这里。

### 12. 许可

**Apache License 2.0**，见 [LICENSE](LICENSE)。
技能注册表相关内容的派生来源与第三方归属见 [NOTICE.md](NOTICE.md)。

---

## English

### Quick start

1. **Get the upstream skills** (this repository ships none):

   ```bash
   git clone https://github.com/aipoch/medical-research-skills.git
   ```

2. **Read the spec** — start with [workflow/overview.md](workflow/overview.md).
3. **Work from the examples** — [workflow/examples/](workflow/examples/) holds six real outputs to compare field by field.
4. **Build your own** — define stages per [workflow/orchestration.yaml](workflow/orchestration.yaml) and write your own adapters.

> This repository is a **specification plus examples**, not an implementation. The executor is up to you.

### What this repository is

A **blueprint for the compatibility layer** that any large collection of agent skills needs.
Put a few hundred agent skills side by side and their interfaces will not line up: some declare a name
but no input format, some have no executable entry at all, some simply cannot hand data to the next one.
Each skill reads fine on its own; **chained together, it breaks — silently.**

Acquisition cost has collapsed; utilization cost has not. **Money is not the barrier — time and
attention are**, and waiting for a unified interface standard is not a plan for the next few years.
This repository is that missing layer — with [six validated real outputs](workflow/examples/).
This repository is that missing layer: five data contracts, adapter conventions, a 4-level breakpoint
criterion, nine structured intervention reasons, safety semantics, and a
[declarative orchestration file](workflow/orchestration.yaml) you can edit directly.
For most people **this compatibility layer is the valuable part** — which skills you connect is
replaceable; how stages talk to each other is not.

Built on the upstream skill library **[aipoch/medical-research-skills]** (601 skills, MIT licensed,
Copyright (c) 2026 AIpoch) — but the design itself is library-agnostic: swap in another collection of
skills and the same structure still holds. See [NOTICE.md](NOTICE.md) for attribution.

[aipoch/medical-research-skills]: https://github.com/aipoch/medical-research-skills

Auditing the upstream skills:

| Fact | Count |
| --- | --- |
| Total skills | 601 |
| **Verified executable** | **1** |
| Prompt-only (no executable entry) | **201** |
| Input format undeclared | **401** |
| Entry point undocumented | **202** |

The upstream library's value is **knowledge**, not **executable workflow**. Treating it as automation
hits a wall at stage one — silently.

> **This is not a fully-automated product. It is an orchestration layer that knows when to stop and ask.**

### Why build this now

**Acquisition cost has collapsed; utilization cost has not.**

A collection of several hundred or thousand skills costs almost nothing to download, so downloading all
of it is **the simplest thing to do**. But most people then stall at the same place: **which of these
actually runs, in what order do they chain, and what happens when they don't connect.**

Learning how to use them, and how to pick the few you actually need, **costs almost no money** — the
material is already in your hands. **What it costs is time and attention**, and not everyone has those
to spare.

This is not just our impression. **GraSP** ([arXiv:2604.17870](https://arxiv.org/abs/2604.17870),
Tencent, 2026) names the root cause: skill availability is no longer the bottleneck — **orchestration
is** — and between retrieval and execution there is a missing layer that should answer *how these
skills depend on each other and what the correct execution order is*.

**Orchestration presupposes that every step can state its own inputs and outputs** — precisely what
existing collections lack most.

The common answer is to wait for a unified interface standard. That is not coming soon: every platform
is pushing its own skill format and none has the authority to impose one; and even a standard published
tomorrow would not retroactively convert the skills that already exist.

**A standard would not solve this anyway.** A standard governs *can they connect*; a compatibility layer
governs *does the data match*. Two different layers:

| | What a standard gives | What a compatibility layer gives |
| --- | --- | --- |
| Wire format | ✅ unified | — |
| Field names | ⚠️ can specify, hard to enforce | ✅ mapped by contract |
| Upstream missing a field | ❌ no answer | ✅ structured breakpoint telling a human what to supply |
| Existing skills | ❌ not converted retroactively | ✅ wrapped by adapters, originals untouched |

So rather than wait, build this layer. It has a side benefit: **the breakpoint data it produces is itself
evidence for what the eventual standard should look like.**

> This mirrors what happened with npm, PyPI and Docker Hub — all of them were "easy to download, hard to
> use". The fix was never "download less"; it was **giving the pile a structure that can actually turn.**

One boundary to state plainly: **the structure here currently opens up one path**
(simulated data → clean → analyse → figure → report; 4 figure types, 1 native skill), not all 601 skills.
See [§Status](#status) and [capability.md](workflow/capability.md). **But that one path genuinely runs,
and its real output is in [workflow/examples/](workflow/examples/).**

### Pipeline

```
S1 Generate → S2 Process → S3 Analyze → S4 Figure → S5 Report
SimulatedDataset → CleanDataset → AnalysisResult → FigureArtifact → ReportBundle
   every handoff: schema check + 4-level criterion + rollback + structured human intervention
```

All five contracts share a **13-field envelope** and carry `provenance` lineage traceable back to the
original data via `inputs`.

### What you can build with this

A specification is a starting point, not a deliverable. Following it, you can build:

| What you want | What the spec already gives you |
| --- | --- |
| **A pipeline that runs** | Stage order, per-stage I/O, what rollback must clean, and how to verify upstream is intact |
| **Stages that understand each other** | Field-level definitions of five contracts plus 9 JSON Schemas for validation |
| **Root-cause traceability** | Unified lineage: any artifact traces back to the original data via \`inputs\` |
| **Repeatable runs without babysitting** | Idempotency semantics: byte-identical output, no rewrite when unchanged |
| **A system that asks for help** | 4-level criterion, 9 reason codes, and the four required elements of a breakpoint |
| **A guarantee that bad data stops** | Safety interface: three policies, fault classes, four trigger points |
| **Your own skills and tools** | Adapter contract: uniform signature, required behaviours, skill gating |
| **An honest capability boundary** | How to write a capability statement — including unflattering numbers |

Concretely:

- **Add an execution layer to a skill library** — make a pile of SKILL.md files or tool scripts run in
  order instead of being hand-assembled every time.
- **Review analysis results someone handed you** — validate against the Schemas and flag fields that
  are actually unusable before they reach a conclusion.
- **Turn breakpoints into tickets** — exit code 2 plus structured JSON lets an upstream system capture,
  dispatch and close them without digging through logs.
- **Assess whether a research pipeline is trustworthy** — lineage, idempotency and halting semantics
  make "reproducible" something you can verify.

> The hard part of integration is never writing the transformation functions — it is pinning down how
> stages talk to each other and what happens when they can't. That layer is what this spec writes down.

### Specification documents

| Document | Contents |
| --- | --- |
| [workflow/overview.md](workflow/overview.md) | Pipeline, envelope, lineage, idempotency, rollback |
| [workflow/contracts.md](workflow/contracts.md) | Payload fields and hard constraints for all five contracts |
| [workflow/criteria.md](workflow/criteria.md) | 4-level breakpoint criterion and per-column audit trail |
| [workflow/interventions.md](workflow/interventions.md) | 9 reason codes, four required elements, exit codes |
| [workflow/safety.md](workflow/safety.md) | Safety interface: which faults must halt immediately |
| [workflow/adapters.md](workflow/adapters.md) | Adapter contract and engineering pitfalls |
| [workflow/capability.md](workflow/capability.md) | **Capability and limitation statement** |
| [workflow/schemas/](workflow/schemas/) | 9 JSON Schemas (machine-readable contract definitions) |
| [workflow/prior-art.md](workflow/prior-art.md) | Related work: what GraSP does, how this differs, MCP fault taxonomy |
| **[workflow/examples/](workflow/examples/)** | **Five contract instances plus a real intervention request** — actual pipeline output to compare against |
| [CONTRIBUTING.md](CONTRIBUTING.md) | What contributions are wanted, self-check list, commit conventions |
| [SECURITY.md](SECURITY.md) | What counts as a security issue here, and how to report it |

### Key design points

- **Hard constraints live in the model validators, not the docs** — the envelope, lineage and
  breakpoints are general, but **which constraints apply depends on what your skill collection is
  for**; each blueprint decides its own. This one, for example: data must be marked synthetic and
  carry no real patient data; numbers absent from the source stay `TODO` and are never fabricated;
  path fields must stay workspace-relative. **What transfers is the practice of enforcing
  constraints in validation — not these three constraints.**
- **Breakpoints are first-class**: every stop emits a structured request with blocker, required
  format, suggested actions and evidence. Logging alone is not a breakpoint.
- **Safety interface**: answers *which faults mean "continuing is simply wrong"* — three policies
  (`off` / `default` / `strict`) halt immediately on fatal faults.
- **Idempotency**: identical input yields byte-identical output, and unchanged content is never
  rewritten. The persisted content must not embed "the time this run happened".

### Status

Works: 5-stage pipeline with rollback and idempotency; 4-level criterion; 9 structured intervention
reason codes; safety interface; two rendering channels.
Does not work: only **1 of 601** skills is verified executable; 201 are prompt-only and can never be
automated; figure-type coverage is limited; only Windows x64 + CPython 3.10 has been tested.

### Upstream and acknowledgements

This project ships **no skills of its own** — it is an integration specification for the upstream
skill library. Without the upstream, this specification is meaningless.

| Item | Value |
| --- | --- |
| Upstream repository | **[aipoch/medical-research-skills]** |
| URL | https://github.com/aipoch/medical-research-skills |
| Licence | **MIT License** |
| Copyright | Copyright (c) 2026 AIpoch |
| Relationship | Derived from a factual audit of the upstream; **does not copy** its content and **does not distribute** its skill snapshots or scripts |

Get the upstream first:

\`\`\`bash
git clone https://github.com/aipoch/medical-research-skills.git
\`\`\`

The upstream MIT licence requires preserving its copyright and permission notice, which
[NOTICE.md](NOTICE.md) does. This repository quotes no upstream prose — only factual statistics.

### Licence

**Apache License 2.0** — see [LICENSE](LICENSE) and [NOTICE.md](NOTICE.md).
