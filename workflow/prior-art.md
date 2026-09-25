# 相关工作与先技术 / References and Prior Art

> 这份清单区分两类：**验证了问题存在的**（说明我们的出发点是公认的），
> 和**已经给出解法的**（说明我们和它们的区别在哪）。
> 只列确实查阅过的，不列"可能相关"的。

---

## 1. GraSP —— 最接近的邻居

**GraSP: Graph-Structured Skill Compositions for LLM Agents**
Tianle Xia, Lingxiang Hu, Yiding Sun, Ming Xu, Lan Xu, Siying Wang, Wei Xu, Jie Jiang — **Tencent**
arXiv:2604.17870 [cs.CL], 2026-04-20
<https://arxiv.org/abs/2604.17870>

### 它说了什么

**① 瓶颈已经从"技能有没有"转移到"技能怎么编排"**

> "Skill availability is no longer the bottleneck. ... the bottleneck has shifted from *skill availability* to *skill orchestration*."

**② 根因是检索与执行之间缺一层"编译"**

> "The root cause is the absence of a **compilation stage** between skill retrieval and skill execution."

检索回答"哪些技能相关"，执行回答"现在做这一步"，中间**没人回答"这些技能怎么互相依赖、最小且因果有序的计划是什么"**。

**③ 它的解法：把扁平技能集编译成带类型的 DAG**

```
GraSP = (V, E)，V = 技能调用节点，E ⊆ V × {state, data, order} × V
约束：无环、源到汇可达、目标完备、每个节点可执行
```

四个阶段：记忆条件检索 → **DAG 编译** → 带验证的执行 + 局部修复 → 基于置信度的路由回退。

**④ 局部修复把重规划从 O(N) 降到 O(d^h)**

依赖边保留了因果关系，所以**一个节点失败只影响它的拓扑后继**，不必整条重来。
五个带类型的修复算子。

**⑤ 实测**

四个交互式基准（ALFWorld / ScienceWorld / WebShop / InterCode）× 八个 LLM 骨干，
**在所有配置下都优于** ReAct、Reflexion、ExpeL 与扁平技能基线：

| 指标 | 结果 |
| --- | --- |
| 奖励提升 | 最高 **+19 分**（相对最强基线） |
| 环境步数 | 最多**减少 41%** |
| 复杂度相关性 | 任务越复杂优势越大 |
| 鲁棒性 | 对**技能过度检索**与**技能质量下降**都稳 |

> 结论句："structured orchestration—not larger skill libraries—is the key to reliable agent execution."

### 我们和它的区别（重要，别混为一谈）

两边结论一致（编排才是关键），但**解决的问题不同**：

| | GraSP | 本仓库 |
| --- | --- | --- |
| 面对的东西 | 已有类型化技能库（`L = (S, R)`，每个技能有 schema） | 真实世界里**没有类型声明**的合集（601 个里 401 个未声明输入格式） |
| 何时构建结构 | 每个任务**动态编译**一张 DAG | **固定的五阶段**编排定义，一次写好反复用 |
| 失败怎么办 | **自动局部修复**（五个算子） | **抛结构化断点，交给人** |
| 上游要不要改 | 需要技能可类型化、可验证 | **不改原件**，适配器包在外面 |
| 依赖的前提 | 技能有 precondition / effect 声明 | 承认**大部分技能没有**，所以先补契约层 |

**一句话**：GraSP 假设技能是"可编译的零件"；本仓库处理的是"零件连规格书都不全"的存量资产。
两者可以叠加 —— 先把规格书补齐（本仓库），再上动态编译与自动修复（GraSP 那条路）。

### 它没覆盖的

论文自己列的局限：DAG 假设排除了循环执行模式；验证只在四个文本交互环境。
**另外它假定技能已经有类型声明** —— 而存量合集里，这件事本身要先补（见 README §2）。

---

## 2. 验证了问题存在的其他来源

| 来源 | 说了什么 | 对本仓库的意义 |
| --- | --- | --- |
| **Real Faults in Model Context Protocol (MCP) Software: a Comprehensive Taxonomy** | 学术论文**专门给 MCP 故障做分类学** | 接口不匹配多到值得系统研究，不是个别现象 |
| **AWS AgentCore Gateway** 文档 | 明说 MCP 服务器之间**没有显式同步**时，网关会提供**过期的工具定义**，影响延迟与可靠性 | 平台自己也承认 schema 会漂移 |
| Hitachi 白皮书 *Skills are all you need* | 技能作为智能体能力单元 | 行业方向一致的信号 |

---

## 3. 走"定新标准"路线的提案（与本仓库不同路）

| 来源 | 做法 |
| --- | --- |
| `djm204/agent-skills` issue #91 —— "Evolving into a Universal Agent Skills Framework" | 提议 `skill.yaml` 结构化清单，把技能做成可安装的技能包 |
| `openclaw` issue #78294 —— "ASFS skill format + Handoff Protocol" | 提议技能格式 + 跨智能体交接协议 |

**共同点**：都要求**生态先统一**才能生效。
**本仓库的差别**：不要求任何人改自己的技能，**对存量资产直接补一层**。

---

## 4. 引用格式

```bibtex
@article{xia2026grasp,
  title  = {GraSP: Graph-Structured Skill Compositions for LLM Agents},
  author = {Xia, Tianle and Hu, Lingxiang and Sun, Yiding and Xu, Ming and
            Xu, Lan and Wang, Siying and Xu, Wei and Jiang, Jie},
  journal = {arXiv preprint arXiv:2604.17870},
  year   = {2026},
  url    = {https://arxiv.org/abs/2604.17870}
}
```

---

## 5. 查阅说明

以上来源经检索与原文阅读确认，**引用的数字与结论句均取自论文原文**（摘要、引言、结论）。
检索范围为一轮定向搜索，**非穷尽综述**；若要把"这一层少见"作为对外论断，建议再做一轮系统检索。
