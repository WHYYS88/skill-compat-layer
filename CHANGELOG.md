# Changelog

本项目遵循 [语义化版本](https://semver.org/lang/zh-CN/)。

## [0.1.0] - 2026-09-25

首个公开版本：**Skill 合集的接口兼容层蓝本**。
定义了契约、阶段衔接、断点判定与安全语义，并给出一份可直接修改的流程编排定义。

### 新增

- **流程编排定义**（[workflow/orchestration.yaml](workflow/orchestration.yaml)）：声明式的阶段顺序、每阶段输入输出契约、技能门禁、失败回滚规则与验证方式、五个断点触发点。不含算法，拿去就能改。
- **社区文件**：CONTRIBUTING.md（贡献范围与自查清单）、SECURITY.md（什么算安全问题）、REPO_DESCRIPTION.md（描述与推广文案）、.github/ 下的 Issue 与 PR 模板
- **README 结构**：加了徽章、快速开始、流水线示意图（mermaid），中英双语同步
- **为什么是现在做这个**：下载成本趋近于零而使用成本没有；为什么等统一接口标准不是方案；标准规范的是「能连上」、兼容层解决的是「数据对得上」，是两层
- **工作流总览**：五阶段流水线、13 字段统一信封、血缘链、幂等语义、回滚与验证要求
  （[workflow/overview.md](workflow/overview.md)）
- **五份数据契约规格**：`SimulatedDataset` / `CleanDataset` / `AnalysisResult` /
  `FigureArtifact` / `ReportBundle` 的载荷字段与硬约束
  （[workflow/contracts.md](workflow/contracts.md)）
- **9 个 JSON Schema**：机器可读的契约定义，可直接用于实现侧校验
  （[workflow/schemas/](workflow/schemas/)）
- **四级断点判定尺度**：`PASS` / `WARN_NO_TYPE` / `WARN_NO_SEMANTICS` / `BLOCK`，
  含逐列留痕要求（[workflow/criteria.md](workflow/criteria.md)）
- **九种人工介入原因码**与请求四要素、退出码约定
  （[workflow/interventions.md](workflow/interventions.md)）
- **安全接口规格**：三态策略、故障分类、四类触发源、实现陷阱
  （[workflow/safety.md](workflow/safety.md)）
- **适配器实现契约**：统一签名、必需行为、技能门禁、执行外部技能的工程注意点
  （[workflow/adapters.md](workflow/adapters.md)）
- **能力与限制声明**：如实列出做不到什么
  （[workflow/capability.md](workflow/capability.md)）

### 说明

- 本版本**不分发**上游技能快照与技能审计注册表。

### 已知限制

- 本蓝本在一个 601 技能的合集上验证：实测验证可执行的仅 1 个；201 个为纯提示词，无法自动执行。
  这组数字在同类 skill 合集里很有代表性 —— **接口缺口是结构性的，不是个别仓库的问题**。
- 图型覆盖有限
- 平台实测范围：Windows x64 + CPython 3.10

### 本版本沉淀的工程经验（已写进规格）

以下都是实现过程中真实踩过的坑，已转化为规格中的显式要求：

- 落盘内容含"本次执行时刻"会让幂等声明失效；且某些文件系统的时间戳粒度
  （exFAT 为 2 秒）会把"其实每次都在重写"伪装成"没有重写"
- 执行外部脚本时，脚本路径不存在会导致解释器直接崩溃且 stderr 为空
- "能力具备" ≠ "这份输入喂得进去"：自动择优如果只判断前者，选出的通道会直接撞墙
- 安全接口最容易误伤正常流程：先用正常路径跑一遍确认不误报，再验证它该报的时候报不报

[aipoch/medical-research-skills]: https://github.com/aipoch/medical-research-skills
