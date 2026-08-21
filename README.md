# PR Implementation for Codex

`pr-implementation` 是一个仓库级 Codex Skill：它将一份 PR requirements 和
acceptance criteria，推进到**可验证、已独立审查、可交付**的本地改动。

它不是“再写一个 reviewer”，也不是 GitHub PR 管理工具。它负责把需求、实现、
测试、验收和已有的 `$code-review` 串成同一条交付链路，避免“测试绿了，但 PR 仍然
无法合并”的情况。

## 它解决什么问题

普通开发流程很容易出现这些断点：

- 代码实现了，但遗漏了某条 acceptance criterion；
- 只跑了局部测试，没有验证兼容性、格式、类型或构建；
- 历史人工 review 已指出过同类问题，但下一次 PR 又重复发生；
- 修复 review finding 后没有复测，就直接声称完成；
- 代码遵循规范，却实现错了需求；或需求实现了，却违背仓库约定。

`pr-implementation` 将这些断点变成明确的交付门禁。

## 核心优势

### 1. 从需求到证据，而不只是从需求到代码

Skill 会先将 requirements、acceptance criteria、constraints 和 non-goals 提炼为
当前 PR checklist，再建立“需求 → 实现位置 → 验证方式”的映射。每个验收项都必须
标记为 `VERIFIED`、`PARTIALLY VERIFIED` 或 `NOT VERIFIED`；关键项未验证时不能宣布完成。

### 2. 强制独立审查，而不是自我确认

实现和验证后，Skill **必须调用已有的 `$code-review`**。`$code-review` 独立检查
仓库标准与需求/spec；`pr-implementation` 只负责提供上下文、修复 finding 并重新验证，
不会复制或伪造 code review 能力。

### 3. 让历史 review 经验真正参与当前 PR

`.codex/review-lessons.md` 保存提炼后的经验，而非原始评论。当前 PR 触及相关场景时，
Skill 必须把 lesson ID、它保护的不变量和验证证据写入实施计划。例如状态相关字段需要
覆盖完整的合法/非法组合矩阵，而不是只检查字段格式。

### 4. 有闭环，也有边界

发现 review 问题后，流程是“修复 → 定向测试 → 重新验收 → 必要时再次审查”。最多进行
三轮完整 review pass，避免无限循环；若仍有 blocker 或 major correctness issue，会如实
报告而不是假装完成。

## 与其他方式的区别

| 方式 | 主要职责 | 缺少什么 |
| --- | --- | --- |
| 普通编码助手 | 根据请求修改代码 | 容易缺少验收证据、独立审查与修复闭环。 |
| `$code-review` | 独立检查一个已有 diff 的标准与 spec 问题 | 不负责理解需求后实施、测试和修复。 |
| `$pr-implementation` | 协调 requirements 到最终本地交付的完整生命周期 | 不创建远端 PR、不 push、不 merge，也不替代 `$code-review`。 |

## 安装

将本仓库中的 `.codex/` 复制到需要使用该流程的目标仓库：

```text
.codex/
├── review-lessons.md
└── skills/
    └── pr-implementation/
        └── SKILL.md
```

运行环境还需要已有的 `$code-review` Skill。

## 使用方式

在目标仓库根目录开启 Codex 对话，直接提供当前 PR context；不需要先把需求保存到固定
Markdown 文件。

```text
$pr-implementation

PR requirements:
- 为订单查询增加 customerId 筛选。
- 保持现有响应结构不变。

Acceptance criteria:
- 合法 customerId 仅返回该客户的订单。
- 非法参数返回现有格式的 400 响应。
- 未传 customerId 时保持原有行为。
- 新行为有对应测试。

Constraints:
- 不新增依赖。
- 不进行无关重构。

Base branch: main
```

显式给出 `Base branch`（例如 `main` 或 `origin/main`）可让 `$code-review` 用
`git diff <base>...HEAD` 审查实际会合并的 diff。未提供时，Skill 会在能可靠判断的
情况下使用跟踪/默认分支；无法确认时会在审查门禁前询问。

## Review lessons

`review-lessons.md` 用于保存过去人工 review 提炼出的、可泛化且可验证的经验。它不是：

- 原始 review comment 的归档；
- 自动抓取 GitHub 评论的系统；
- 数据库、向量库或长期 memory service。

v1 不会自动写入 lessons；团队可在复盘后手动添加。每条 lesson 应说明其保护的不变量、
适用场景和所需验证方式。

## 最终交付门禁

只有同时满足以下条件，Skill 才会报告 PR 完成：

- 核心 requirements 和 acceptance criteria 均为 `VERIFIED`；
- 相关测试与仓库要求的检查实际通过；
- `$code-review` 已执行；
- 没有未解决的 blocker、major correctness issue、已知架构违规或明显回归；
- 最终 `git diff` 与 `git status` 没有无关文件、debug code 或临时 workaround。

## 有意不做的事

该 Skill 仅管理本地开发交付流程，不会自动：创建/修改远端 PR、push、merge、抓取
GitHub mentor comments、创建 reviewer subagent，或建立长期 memory 基础设施。
