---
name: skill-creator
description: 创建有效 Skill 的指南。当用户想要创建新 Skill（或更新现有 Skill）以使用专业知识、工作流程或工具集成扩展 Codex 的功能时，应使用此 Skill。
metadata:
  short-description: 创建或更新 Skill
---

# Skill Creator

本 Skill 提供创建有效 Skill 的指导。

## 关于 Skill

Skill 是模块化的、自包含的文件夹，通过提供专业的工作流程和工具来扩展 Codex 的功能。将它们视为特定领域或任务的"入职指南"——它们将 Codex 从通用 Agent 转变为配备有模型无法完全拥有的程序性知识的专业 Agent。

### Skill 提供什么

1. 专业工作流程 - 特定领域的多步骤程序
2. 工具集成 - 使用特定文件格式或 API 的说明
3. 领域专业知识 - 公司特定的知识、架构、业务逻辑
4. 捆绑资源 - 用于复杂和重复任务的脚本、参考和资产

## 核心原则

### 简洁是关键

上下文窗口是一种公共资源。Skill 与 Codex 需要的其他所有内容共享上下文窗口：系统提示、对话历史、其他 Skill 的元数据以及实际的用户请求。

**默认假设：Codex 已经非常聪明。** 只添加 Codex 还没有的上下文。质疑每条信息："Codex 真的需要这个解释吗？"和"这段文字值得它的 token 成本吗？"

优先选择简洁的示例而非冗长的解释。

### 设置适当的自由度

根据任务的脆弱性和可变性匹配特异性级别：

**高自由度（基于文本的说明）**：当多种方法有效、决策取决于上下文或启发式指导方法时使用。

**中自由度（伪代码或带参数的脚本）**：当存在首选模式、允许一些变化或配置影响行为时使用。

**低自由度（特定脚本，很少参数）**：当操作脆弱且容易出错、一致性至关重要或必须遵循特定序列时使用。

将 Codex 视为探索路径：有悬崖的狭窄桥梁需要特定的护栏（低自由度），而开阔的田野允许多条路线（高自由度）。

### Skill 的解剖结构

每个 Skill 由一个必需的 SKILL.md 文件和可选的捆绑资源组成：

```
skill-name/
├── SKILL.md (必需)
│   ├── YAML frontmatter 元数据 (必需)
│   │   ├── name: (必需)
│   │   └── description: (必需)
│   └── Markdown 说明 (必需)
├── agents/ (推荐)
│   └── openai.yaml - Skill 列表和芯片的 UI 元数据
└── 捆绑资源 (可选)
    ├── scripts/          - 可执行代码（Python/Bash 等）
    ├── references/       - 旨在根据需要加载到上下文中的文档
    └── assets/           - 在输出中使用的文件（模板、图标、字体等）
```

#### SKILL.md (必需)

每个 SKILL.md 包含：

- **Frontmatter** (YAML)：包含 `name` 和 `description` 字段。这些是 Codex 读取以确定何时使用 Skill 的唯一字段，因此清楚全面地描述 Skill 是什么以及何时应该使用它非常重要。
- **正文** (Markdown)：使用 Skill 的说明和指导。仅在 Skill 触发后加载（如果有的话）。

#### Agents 元数据 (推荐)

- Skill 列表和芯片的面向 UI 的元数据
- 在生成值之前阅读 references/openai_yaml.md 并遵循其描述和约束
- 创建：通过阅读 Skill 生成面向人类的 `display_name`、`short_description` 和 `default_prompt`
- 通过将值作为 `--interface key=value` 传递给 `scripts/generate_openai_yaml.py` 或 `scripts/init_skill.py` 来确定性生成
- 更新时：验证 `agents/openai.yaml` 是否仍与 SKILL.md 匹配；如果过时则重新生成
- 仅当用户明确提供时才包含其他可选界面字段（图标、品牌颜色）
- 有关字段定义和示例，请参阅 references/openai_yaml.md

#### 捆绑资源 (可选)

##### 脚本 (`scripts/`)

可执行代码（Python/Bash 等），用于需要确定性可靠性或重复重写的任务。

- **何时包含**：当相同的代码被重复重写或需要确定性可靠性时
- **示例**：用于 PDF 旋转任务的 `scripts/rotate_pdf.py`
- **好处**：Token 高效、确定性、可以在不加载到上下文中的情况下执行
- **注意**：脚本可能仍需要被 Codex 读取以进行修补或环境特定调整

