# 人工介入规格 / Human Intervention

断点是本工作流的**一等公民**：接口对不上时，系统主动、结构化地喊人，
而不是打条日志继续跑。

---

## 1. 请求结构

| 字段 | 必填 | 含义 |
| --- | --- | --- |
| `request_id` | ✅ | 确定性派生的请求 id |
| `contract_id` | | 涉及的契约 id；未知为 `TODO` |
| `stage` | | 发生的阶段 |
| `adapter` | ✅ | 发起请求的模块名 |
| `source_skill` | | 相关技能 id |
| `blocker` | ✅ | 卡点现象（人类可读） |
| `blocker_code` | ✅ | 原因码，见下（**不得用自由文本代替**） |
| `required_format` | ✅ | 需要人类提供的具体数据格式，或需要确认的字段 |
| `required_fields` | | 具体缺哪些字段 |
| `suggested_actions` | | 建议操作（**不得为空**） |
| `evidence` | | 可复现证据 |
| `severity` | | `blocking` / `degraded` |
| `created_at` | | 时刻 |

## 2. 九种原因码

| 原因码 | 触发场景 | 典型的人类动作 |
| --- | --- | --- |
| `prompt_only_skill` | 目标是纯提示词技能，无可执行入口 | 人工运行该技能并提供结果文件 |
| `entry_not_documented` | 技能入口未文档化 | 补全注册表 entry，或人工提供结果 |
| `format_undeclared` | 技能未声明输入格式 | 声明格式，或人工转换 |
| `dependency_undeclared` | 声明了依赖但未声明版本 | 补版本，或人工确认环境 |
| `payload_schema_mismatch` | 契约字段对不上（如要画森林图却没有置信区间） | 提供含所需字段的结果表 |
| `unsupported_transformation` | 该转换无法进行（如上游是纯文本没有结构化表） | 提供结构化数据 |
| `ambiguous_semantics` | 列无语义也无类型 | 提供列映射 |
| `data_quality_blocker` | 数据本身不可用 | 修数据后重跑 |
| `renderer_unavailable` | 出图通道不可用（缺运行时/依赖/脚本） | 装依赖，或改用兜底通道 |

> 本蓝本让实现层只使用这组枚举，不用自由文本代替原因码 —— 否则原因无法统计、无法路由。

## 3. 严重级别与会话语义

| 级别 | 含义 | 默认动作 |
| --- | --- | --- |
| `blocking` | 本阶段无法继续 | 中断本阶段 + 回滚 + 非零退出码 |
| `degraded` | 结果可用但有折损 | 记录并继续 |

阶段的默认动作在编排声明里给出（`on_human_intervention`）。
**但"遇到 blocking 断点是否继续"与"是否允许把降级结果当正常结果往下传"是两件事**，
后者由[安全接口](safety.md)负责。

## 4. 退出码约定

| 码 | 含义 |
| --- | --- |
| `0` | 成功 |
| `2` | 需要人工介入（结构化请求已写入 stdout / `--intervention-out`） |
| `3` | 输入或契约错误 |
| `1` | 其他异常 |

约定：**退出码 2 时，stdout 或 `--intervention-out` 必须包含合法 JSON 请求**，
便于上游系统自动捕获并派单给人工。
