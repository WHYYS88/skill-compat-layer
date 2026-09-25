# 断点判定尺度 / Breakpoint Criteria

**逐列判定"这个字段到底能不能用"**。

之所以要把它写成显式的四级，是因为技能合集里字段的声明状况参差不齐 —— 靠感觉判断，同一个人隔天都会给出不同结论，更不用说换个人接手。

---

## 四级判定

| 级别 | 条件 | 处理 |
| --- | --- | --- |
| `PASS` | **有语义 + 有类型** | 直接放行 |
| `WARN_NO_TYPE` | 有语义（列名可映射到规范名）、无类型声明 | 记警告并**放行** |
| `WARN_NO_SEMANTICS` | 有类型、无语义（列名无法映射） | 记警告并**放行** |
| `BLOCK` | **既无语义也无类型** | **抛结构化人工介入** |

### 为什么这么分

上游技能库 **[aipoch/medical-research-skills](https://github.com/aipoch/medical-research-skills)**（MIT, Copyright (c) 2026 AIpoch）里，大量字段只声明一半（有名字没说类型，或反之）——这套尺度就是为应对这种形态设计的。
如果"缺一半就拦"，几乎所有数据都会被拦下，流水线无法推进；
如果"缺一半也放行且不记录"，缺失会一路静默传到结论里。

所以尺度是：**缺一半 → 放行但留痕；全缺 → 拦下问人。**
留痕不是形式：`column_status` 会写进契约的 `metadata` 与血缘 `params`，事后可审计。

---

## 逐列状态记录

判定结果必须逐列落盘：

| 记录项 | 位置 | 内容 |
| --- | --- | --- |
| 每列判定 | `metadata.column_status` | `{列名: pass/warn_no_type/warn_no_semantics/block}` |
| 告警明细 | `metadata.column_warnings` | 具体哪列、缺什么 |
| 阻断列 | `metadata.blocking_columns` | 触发 BLOCK 的列 |
| 未解析列 | `metadata.unresolved_columns` | 未能映射到规范名的列 |
| 血缘留痕 | `provenance[-1].params.column_status` | 便于沿血缘回溯 |

---

## 人工介入的四要素（硬要求）

任何断点**必须**给出以下四样，缺一不可。只打日志不算断点：

| 要素 | 字段 | 回答什么问题 |
| --- | --- | --- |
| 卡在哪 | `blocker` + `blocker_code` | 现象是什么，属于哪一类 |
| 需要什么格式 | `required_format` + `required_fields` | 人必须补什么才能继续 |
| 建议怎么做 | `suggested_actions` | 至少一条可执行步骤 |
| 证据 | `evidence` | 可复现的文件路径 / 命令 / 校验摘要 |

`severity` 取 `blocking`（中断本阶段）或 `degraded`（记录并继续）。

> `suggested_actions` **不得为空**；为空时应由实现层拒绝构造该请求。
