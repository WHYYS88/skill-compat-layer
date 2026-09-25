# 数据契约规格 / Data Contracts

五份契约，共用一个 13 字段信封（见 [overview.md](overview.md#3-统一信封13-字段)）。
本文件只规定**载荷字段**。机器可读定义在 [schemas/](schemas/)。

标记：`*` = 必填；`TODO` = 未声明时的占位值，**不得编造**。

---

## 1. SimulatedDataset（S1 生成）

> 本蓝本的合规硬约束：这一层**只接受模拟数据**（医疗场景，合规风险高，所以让模型层直接拒绝）。

| 字段 | 类型 | 约束 |
| --- | --- | --- |
| `design` | `DatasetDesign` | 设计声明 |
| `tables` | `{name: TablePayload}` | 数据表 |
| `documents` | `[DocPayload]` | 文本/文档载荷 |
| `answer_key` | object | 模拟真值（若有） |
| `is_synthetic` | boolean | **必须为 `true`** —— 模型层拒绝非模拟数据 |
| `contains_real_patient_data` | boolean | **必须为 `false`** |

### DatasetDesign

| 字段 | 含义 |
| --- | --- |
| `n_samples` / `n_features` | 样本/特征数 |
| `groups` | 分组与每组样本数 |
| `seed` | **随机种子：可复现的前提**，缺失时校验告警 |
| `missing_rate` | 缺失率 |
| `notes` | 备注 |

**逐列可审计**要求：每列都要记录生成方式（常量/分布/分类/派生）与参数，
写入 `metadata`。**未声明生成方式的列不得生成。**

---

## 2. CleanDataset（S2 处理）

| 字段 | 类型 | 约束 |
| --- | --- | --- |
| `tables` | `{name: TablePayload}` | 规范化后的表 |
| `features` | `{group: [列名]}` | 特征分组 |
| `group_labels` | `{sample_id: group}` | 样本 → 分组 |
| `normalization` | string | `log2` / `zscore` / `minmax` / `combat` / **`TODO`** |
| `batch_corrected` | boolean | 是否做过批次校正 |
| `dropped_columns` | [string] | 被丢弃的列（须留痕） |
| `imputation` | string | 插补方法，未声明为 `TODO` |

> 语义未声明的列**保持 `TODO`**，不得猜测。猜测会产出静默错误的结果，比报错严重得多。

---

## 3. AnalysisResult（S3 分析）

| 字段 | 类型 | 约束 |
| --- | --- | --- |
| `method` | string | `lasso` / `cox` / `deseq2` / `limma` / `TODO` |
| `metrics` | `[MetricSpec]` | 统计指标 |
| `result_tables` | `{name: TablePayload}` | 结果表（供下游出图使用） |
| `model_kind` | string | 模型类型 |
| `n_features` | int? | 特征数 |
| `artifacts` | `[ArtifactRef]` | 产物引用 |
| `warnings` | [string] | 告警 |

**至少要有 `metrics` 或 `result_tables` 之一**，否则视为无结果。

### MetricSpec

| 字段 | 含义 |
| --- | --- |
| `name` * | 指标名，建议带括号标注原始名：`auc(roc_auc)` |
| `value` | 数值 |
| `ci_low` / `ci_high` | 置信区间 |
| `p_value` | p 值 |
| `n` | 样本数 |
| `unit` | 单位 |
| `method` | 统计方法，未声明为 `TODO` |

> **缺失的统计量一律留空并在校验中报 warning，不得编造。**

---

## 4. FigureArtifact（S4 出图）

| 字段 | 类型 | 约束 |
| --- | --- | --- |
| `image_path` * | string | 图形文件的**工作区相对路径**；禁止绝对路径与 `..` |
| `legend` | string | 图注；缺失写显式占位符 |
| `caption` | string | 正文引用用的 caption |
| `statistical_notes` | string | 统计说明；未声明为 `TODO`，**不得编造** |
| `source_analysis_id` | string | 来源 `AnalysisResult` 契约 id |
| `figure_type` | string | `volcano` / `km` / `forest` / `heatmap` / … |
| `panel_labels` | [string] | 分图编号（多面板时） |
| `dpi` / `width_in` / `height_in` | number? | 出图参数；由渲染器决定，取不到留空 |

> 五字段（`image_path` / `legend` / `caption` / `statistical_notes` / `source_analysis_id`）
> 是**硬要求**。图注缺失时写 `<!-- TODO -->` 显式占位符，而不是编一段看起来合理的说明。

---

## 5. ReportBundle（S5 报告）

| 字段 | 类型 | 约束 |
| --- | --- | --- |
| `title` | string | 报告标题 |
| `sections` | `[ReportSection]` | 章节 |
| `figures` | [string] | 引用的 `FigureArtifact` 契约 id 列表 |
| `tables` | [string] | 引用的 `AnalysisResult` 契约 id 列表 |
| `markdown_path` | string? | 落盘的 Markdown 路径 |
| `docx_path` | string? | 落盘的 DOCX 路径 |
| `citations` | [string] | 引用文献，**不得编造** |
| `reporting_guideline` | string | `CONSORT` / `STROBE` / `PRISMA` / `TRIPOD` / `TODO` |

**ReportSection**：`heading` / `body` / `figure_ids` / `table_ids` / `order`。

> 引用不存在的图或表 → 校验报 warning（悬空引用）。

---

## 6. 公共构件

### TablePayload
`columns: [ColumnSpec]` / `rows: [{...}]` / `n_rows` / `index_name` / `artifact: ArtifactRef?`

### ColumnSpec
`name` * / `dtype` / `unit` / `description` / `missing_count` / `unique_count`
（`dtype` / `unit` / `description` 未声明时为 `TODO`）

### ArtifactRef
`path`（**工作区相对路径，禁绝对与 `..`**）/ `format` / `size_bytes` / `sha256` / `exists`

### DocPayload
`text` / `title` / `language` / `sections: [ReportSection]` / `artifact`