##### 参考资料 (`references/`)

文档和参考材料，旨在根据需要加载到上下文中，以告知 Codex 的过程和思考。

- **何时包含**：对于 Codex 应该在工作时参考的文档
- **示例**：用于财务架构的 `references/finance.md`、用于公司 NDA 模板的 `references/mnda.md`、用于公司政策的 `references/policies.md`、用于 API 规范的 `references/api_docs.md`
- **用例**：数据库架构、API 文档、领域知识、公司政策、详细工作流程指南
- **好处**：保持 SKILL.md 精简，仅在 Codex 确定需要时加载
- **最佳实践**：如果文件很大（>10k 字），请在 SKILL.md 中包含 grep 搜索模式
- **避免重复**：信息应该存在于 SKILL.md 或 references 文件中，而不是两者都有。优先选择 references 文件来获取详细信息，除非它确实是 Skill 的核心——这可以保持 SKILL.md 精简，同时使信息可被发现而不会占用上下文窗口。仅在 SKILL.md 中保留基本的程序性说明和工作流程指导；将详细的参考材料、架构和示例移至 references 文件。

##### 资产 (`assets/`)

不打算加载到上下文中，而是在 Codex 产生的输出中使用的文件。

- **何时包含**：当 Skill 需要将在最终输出中使用的文件时
- **示例**：用于品牌资产的 `assets/logo.png`、用于 PowerPoint 模板的 `assets/slides.pptx`、用于 HTML/React 样板文件的 `assets/frontend-template/`、用于排版的 `assets/font.ttf`
- **用例**：模板、图像、图标、样板代码、字体、被复制或修改的示例文档
- **好处**：将输出资源与文档分离，使 Codex 可以使用文件而不将它们加载到上下文中

#### Skill 中不应包含什么

Skill 应该只包含直接支持其功能的必要文件。不要创建无关的文档或辅助文件，包括：

- README.md
- INSTALLATION_GUIDE.md
- QUICK_REFERENCE.md
- CHANGELOG.md
- 等等。

Skill 应该只包含 AI Agent 完成当前工作所需的信息。它不应该包含关于创建它的过程的辅助上下文、设置和测试程序、面向用户的文档等。创建额外的文档文件只会增加混乱和困惑。

### 渐进式披露设计原则

Skill 使用三级加载系统来高效管理上下文：

1. **元数据（name + description）** - 始终在上下文中（~100 字）
2. **SKILL.md 正文** - 当 Skill 触发时（<5k 字）
3. **捆绑资源** - 根据 Codex 的需要（无限制，因为脚本可以在不读入上下文窗口的情况下执行）

#### 渐进式披露模式

将 SKILL.md 正文保持在基本内容内，并在 500 行以内，以最小化上下文膨胀。当接近此限制时，将内容拆分为单独的文件。将内容拆分为其他文件时，从 SKILL.md 引用它们并清楚描述何时阅读它们非常重要，以确保 Skill 的读者知道它们存在以及何时使用它们。

**关键原则：** 当 Skill 支持多种变体、框架或选项时，仅在 SKILL.md 中保留核心工作流程和选择指导。将变体特定的细节（模式、示例、配置）移至单独的参考文件中。

**模式 1：带有参考资料的高级指南**

```markdown
# PDF 处理

## 快速开始

使用 pdfplumber 提取文本：
[代码示例]

## 高级功能

- **表单填写**：有关完整指南，请参阅 [FORMS.md](FORMS.md)
- **API 参考**：有关所有方法，请参阅 [REFERENCE.md](REFERENCE.md)
- **示例**：有关常见模式，请参阅 [EXAMPLES.md](EXAMPLES.md)
```

Codex 仅在需要时加载 FORMS.md、REFERENCE.md 或 EXAMPLES.md。

**模式 2：按领域组织**

对于具有多个领域的 Skill，按领域组织内容以避免加载不相关的上下文：

```
bigquery-skill/
├── SKILL.md（概述和导航）
└── reference/
    ├── finance.md（收入、账单指标）
    ├── sales.md（机会、管道）
    ├── product.md（API 使用、功能）
    └── marketing.md（活动、归因）
```

当用户询问销售指标时，Codex 只读取 sales.md。

同样，对于支持多个框架或变体的 Skill，按变体组织：

