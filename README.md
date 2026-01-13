# Claude Code Automation Template

一个基于 [claude-code-action](https://github.com/anthropics/claude-code-action) 和 [Claude Code Skills](https://code.claude.com/docs/en/skills) 的 GitHub 自动化模板项目。

## 功能

| 触发器 | 工作流 | Skill | 功能 |
|--------|--------|-------|------|
| Issue 创建 | `issue-analyze.yml` | `issue-analyze` | 自动分析 Bug，定位根因，小型问题自动修复 |
| Discussion 创建 | `discussion-feature-plan.yml` | `feature-plan` | 评估需求可行性，输出技术方案和工作量预估 |
| PR 创建/更新 | `pr-review.yml` | `pr-review` | 自动代码审查，输出结构化审查报告 |

## 项目结构

```
.
├── .claude/
│   └── skills/                      # Claude Code Skills
│       ├── issue-analyze/
│       │   └── SKILL.md             # Issue 分析 Skill
│       ├── feature-plan/
│       │   └── SKILL.md             # 需求评估 Skill
│       └── pr-review/
│           └── SKILL.md             # PR 审查 Skill
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.yml           # Bug 报告模板
│   │   ├── feature_request.yml      # 功能请求模板
│   │   └── config.yml               # Issue 模板配置
│   └── workflows/
│       ├── issue-analyze.yml        # Issue 分析工作流
│       ├── discussion-feature-plan.yml  # 需求评估工作流
│       └── pr-review.yml            # PR 审查工作流
└── README.md
```

## 快速开始

### 1. 使用此模板创建仓库

点击 **Use this template** 按钮创建你的仓库。

### 2. 配置 Secrets

在仓库的 **Settings → Secrets and variables → Actions** 中添加：

| Secret 名称 | 必填 | 说明 |
|-------------|------|------|
| `ANTHROPIC_API_KEY` | ✅ | Anthropic API 密钥 |
| `ANTHROPIC_BASE_URL` | ❌ | 自定义 API 端点（可选） |

### 3. 启用 Discussions（可选）

如需使用需求评估功能：

1. 进入 **Settings → General → Features**
2. 勾选 **Discussions**
3. 创建以下分类（任选其一或多个）：
   - `Ideas`
   - `Feature Request`
   - `RFC`
   - `需求`

### 4. 开始使用

- **报告 Bug**：创建 Issue 并使用 Bug Report 模板
- **提交需求**：创建 Discussion 并选择 Ideas/Feature Request 分类
- **代码审查**：提交 PR 时自动触发

## Skills 详情

Skills 是模块化的指令包，定义了 Claude 如何执行特定任务。

### issue-analyze

分析 GitHub Issue，判断问题类型和严重程度，定位 Bug 根因。

**主要功能：**
- 问题分类：bug / feature / question / documentation
- 严重程度评估：critical / high / medium / low
- 根因分析：搜索代码库定位问题代码
- 自动修复：对于 low/medium 的简单 bug，自动创建修复分支

**结构化输出：**
```json
{
  "issue_type": "bug",
  "severity": "medium",
  "root_cause": "在 src/utils.ts:42 处，未处理空值情况",
  "affected_files": ["src/utils.ts"],
  "auto_fixed": true,
  "branch_name": "fix/issue-123"
}
```

### feature-plan

评估产品需求（PRD），提供技术方案和工作量预估。

**主要功能：**
- 需求理解：核心目标、功能点、边界条件
- 技术方案：1-3 个可行方案，含优缺点分析
- 影响评估：修改文件、受影响模块、爆炸半径
- 工作量预估：small / medium / large / extra-large

**结构化输出：**
```json
{
  "requirement_summary": "用户登录功能增加第三方 OAuth 支持",
  "recommended_approach": {
    "name": "OAuth 2.0 集成",
    "description": "使用 passport.js 集成多个 OAuth 提供商"
  },
  "scope": "medium",
  "blast_radius": "contained",
  "implementation_steps": [
    {"step": "安装依赖", "complexity": "simple"},
    {"step": "创建 OAuth 模块", "complexity": "moderate"}
  ]
}
```

### pr-review

自动审查 Pull Request，检查代码质量、安全性和性能。

**审查维度：**
- 代码质量：风格、复杂度、命名、错误处理
- 架构设计：结构一致性、依赖合理性
- 安全性：SQL 注入、XSS、敏感信息泄露
- 性能：N+1 查询、内存泄漏、冗余计算

**结构化输出：**
```json
{
  "conclusion": "REQUEST_CHANGES",
  "summary": "发现 1 个安全问题和 2 个代码质量问题",
  "critical_count": 1,
  "important_count": 2,
  "suggestion_count": 3,
  "security_issues": ["SQL 注入风险 - src/db.ts:56"]
}
```

## 自定义配置

### 修改 Skill 行为

编辑 `.claude/skills/{skill-name}/SKILL.md` 文件来定制 Skill 的行为。

### 添加新 Skill

1. 创建目录：`.claude/skills/your-skill-name/`
2. 创建 `SKILL.md` 文件，包含 YAML 前置数据：

```yaml
---
name: your-skill-name
description: 描述此 Skill 的功能和使用场景。
---

# Skill 内容

详细的指令和流程...
```

### 修改触发条件

编辑工作流文件中的 `on` 部分：

```yaml
# 示例：只在添加特定标签时触发
on:
  issues:
    types: [labeled]
```

### 修改 Discussion 分类

编辑 `discussion-feature-plan.yml` 中的条件：

```yaml
if: |
  contains(fromJSON('["Ideas", "Feature Request", "RFC", "需求", "你的分类"]'), github.event.discussion.category.name)
```

## 后续集成

工作流输出的结构化 JSON 可用于后续自动化：

```yaml
# 示例：根据分析结果发送通知
- name: Send Notification
  if: always()
  run: |
    RESULT='${{ steps.analyze.outputs.structured_output }}'
    curl -X POST https://your-webhook-url \
      -H "Content-Type: application/json" \
      -d "$RESULT"
```

## 权限说明

工作流需要以下权限：

| 权限 | 用途 |
|------|------|
| `contents: write` | 创建分支和推送代码 |
| `issues: write` | 在 Issue 下发表评论 |
| `pull-requests: write` | 在 PR 下发表评论 |
| `discussions: write` | 在 Discussion 下发表评论 |

## 常见问题

### Q: Claude 分析超时怎么办？

在 `claude_args` 中添加 `--max-turns` 参数限制分析轮次。

### Q: 如何使用自定义 API 端点？

设置 `ANTHROPIC_BASE_URL` secret 为你的 API 端点地址。

### Q: 如何禁用自动修复？

编辑 `.claude/skills/issue-analyze/SKILL.md`，移除自动修复相关的指令。

### Q: Skill 没有被识别怎么办？

确保：
1. `SKILL.md` 文件包含正确的 YAML 前置数据
2. `name` 字段只包含小写字母、数字和连字符
3. 文件路径正确：`.claude/skills/{name}/SKILL.md`

## 许可证

MIT
