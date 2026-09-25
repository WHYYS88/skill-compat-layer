# 契约实例 / Contract Examples

> **这些是真实跑出来的产物，不是手写的示意。**
> 用 [../orchestration.yaml](../orchestration.yaml) 定义的流程走完一遍，
> 每一阶段的契约存档在这里。六个文件全部通过 [../schemas/](../schemas/) 的校验。

用途：**照着它们做，就能产出结构一致的兼容层。**
字段含义看 [../contracts.md](../contracts.md)，这里给的是"拼起来长什么样"。

---

## 文件

| 文件 | 阶段 | 关键内容 |
| --- | --- | --- |
| [S1-simulated_dataset.json](S1-simulated_dataset.json) | S1 生成 | 设计声明 + 逐列生成方式；`is_synthetic: true`、`contains_real_patient_data: false` 硬约束 |
| [S2-clean_dataset.json](S2-clean_dataset.json) | S2 处理 | 规范化后的表 + **逐列判定留痕**（`column_status` / `column_warnings`） |
| [S3-analysis_result.json](S3-analysis_result.json) | S3 分析 | `metrics` 指标 + `result_tables` 结果表；缺失统计量留空并记 warning |
| [S4-figure_artifact.json](S4-figure_artifact.json) | S4 出图 | **五字段齐备**：image_path / legend / caption / statistical_notes / source_analysis_id |
| [S5-report_bundle.json](S5-report_bundle.json) | S5 报告 | 章节 + 图文引用 + 报告规范 |
| [HIR-intervention_request.json](HIR-intervention_request.json) | 断点 | **一次真实触发的人工介入请求**：要画森林图但上游没有置信区间 |

---

## 读的时候注意这几处

### 1. 信封是统一的

六个文件的**外层结构完全一样**，都是 13 个字段：

```
id  source_skill  source_skill_path  stage  format  schema_version  payload
metadata  provenance  human_interventions  created_at  updated_at  notes
```

差异只在 `payload` 里。这就是"统一信封"的意义：编排器与审计工具只需要处理一种外层结构。

### 2. 血缘是串起来的

每个契约的 `provenance[-1].inputs` 指向上游契约的 `id`：

```
S1-sim-contract-fc3227fa
  └─ S2-clean-s1-sim-contract-773528d8      inputs: [S1-sim-contract-fc3227fa]
       └─ S3-analyze-s2-clean-s1-sim-28233fd4
            └─ S4-figure-s3-analyze-s2-cl-a5612444
                 └─ S5-report-s4-figure-s3-ana-b7d45ea2
```

从任一产物都能沿 `inputs` 回溯到最初的模拟数据。

### 3. 断点请求长什么样

[HIR-intervention_request.json](HIR-intervention_request.json) 是一次真实触发。它回答了四个问题：

| 要素 | 这个例子里 |
| --- | --- |
| **卡在哪** | `blocker`：图型 forest 需要 estimate / ci_low / ci_high，上游只有 auc / sensitivity / specificity |
| **属于哪一类** | `blocker_code: payload_schema_mismatch` |
| **需要什么格式** | `required_format`：包含三列的统计表（CSV）；`required_fields` 列出具体字段 |
| **建议怎么做** | `suggested_actions`：补跑能产出该统计量的技能，或人工提供结果表 |
| **证据** | `evidence`：上游实际有哪些 metrics |

注意 `severity: "blocking"` —— 这个断点会**中断本阶段并触发回滚**，而不是打条日志继续。

---

## 怎么用它们

**① 当作实现的验收基准**

一个能对上的实现，其产出应当能通过 [../schemas/](../schemas/) 校验，
并且结构与这些实例一致。字段对不上就是接口没对齐。

```bash
# 用 JSON Schema 校验自己的产出
python -m jsonschema -i your_contract.json workflow/schemas/analysis_result.schema.json
```

**② 当作端到端的对照**

按 [../orchestration.yaml](../orchestration.yaml) 走完 S1→S5，逐阶段比对自己的产物。
差异通常暴露在两个地方：`payload` 字段缺失，或 `provenance` 没接上。

**③ 换领域时改哪里**

这些实例绑定了具体的表名与指标（如 `clinical` 表、`auc(roc_auc)` 指标）。
换到别的技能合集时，**改的是 `payload` 里的内容与 `orchestration.yaml` 里的阶段划分；
信封结构、血缘规则、断点四要素都不用动** —— 那部分是通用的。

---

## 生成方式（可复现）

这些实例由本项目在 **601 技能的医学研究技能合集**（[aipoch/medical-research-skills]，
MIT，Copyright (c) 2026 AIpoch）上跑通一次得到：模拟数据 → 清洗 → 分析 → 出图 → 报告。

- 数据全部为**模拟数据**，不含任何真实患者信息
- 绝对路径已全部替换为工作区相对路径
- 断点实例来自一次**真实的**森林图缺置信区间触发

[aipoch/medical-research-skills]: https://github.com/aipoch/medical-research-skills
