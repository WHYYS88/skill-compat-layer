# 贡献指南 / Contributing

感谢你关注 **skill-compat-layer**。这个仓库是**规格类项目**，没有可执行代码 —— 所以这里的"贡献"主要是：把规格写得更清楚、把实例补得更全、把踩过的坑记下来。

## 什么样的贡献最有价值

| 类型 | 例子 |
| --- | --- |
| **补充实现经验** | 你按这套规格做了实现，遇到规格没覆盖到的情况 —— 提 Issue 说明，或直接补进对应文档 |
| **补实例** | 用你自己的技能合集跑通一遍，产出新的契约实例（尤其是不同领域的 payload 长什么样） |
| **指出歧义** | 某条约束你读出了两种意思 —— 这很有价值，说明写法有问题 |
| **修正错误** | 路径失效、Schema 与文档不一致、数字对不上 |
| **翻译** | 目前是中文 + 英文，其他语言欢迎 |

**不太需要的**：新增规则性的约束。本蓝本刻意保持克制 —— 少一条约束往往比多一条好。

## 提交流程

1. Fork 本仓库
2. 克隆并建分支：```bash
   git clone https://github.com/<你的用户名>/skill-compat-layer.git
   cd skill-compat-layer
   git checkout -b docs/your-change
   ```
3. 改完自查（见下）
4. 推送并提 Pull Request

## 改完请自查这几点

- [ ] **中英双语的对应段落是否都改了** —— README 是双语的，只改一边会造成不对称
- [ ] **文档里引用的路径是否都存在** —— 改名或移动文件后，检查其他文档有没有指向它
- [ ] **Schema 与文档是否仍然一致** —— 改了 `workflow/contracts.md` 的字段表，就要确认 `workflow/schemas/` 里的定义跟得上
- [ ] **行尾是 LF** —— `.gitattributes` 已设 `eol=lf`，正常提交即可
- [ ] **没有引入绝对路径** —— 文档里不出现 `C:\\`、`/Users/` 这类机器相关路径

## 目录结构

```
skill-compat-layer/
├── workflow/                    # 规格与实例
│   ├── overview.md              # 流水线总览、信封、血缘、幂等
│   ├── contracts.md             # 五份契约的载荷字段
│   ├── criteria.md              # 四级断点判定尺度
│   ├── interventions.md         # 九种人工介入原因码
│   ├── safety.md                # 安全接口（熔断器）
│   ├── adapters.md              # 适配器实现契约
│   ├── capability.md            # 能力与限制声明
│   ├── prior-art.md             # 相关工作与先技术
│   ├── orchestration.yaml       # 流程编排定义
│   ├── schemas/                 # 9 个 JSON Schema
│   └── examples/                # 契约实例与断点请求实例
├── .github/                     # Issue / PR 模板
├── .env.example                 # 环境变量模板
├── CHANGELOG.md
├── LICENSE                      # Apache-2.0
├── NOTICE.md                    # 版权与第三方归属
├── SECURITY.md
└── README.md
```

## 提交信息

用 [Conventional Commits](https://www.conventionalcommits.org/lang/zh-CN/)：

```
<type>(<scope>): <描述>
```

| type | 用于 |
| --- | --- |
| `docs` | 绝大多数改动都属于这类 |
| `feat` | 新增规格章节、新增 Schema、新增实例 |
| `fix` | 修正错误、失效链接、字段不一致 |
| `chore` | 仓库配置、模板、gitignore |

示例：

```
docs(contracts): 补充 S3 载荷中 result_tables 的用途说明
feat(examples): 增加一份不同领域的 AnalysisResult 实例
fix(readme): 修正英文段落的失效链接
```

## 关于上游归属

如果贡献涉及上游技能库的**事实性引用**（统计数字、技能形态判断），请一并说明**基于哪个 commit**。
本仓库当前锚点是 `686e09dfdb`（2026-09-17）。数字会随上游变化，注明锚点才能复核。

## 行为准则

参与本项目即表示你同意遵守 [Contributor Covenant](https://www.contributor-covenant.org/zh-cn/version/2/1/code_of_conduct/)。
简言之：**对事不对人**。技术分歧很正常，指出问题请附带依据。
