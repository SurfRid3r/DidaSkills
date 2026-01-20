# Claude Skills 创建指南

本指南基于 Anthropic Agent Skills 开放标准，帮助您创建符合规范的 Skills。

## 参考资料

- **官方规范**: https://agentskills.io/specification
- **官方仓库**: https://github.com/anthropics/skills
- **规范首页**: https://agentskills.io/home

---

## 目录结构

一个技能目录至少包含 `SKILL.md` 文件：

```
skill-name/
├── SKILL.md          # 必需：技能定义文件
├── scripts/          # 可选：可执行代码
├── references/       # 可选：参考文档
├── assets/           # 可选：静态资源
└── LICENSE.txt       # 可选：许可证文件
```

---

## SKILL.md 格式规范

### Frontmatter（必需）

`SKILL.md` 必须以 YAML frontmatter 开头：

```yaml
---
name: skill-name
description: 技能描述和使用场景
---
```

### 可选字段

```yaml
---
name: pdf-processing
description: 从 PDF 文件中提取文本和表格，填写表单，合并文档
license: Apache-2.0
metadata:
  author: example-org
  version: "1.0"
---
```

### 字段约束

| 字段 | 必需 | 约束 |
|------|------|------|
| `name` | 是 | 最多 64 字符。仅限小写字母、数字和连字符。不能以连字符开头或结尾。 |
| `description` | 是 | 最多 1024 字符。非空。描述技能功能和使用场景。 |
| `license` | 否 | 许可证名称或对捆绑许可证文件的引用。 |
| `compatibility` | 否 | 最多 500 字符。指示环境要求（目标产品、系统包、网络访问等）。 |
| `metadata` | 否 | 任意键值映射，用于附加元数据。 |
| `allowed-tools` | 否 | 技能可使用的事先批准的工具的空格分隔列表。（实验性） |

#### name 字段规则

- 必须为 1-64 个字符
- 只能包含 unicode 小写字母数字字符和连字符（`a-z` 和 `-`）
- 不能以 `-` 开头或结尾
- 不能包含连续的连字符（`--`）
- 必须与父目录名称匹配

**有效示例**：
- `pdf-processing`
- `task-manager`
- `my-skill-123`

**无效示例**：
- `PDF-Processing`（不允许大写）
- `-pdf`（不能以连字符开头）
- `pdf--processing`（不允许连续连字符）

#### description 字段规则

- 必须为 1-1024 个字符
- 应描述技能功能和使用场景
- 应包含特定关键词，帮助代理识别相关任务

**好的示例**：
```yaml
description: 从 PDF 文件中提取文本和表格，填写 PDF 表单，合并多个 PDF。当处理 PDF 文档或用户提到 PDF、表单或文档提取时使用。
```

**不好的示例**：
```yaml
description: 帮助处理 PDF。
```

#### license 字段

```yaml
license: Proprietary. LICENSE.txt has complete terms
```

#### compatibility 字段

```yaml
compatibility: Requires git, docker, jq, and access to the internet
```

#### metadata 字段

```yaml
metadata:
  author: example-org
  version: "1.0"
```

#### allowed-tools 字段（实验性）

```yaml
allowed-tools: Bash(git:*) Bash(jq:*) Read
```

---

## 正文内容

Frontmatter 之后的 Markdown 正文包含技能指令。没有格式限制。编写任何有助于代理有效执行任务的内容。

**推荐部分**：
- 分步说明
- 输入输出示例
- 常见边界情况

---

## 可选目录说明

### scripts/ 目录

包含代理可运行的可执行代码。脚本应该：
- 自包含或清晰记录依赖关系
- 包含有用的错误消息
- 优雅地处理边界情况

支持的语言取决于代理实现。常见选项包括 Python、Bash 和 JavaScript。

### references/ 目录

包含代理可阅读的附加文档：
- `REFERENCE.md` - 详细技术参考
- `FORMS.md` - 表单模板或结构化数据格式
- 特定领域文件（`finance.md`、`legal.md` 等）

保持单个参考文件聚焦。代理按需加载这些文件，因此较小的文件意味着更少的上下文使用。

### assets/ 目录

