医疗研究 Agent Skill 集成层 · 规格
Copyright 2026 The Integration Layer Authors

本仓库的内容是工作流规格与契约定义。
以下为其中涉及的第三方归属与派生内容说明。

================================================================================
1. 派生内容（上游技能库）
================================================================================

本仓库的规格文档与 JSON Schema 中包含对下列作品的事实性描述与统计：

    medical-research-skills
    Copyright (c) 2026 AIpoch, AIPOCH
    原仓库：https://github.com/aipoch/medical-research-skills
    来源：https://github.com/aipoch/medical-research-skills
    许可：MIT License

派生方式为**事实性抽取与统计**：读取各技能的 SKILL.md，统计其入口文档化情况、
输入格式声明情况与依赖声明情况（见 workflow/capability.md 的能力声明表）。
本仓库**不复制**该作品的正文内容，也**不分发**其 SKILL.md 快照或可执行脚本；
使用者需自行从其上游获取。

本仓库**不包含**具体技能的审计注册表（技能级元数据），该内容属于实现侧的运行数据。

================================================================================
2. 第三方署名内容的处置
================================================================================

审计过程中识别出 3 个技能带有非上游作者的第三方署名。这些技能已从本项目使用的
技能集合中移除。本仓库不包含它们，也不包含其任何内容。

================================================================================
3. 契约定义的来源
================================================================================

workflow/schemas/ 下的 JSON Schema 与 workflow/contracts.md 由本项目独立设计，
描述本项目自定义的五个数据契约与信封结构，不派生自任何第三方作品。

================================================================================
4. MIT License（适用于第 1 节所述派生内容）
================================================================================

MIT License

Copyright (c) 2026 AIpoch

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