```
cloud-deploy/
├── SKILL.md（工作流程 + 提供商选择）
└── references/
    ├── aws.md（AWS 部署模式）
    ├── gcp.md（GCP 部署模式）
    └── azure.md（Azure 部署模式）
```

当用户选择 AWS 时，Codex 只读取 aws.md。

**模式 3：条件细节**

显示基本内容，链接到高级内容：

```markdown
# DOCX 处理

## 创建文档

使用 docx-js 创建新文档。请参阅 [DOCX-JS.md](DOCX-JS.md)。

## 编辑文档

对于简单编辑，直接修改 XML。

**对于修订**：请参阅 [REDLINING.md](REDLINING.md)
**对于 OOXML 细节**：请参阅 [OOXML.md](OOXML.md)
```

Codex 仅在用户需要这些功能时读取 REDLINING.md 或 OOXML.md。

**重要指南：**

- **避免深度嵌套的引用** - 保持引用从 SKILL.md 只有一层深度。所有引用文件都应该直接从 SKILL.md 链接。
- **结构化较长的引用文件** - 对于超过 100 行的文件，在顶部包含目录，以便 Codex 在预览时可以看到完整范围。

## Skill 创建流程

Skill 创建涉及以下步骤：

1. 用具体示例理解 Skill
2. 规划可重用的 Skill 内容（脚本、参考、资产）
3. 初始化 Skill（运行 init_skill.py）
4. 编辑 Skill（实现资源并编写 SKILL.md）
5. 验证 Skill（运行 quick_validate.py）
6. 根据实际使用进行迭代

按顺序遵循这些步骤，仅当有明确理由说明它们不适用时才跳过。

### Skill 命名

- 仅使用小写字母、数字和连字符；将用户提供的标题规范化为连字符形式（例如，"Plan Mode" -> `plan-mode`）。
- 生成名称时，生成少于 64 个字符的名称（字母、数字、连字符）。
- 优先使用描述动作的简短动词短语。
- 通过在工具前命名来命名空间，以提高清晰度或触发（例如，`gh-address-comments`、`linear-address-issue`）。
- 将 Skill 文件夹命名为与 Skill 名称完全相同。

### 第 1 步：用具体示例理解 Skill

仅当 Skill 的使用模式已经清楚理解时跳过此步骤。即使在使用现有 Skill 时，此步骤也很有价值。

要创建有效的 Skill，清楚地理解 Skill 将如何被使用的具体示例。这种理解可以来自直接的用户示例或经过用户反馈验证的生成示例。

例如，在构建 image-editor Skill 时，相关问题包括：

- "image-editor Skill 应该支持什么功能？编辑、旋转还有其他吗？"
- "你能举一些如何使用此 Skill 的示例吗？"
- "我可以想象用户要求像'去除这张图片的红眼'或'旋转这张图片'这样的事情。你能想象到此 Skill 的其他使用方式吗？"
- "用户会说什么来触发此 Skill？"

为避免让用户不知所措，避免在单条消息中问太多问题。从最重要的问题开始，并根据需要跟进以获得更好的效果。

当对 Skill 应支持的功能有清晰的感觉时，结束此步骤。

### 第 2 步：规划可重用的 Skill 内容

要将具体示例转化为有效的 Skill，通过以下方式分析每个示例：

1. 考虑如何从头开始执行示例
2. 确定在重复执行这些工作流程时，哪些脚本、参考和资产会有帮助

示例：在构建 `pdf-editor` Skill 来处理"帮我旋转此 PDF"等查询时，分析显示：

1. 旋转 PDF 每次都需要重写相同的代码
2. 在 Skill 中存储 `scripts/rotate_pdf.py` 脚本会很有帮助

示例：在为"构建一个待办应用"或"构建一个仪表板来跟踪我的步数"等查询设计 `frontend-webapp-builder` Skill 时，分析显示：

1. 编写前端 Web 应用每次都需要相同的样板 HTML/React
2. 在 Skill 中存储包含样板 HTML/React 项目文件的 `assets/hello-world/` 模板会很有帮助

示例：在为"今天有多少用户登录？"等查询构建 `big-query` Skill 时，分析显示：

1. 查询 BigQuery 每次都需要重新发现表架构和关系
2. 在 Skill 中存储记录表架构的 `references/schema.md` 文件会很有帮助

要建立 Skill 的内容，分析每个具体示例以创建要包含的可重用资源列表：脚本、参考和资产。

### 第 3 步：初始化 Skill

此时，是时候实际创建 Skill 了。