包含静态资源：
- 模板（文档模板、配置模板）
- 图片（图表、示例）
- 数据文件（查找表、模式）

---

## 渐进式披露原则

技能应构建为高效使用上下文：

1. **元数据**（~100 tokens）：`name` 和 `description` 字段在启动时为所有技能加载
2. **指令**（建议 < 5000 tokens）：完整的 `SKILL.md` 正文在技能激活时加载
3. **资源**（按需）：文件（如 `scripts/`、`references/` 或 `assets/` 中的）仅在需要时加载

**组织建议**：
- 保持主 `SKILL.md` 在 500 行以下
- 将详细参考资料移至单独文件（`references/` 目录）
- 复杂代码逻辑移至 `scripts/` 目录

### 模式示例

**模式 1: 高层指南 + 引用**
```markdown
# PDF 处理

## 快速开始
使用 pdfplumber 提取文本...

## 高级功能
- 表单填写: 见 [FORMS.md](references/FORMS.md)
- API 参考: 见 [REFERENCE.md](references/REFERENCE.md)
```

**模式 2: 领域特定组织**
```
bigquery-skill/
├── SKILL.md (概览和导航)
└── references/
    ├── finance.md
    ├── sales.md
    └── product.md
```

---

## 文件引用

在技能中引用其他文件时，使用从技能根目录开始的相对路径：

```markdown
详见 [参考指南](references/REFERENCE.md)

运行提取脚本：
scripts/extract.py
```

保持文件引用距离 `SKILL.md` 只有一层。避免深层嵌套的引用链。

---

## 验证

使用 skills-ref 参考库验证技能：

```bash
skills-ref validate ./my-skill
```

这将检查：
- YAML frontmatter 格式
- 技能命名约定
- 目录结构
- 描述完整性

---

## 实战示例

参考本仓库的 `ticktick-task-management` 技能：

```
ticktick-task-management/
├── SKILL.md              # 主技能定义
├── LICENSE.txt           # 许可证
├── .env.template         # 环境变量模板
├── requirements.txt      # Python 依赖
├── scripts/              # CLI 脚本
│   ├── ticktick.py       # 主入口
│   ├── auth/             # 认证模块
│   ├── api/              # API 服务层
│   └── utils/            # 工具函数
└── references/           # 参考文档
    └── examples.md       # 使用示例
```

### Frontmatter 示例

```yaml
---
name: ticktick-task-management
description: TickTick/滴答清单统一管理 CLI，支持任务、项目、标签、评论和习惯管理。当用户需要以下操作时使用: (1) 创建/更新/删除/完成任务，(2) 管理项目，(3) 整理标签，(4) 添加任务评论，(5) 追踪习惯。
---
```

### SKILL.md 结构示例

```markdown
# TickTick 任务管理

## 快速参考

| 分类 | 命令 |
|------|------|
| **项目管理** | `list`, `create`, `update`, `delete` |
| **任务管理** | `list`, `create`, `update`, `complete` |

## 初始设置

```bash
pip install -r requirements.txt
cp .env.template .env
```

## 常用命令

### 创建任务
```bash
python scripts/ticktick.py tasks create --title "任务" --project-name "工作"
```

## 高级工作流

完整工作流示例见 [references/examples.md](references/examples.md)
```

---

## 最佳实践

### 1. 描述写作

- **说明功能和使用场景**：让 Claude 知道何时激活此技能
- **包含关键词**：使用用户可能说的词汇
- **具体而非抽象**：避免模糊的描述

### 2. 内容组织

- **SKILL.md 保持简洁**：核心指令和快速参考
- **详细内容放 references/**：API 文档、工作流示例
- **代码放 scripts/**：可重用的脚本和工具

### 3. 错误处理

- 提供清晰的错误消息
- 包含故障排除步骤
- 链接到相关文档

### 4. 示例质量

- 提供可运行的完整示例
- 展示常见用例
- 包含预期输出

---

## 相关资源

- [Agent Skills 规范](https://agentskills.io/specification)
- [Anthropic Skills 仓库](https://github.com/anthropics/skills)
- [技能创建器](https://github.com/anthropics/skills/tree/main/skills/skill-creator)
