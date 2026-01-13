# CLAUDE.md - Claude Code 项目上下文

本文档为 Claude Code 提供项目架构、工作流机制和 Skills 使用指南，帮助 Claude 更好地理解和处理项目自动化任务。

## 项目概述

### 项目目的

这是一个基于 [claude-code-action](https://github.com/anthropics/claude-code-action) 和 [Claude Code Skills](https://code.claude.com/docs/en/skills) 的 GitHub 自动化模板项目。通过 GitHub Actions 和 Claude Skills，实现：

- 自动分析和处理 GitHub Issues（Bug 和功能需求）
- 自动生成技术方案和实现代码
- 自动进行 Pull Request 代码审查
- 完整的从问题到 PR 的自动化工作流

### 核心功能

1. **Issue 自动分析**
   - Bug 报告：自动定位根因，评估严重程度
   - 功能需求：技术可行性评估，方案设计

2. **代码自动实现**
   - 根据确认的技术方案自动实现代码
   - 创建分支、提交代码、创建 PR

3. **PR 自动审查**
   - 代码质量、安全性、性能检查
   - 结构化审查报告

### 技术栈

| 类别 | 技术 |
|------|------|
| **CI/CD** | GitHub Actions |
| **AI 能力** | Claude API (Anthropic) |
| **代码托管** | GitHub |
| **Skills 系统** | Claude Code Skills |
| **自动化工具** | GitHub CLI (`gh`), Git |

## 项目架构

### 目录结构

```
.
├── .claude/skills/          # Claude Code Skills 定义
│   ├── issue-analyze/       # Issue 分析 Skill
│   │   └── SKILL.md
│   ├── feature-plan/        # 需求评估 Skill
│   │   └── SKILL.md
│   ├── issue-implement/     # 代码实现 Skill
│   │   └── SKILL.md
│   └── pr-review/          # PR 审查 Skill
│       └── SKILL.md
├── .github/
│   ├── ISSUE_TEMPLATE/     # Issue 模板
│   │   ├── bug_report.yml  # Bug 报告模板
│   │   ├── prd.yml         # 产品需求文档模板
│   │   └── feature_request.yml  # 功能请求模板
│   └── workflows/          # GitHub Actions 工作流
│       ├── issue-analyze.yml    # Issue 分析工作流
│       ├── issue-implement.yml  # Issue 实现工作流
│       └── pr-review.yml       # PR 审查工作流
├── CLAUDE.md              # 本文档（项目上下文）
└── README.md              # 项目说明文档
```

### 关键模块

#### 1. Skills 模块 (`.claude/skills/`)

Skills 是 Claude Code 的功能模块，每个 Skill 定义了一个特定的自动化任务。

- **issue-analyze**: 分析 Issue，判断类型和严重程度，定位 Bug 根因
- **feature-plan**: 评估需求，提供技术方案、工作量预估和风险评估
- **issue-implement**: 根据确认的方案实现代码，创建 PR
- **pr-review**: 审查 PR，检查代码质量、安全性和性能

#### 2. 工作流模块 (`.github/workflows/`)

GitHub Actions 工作流负责触发 Claude Skills 执行。

- **issue-analyze.yml**: 当 Issue 创建时触发，根据标签调用相应的 Skill
- **issue-implement.yml**: 当 Issue 评论以 "OK" 开头时触发，执行代码实现
- **pr-review.yml**: 当 PR 创建或更新时触发，执行代码审查

#### 3. Issue 模板模块 (`.github/ISSUE_TEMPLATE/`)

提供结构化的 Issue 创建表单，确保问题描述完整。

### 依赖关系

```
GitHub Issue 创建
    ↓
issue-analyze.yml 工作流触发
    ↓
调用 issue-analyze 或 feature-plan Skill
    ↓
Claude 分析并发布技术方案评论
    ↓
用户评论 "OK" 确认
    ↓
issue-implement.yml 工作流触发
    ↓
调用 issue-implement Skill
    ↓
Claude 实现代码并创建 PR
    ↓
pr-review.yml 工作流触发
    ↓
调用 pr-review Skill
    ↓
Claude 发布审查报告
```

## 自动化工作流详解

### Issue 触发规则

#### Bug Issue 触发 `issue-analyze`

**触发条件:**
- Issue 创建时
- Issue 包含 `bug` 标签

**执行流程:**
1. 分析 Bug 类型和严重程度
2. 搜索相关代码，定位根因
3. 提供修复建议
4. 对于 `low`/`medium` 级别的简单问题，可能自动修复
5. 在 Issue 中发布分析报告

**输出:**
- 结构化的 Issue 分析报告（评论）
- 可选的修复分支和 PR

#### Enhancement Issue 触发 `feature-plan`

**触发条件:**
- Issue 创建时
- Issue 包含 `enhancement` 标签

**执行流程:**
1. 理解需求的核心目标
2. 搜索代码库，评估现有架构
3. 设计 1-3 个技术方案
4. 评估工作量和爆炸半径
5. 提供测试建议和风险评估
6. 在 Issue 中发布技术评估报告

**输出:**
- 结构化的技术评估报告（评论）
- 包含推荐方案、工作量预估、风险评估

### Issue 评论触发 `issue-implement`

**触发条件:**
- Issue 评论内容以 `OK` 或 `ok` 开头
- 评论作者是仓库成员

**执行流程:**
1. 使用 `gh issue view` 查看 Issue 及之前的分析评论
2. 提取技术方案和实现步骤
3. 创建 `fix/issue-N` 或 `feat/issue-N` 分支
4. 按照方案实现代码
5. 提交代码并推送到远程
6. 使用 `gh pr create` 创建 PR，关联 Issue
7. 在 Issue 中发布实现结果评论

**支持的确认命令:**
- `OK` - 确认方案，开始实现
- `OK 使用 TypeScript` - 确认并补充要求
- `OK 先只实现核心功能` - 确认并限定范围

**输出:**
- 新的代码分支
- Pull Request
- 结构化输出（分支名、PR 号、变更文件列表）

### PR 触发 `pr-review`

**触发条件:**
- PR 创建或更新时

**执行流程:**
1. 使用 `gh pr view` 获取 PR 信息
2. 使用 `gh pr diff` 查看变更内容
3. 阅读变更文件的完整上下文
4. 从代码质量、架构、安全性、性能四个维度审查
5. 问题分级：严重问题、重要问题、改进建议
6. 在 PR 中发布审查报告

**输出:**
- 结构化的 PR 审查报告（评论）
- 审查结论：APPROVE / REQUEST_CHANGES / COMMENT

## Skills 使用指南

### issue-analyze

**功能**: 分析 GitHub Issue，判断类型和严重程度，定位 Bug 根因

**输入**:
- Issue 标题、内容、标签
- 代码库访问权限

**处理逻辑**:
1. 分类：bug / feature / question / documentation
2. 严重程度评估（Bug）：critical / high / medium / low
3. 根因分析：使用 Grep 搜索错误信息，Read 查看上下文
4. 自动修复判断：仅对 low/medium 级别的简单问题自动修复

**输出格式**:
```json
{
  "issue_type": "bug|feature|question|documentation",
  "severity": "critical|high|medium|low|N/A",
  "root_cause": "问题根因分析",
  "affected_files": ["file1.ts", "file2.ts"],
  "fix_complexity": "simple|moderate|complex",
  "auto_fixed": true|false,
  "branch_name": "fix/issue-123",
  "summary": "简要总结"
}
```

**最佳实践**:
- 给出具体的文件名和行号（如 `src/auth.ts:42`）
- 对 critical/high 级别问题，只分析不修复
- 使用简体中文输出

### feature-plan

**功能**: 评估产品需求，分析技术可行性，提供实现方案

**输入**:
- Issue 标题和需求描述
- 代码库访问权限

**处理逻辑**:
1. 需求理解：总结核心目标和关键功能点
2. 代码库分析：搜索相关模块，评估现有架构
3. 方案设计：提供 1-3 个可行方案
4. 工作量评估：预估改动文件数和复杂度
5. 爆炸半径评估：分析影响范围
6. 风险评估：识别潜在风险和缓解措施
7. 测试建议：单元测试、集成测试、E2E 测试范围

**工作量级别定义**:
- `small`: 1-5 文件，简单修改
- `medium`: 6-15 文件，中等复杂度
- `large`: 16-30 文件，较高复杂度
- `extra-large`: 30+ 文件，架构级变更

**爆炸半径定义**:
- `isolated`: 仅影响单一模块
- `contained`: 影响 2-3 个相关模块
- `moderate`: 影响多个模块，需回归测试
- `wide`: 影响核心流程，需全面测试

**输出格式**:
```json
{
  "requirement_summary": "需求核心摘要",
  "feature_points": ["功能点1", "功能点2"],
  "recommended_approach": {
    "name": "方案名称",
    "description": "方案描述",
    "pros": ["优势1"],
    "cons": ["劣势1"]
  },
  "scope": "small|medium|large|extra-large",
  "estimated_files": 10,
  "blast_radius": "isolated|contained|moderate|wide",
  "implementation_steps": [
    {"step": "步骤描述", "complexity": "simple|moderate|complex"}
  ]
}
```

**最佳实践**:
- 基于实际代码库给出具体建议
- 如需求不清晰，明确列出待澄清问题
- 方案应该可落地，避免过度设计

### issue-implement

**功能**: 根据 Issue 分析结果实现代码，创建分支并提交 PR

**输入**:
- Issue 编号
- 用户补充说明（来自评论）

**处理逻辑**:
1. 使用 `gh issue view {number} --comments` 查看之前的分析
2. 提取问题类型（bug/feature）和技术方案
3. 创建分支：`fix/issue-N` 或 `feat/issue-N`
4. 按照方案逐步实现代码
5. 提交代码：使用 Conventional Commits 格式
6. 推送分支：`git push -u origin 分支名`
7. 创建 PR：使用 `gh pr create`，Body 包含 `Closes #N`
8. 回复 Issue：通知用户 PR 已创建

**分支命名规则**:
- Bug 修复：`fix/issue-{number}`
- 新功能：`feat/issue-{number}`

**Commit Message 格式**:
```
类型: 简要描述 (#issue_number)

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

类型包括：`fix`, `feat`, `refactor`, `docs`, `test`, `chore`

**输出格式**:
```json
{
  "status": "success|failed|blocked",
  "branch_name": "fix/issue-123",
  "pr_number": 456,
  "pr_url": "https://github.com/owner/repo/pull/456",
  "files_changed": ["file1.ts", "file2.ts"],
  "summary": "简要描述实现内容"
}
```

**最佳实践**:
- 确保代码可运行，无语法错误
- 如遇到阻塞，设置 `status: "blocked"` 并说明原因
- PR 标题和 commit message 保持一致
- 添加必要的错误处理

### pr-review

**功能**: 自动审查 Pull Request，检查代码质量、安全性和性能

**输入**:
- PR 编号
- 变更内容

**审查维度**:

1. **代码质量**
   - 代码风格、复杂度、重复代码
   - 命名规范、错误处理
   - 潜在 Bug、边界条件

2. **架构设计**
   - 项目结构一致性
   - 依赖合理性
   - 设计模式、关注点分离

3. **安全性**
   - SQL 注入、XSS、敏感信息泄露
   - 权限验证、输入验证

4. **性能**
   - N+1 查询、内存泄漏
   - 不必要的计算、缓存使用

**问题分级**:
- 🔴 **严重问题 (Critical)**: 安全漏洞、数据丢失风险、崩溃风险
- 🟠 **重要问题 (Important)**: 性能问题、代码质量问题、测试覆盖不足
- 🟢 **改进建议 (Suggestions)**: 代码风格改进、命名优化、文档补充

**审查结论**:
- `APPROVE`: 批准合并（无问题或仅有建议）
- `REQUEST_CHANGES`: 需要修改（存在严重或重要问题）
- `COMMENT`: 仅评论（有问题但不阻塞合并）

**输出格式**:
```json
{
  "conclusion": "APPROVE|REQUEST_CHANGES|COMMENT",
  "summary": "简要总结（100字以内）",
  "critical_count": 0,
  "important_count": 2,
  "suggestion_count": 3,
  "security_issues": ["安全问题1"],
  "performance_issues": ["性能问题1"],
  "highlights": ["代码亮点1"]
}
```

**最佳实践**:
- 给出具体的文件名和行号
- 对好的实践也要给予肯定
- 保持客观、专业的态度
- 使用 `gh pr comment` 发布单条综合评论（不要多条或行内评论）

## 开发指南

### 如何添加新 Skill

1. **创建 Skill 目录**

```bash
mkdir -p .claude/skills/your-skill-name
```

2. **编写 SKILL.md**

```markdown
---
name: your-skill-name
description: 简要描述 Skill 的功能和使用场景
---

# Skill 标题

## 功能说明

## 使用流程

## 输出格式

## 注意事项
```

3. **创建对应的 GitHub Actions 工作流**

在 `.github/workflows/` 中创建工作流文件：

```yaml
name: Your Skill Workflow

on:
  issues:
    types: [opened]  # 或其他触发事件

jobs:
  execute:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      issues: write
      pull-requests: write
      id-token: write
    steps:
      - name: Checkout repository
        uses: actions/checkout@v5
        with:
          fetch-depth: 0

      - name: Execute Skill
        uses: anthropics/claude-code-action@v1
        env:
          ANTHROPIC_BASE_URL: ${{ secrets.ANTHROPIC_BASE_URL }}
        with:
          github_token: ${{ github.token }}
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            使用 your-skill-name skill 处理任务。

            ## 输入信息
            ...
          claude_args: |
            --allowedTools "Bash(gh:*),Read,Write,Edit,Glob,Grep,Task"
            --json-schema '{...}'
```

4. **测试 Skill**

创建测试 Issue 或触发相应事件，验证 Skill 是否正常工作。

### 如何修改现有 Skill

1. **修改 SKILL.md**

编辑 `.claude/skills/{skill-name}/SKILL.md`，更新功能逻辑或输出格式。

2. **更新工作流（如需要）**

如果修改了 Skill 的输入输出格式，需要同步更新对应的工作流文件中的：
- `prompt` 内容
- `--json-schema` 定义
- `--allowedTools` 权限

3. **测试修改**

触发工作流，验证修改是否生效。

### 调试和测试建议

#### 本地测试 Skill

使用 Claude Code CLI 在本地测试 Skill：

```bash
# 安装 Claude Code CLI
npm install -g @anthropic-ai/claude-code

# 执行 Skill
claude-code --skill issue-analyze --prompt "测试输入"
```

#### 查看工作流日志

在 GitHub Actions 页面查看工作流执行日志：
- 检查触发条件是否满足
- 查看 Claude 的输出和错误信息
- 验证结构化输出是否符合 Schema

#### 常见问题

**Q: Skill 没有被触发？**
- 检查工作流的触发条件（`on` 部分）
- 确认 Issue 标签是否正确
- 查看 Actions 页面是否有错误

**Q: Claude 输出格式不正确？**
- 检查 `--json-schema` 定义是否准确
- 在 SKILL.md 中明确说明输出格式要求
- 使用结构化输出示例

**Q: 权限不足？**
- 检查工作流的 `permissions` 配置
- 确认 `--allowedTools` 包含所需工具
- 验证 Secrets 配置是否正确

### Skill 开发最佳实践

1. **明确的输入输出定义**
   - 在 SKILL.md 中清晰定义输入参数
   - 提供结构化输出 Schema
   - 包含示例

2. **详细的执行流程说明**
   - 列出具体的执行步骤
   - 说明使用哪些工具（Bash, Read, Grep 等）
   - 提供命令示例

3. **错误处理和边界情况**
   - 说明如何处理异常情况
   - 定义失败状态的输出格式
   - 提供降级方案

4. **简体中文输出**
   - 所有面向用户的输出使用简体中文
   - 保持专业、客观的语气
   - 避免使用 emoji（除非明确要求）

5. **具体而非抽象**
   - 给出文件名和行号（如 `src/auth.ts:42`）
   - 使用代码示例说明问题
   - 提供可操作的修复建议

## 配置说明

### 必需的 Secrets

在 GitHub 仓库的 **Settings → Secrets → Actions** 中配置：

| Secret | 必填 | 说明 |
|--------|------|------|
| `ANTHROPIC_API_KEY` | ✅ | Anthropic API 密钥，从 [Anthropic Console](https://console.anthropic.com/) 获取 |
| `ANTHROPIC_BASE_URL` | ❌ | 自定义 API 端点（如使用代理），默认为官方端点 |

### 工作流权限

每个工作流需要的 GitHub Token 权限：

```yaml
permissions:
  contents: write      # 读写代码库
  issues: write        # 读写 Issues
  pull-requests: write # 读写 Pull Requests
  id-token: write      # OIDC 认证
```

### 修改确认命令

如需修改 `issue-implement` 触发的确认命令，编辑 `.github/workflows/issue-implement.yml`：

```yaml
if: |
  startsWith(github.event.comment.body, 'OK') ||
  startsWith(github.event.comment.body, 'ok') ||
  startsWith(github.event.comment.body, '确认')  # 添加新命令
```

## 注意事项

### 使用限制

1. **API 配额**
   - Claude API 有调用限制，频繁触发可能达到配额
   - 考虑添加速率限制或手动审批步骤

2. **代码安全**
   - 自动生成的代码需要人工审查
   - 不建议直接自动合并 PR

3. **工作流成本**
   - GitHub Actions 有免费额度限制
   - 监控使用量，避免超额

### 最佳实践

1. **渐进式自动化**
   - 先从分析和建议开始
   - 逐步增加自动修复的范围
   - 保留人工决策点

2. **清晰的 Issue 描述**
   - 使用提供的 Issue 模板
   - 提供足够的上下文信息
   - 包含复现步骤（Bug）或需求背景（Feature）

3. **及时反馈**
   - 在 Issue 评论中补充说明
   - 对 Claude 的方案进行验证
   - 发现问题及时修正

4. **代码审查**
   - 即使有自动审查，也需要人工 Review
   - 关注安全性和性能问题
   - 确保代码符合项目规范

## 常见工作流示例

### Bug 修复完整流程

```
1. 用户创建 Bug Issue（使用 bug_report.yml 模板）
   ↓
2. issue-analyze.yml 自动触发
   ↓
3. Claude 使用 issue-analyze Skill 分析
   - 搜索相关代码
   - 定位根因
   - 评估严重程度
   ↓
4. Claude 在 Issue 评论中发布分析报告
   ↓
5. 用户查看报告，评论 "OK" 确认
   ↓
6. issue-implement.yml 自动触发
   ↓
7. Claude 使用 issue-implement Skill 实现
   - 创建 fix/issue-N 分支
   - 修复代码
   - 提交并推送
   - 创建 PR
   ↓
8. pr-review.yml 自动触发
   ↓
9. Claude 使用 pr-review Skill 审查
   - 检查代码质量、安全性、性能
   - 发布审查报告
   ↓
10. 开发者 Review 并合并 PR
   ↓
11. Issue 自动关闭（因 PR 中包含 "Closes #N"）
```

### 功能需求完整流程

```
1. 用户创建 Feature Issue（使用 prd.yml 或 feature_request.yml 模板）
   ↓
2. issue-analyze.yml 自动触发
   ↓
3. Claude 使用 feature-plan Skill 评估
   - 理解需求
   - 搜索代码库
   - 设计技术方案
   - 评估工作量和风险
   ↓
4. Claude 在 Issue 评论中发布技术评估报告
   ↓
5. 用户查看报告，评论 "OK 使用方案 A" 确认
   ↓
6. issue-implement.yml 自动触发
   ↓
7. Claude 使用 issue-implement Skill 实现
   - 创建 feat/issue-N 分支
   - 实现功能代码
   - 提交并推送
   - 创建 PR
   ↓
8. pr-review.yml 自动触发
   ↓
9. Claude 使用 pr-review Skill 审查
   - 检查代码质量、架构、安全性、性能
   - 发布审查报告
   ↓
10. 开发者 Review 并合并 PR
   ↓
11. Issue 自动关闭
```

## 扩展阅读

- [Claude Code 官方文档](https://code.claude.com/docs)
- [Claude Code Skills 指南](https://code.claude.com/docs/en/skills)
- [claude-code-action GitHub](https://github.com/anthropics/claude-code-action)
- [GitHub Actions 文档](https://docs.github.com/en/actions)
- [Conventional Commits](https://www.conventionalcommits.org/)

---

本文档持续更新中。如有问题或建议，请创建 Issue 讨论。