仅当正在开发的 Skill 已经存在时才跳过此步骤。在这种情况下，继续下一步。

从头开始创建新 Skill 时，始终运行 `init_skill.py` 脚本。该脚本方便地生成一个新的模板 Skill 目录，自动包含 Skill 所需的一切，使 Skill 创建过程更加高效和可靠。

用法：

```bash
scripts/init_skill.py <skill-name> --path <output-directory> [--resources scripts,references,assets] [--examples]
```

示例：

```bash
scripts/init_skill.py my-skill --path skills/public
scripts/init_skill.py my-skill --path skills/public --resources scripts,references
scripts/init_skill.py my-skill --path skills/public --resources scripts --examples
```

该脚本：

- 在指定路径创建 Skill 目录
- 生成具有正确 frontmatter 和 TODO 占位符的 SKILL.md 模板
- 使用通过 `--interface key=value` 传递的 Agent 生成的 `display_name`、`short_description` 和 `default_prompt` 创建 `agents/openai.yaml`
- 根据 `--resources` 可选创建资源目录
- 设置 `--examples` 时可选添加示例文件

初始化后，根据需要自定义 SKILL.md 并添加资源。如果你使用了 `--examples`，请替换或删除占位符文件。

通过阅读 Skill 生成 `display_name`、`short_description` 和 `default_prompt`，然后将它们作为 `--interface key=value` 传递给 `init_skill.py` 或使用以下命令重新生成：

```bash
scripts/generate_openai_yaml.py <path/to/skill-folder> --interface key=value
```

仅当用户明确提供它们时才包含其他可选界面字段。有关完整的字段描述和示例，请参阅 references/openai_yaml.md。

### 第 4 步：编辑 Skill

编辑（新生成的或现有的）Skill 时，请记住 Skill 是为另一个 Codex 实例使用的。包含对 Codex 有益且不明显的信息。考虑什么程序性知识、领域特定的细节或可重用资产可以帮助另一个 Codex 实例更有效地执行这些任务。

#### 从可重用的 Skill 内容开始

要开始实现，请从上面确定的可重用资源开始：`scripts/`、`references/` 和 `assets/` 文件。请注意，此步骤可能需要用户输入。例如，在实现 `brand-guidelines` Skill 时，用户可能需要提供要存储在 `assets/` 中的品牌资产或模板，或要存储在 `references/` 中的文档。

必须通过实际运行来测试添加的脚本，以确保没有错误并且输出符合预期。如果有很多类似的脚本，只需要测试一个有代表性的样本，以确保它们都有效，同时平衡完成时间。

如果你使用了 `--examples`，请删除 Skill 不需要的任何占位符文件。只创建实际需要的资源目录。

#### 更新 SKILL.md

**编写指南：** 始终使用祈使/不定式形式。

##### Frontmatter

编写带有 `name` 和 `description` 的 YAML frontmatter：

- `name`：Skill 名称
- `description`：这是 Skill 的主要触发机制，帮助 Codex 理解何时使用 Skill。
  - 包含 Skill 做什么以及何时使用它的具体触发器/上下文。
  - 在此处包含所有"何时使用"信息 - 不在正文中。正文仅在触发后加载，因此正文中的"何时使用此 Skill"部分对 Codex 没有帮助。
  - `docx` Skill 的示例描述："全面的文档创建、编辑和分析，支持修订、评论、格式保留和文本提取。在 Codex 需要处理专业文档（.docx 文件）时使用：（1）创建新文档，（2）修改或编辑内容，（3）处理修订，（4）添加评论或任何其他文档任务"

不要在 YAML frontmatter 中包含任何其他字段。

##### 正文

编写使用 Skill 及其捆绑资源的说明。

### 第 5 步：验证 Skill

Skill 开发完成后，验证 Skill 文件夹以尽早发现基本问题：

```bash
scripts/quick_validate.py <path/to/skill-folder>
```

验证脚本检查 YAML frontmatter 格式、必需字段和命名规则。如果验证失败，请修复报告的问题并再次运行命令。

### 第 6 步：迭代

测试 Skill 后，用户可能会请求改进。通常这发生在使用 Skill 之后，对 Skill 的表现有新鲜的上下文。

**迭代工作流程：**

1. 在实际任务上使用 Skill
2. 注意困难或低效之处
3. 确定应如何更新 SKILL.md 或捆绑资源
4. 实施更改并再次测试
