# Claude Code Automation Template

基于 [claude-code-action](https://github.com/anthropics/claude-code-action) 和 [Claude Code Skills](https://code.claude.com/docs/en/skills) 的 GitHub 自动化模板。

## 功能

| 触发器 | 条件 | Skill | 功能 |
|--------|------|-------|------|
| Issue 创建 | `bug` 标签 | `issue-analyze` | 分析 Bug，定位根因 |
| Issue 创建 | `enhancement` 标签 | `feature-plan` | 技术评估，方案设计 |
| Issue 评论 | 以 `OK` 开头 | `issue-implement` | 实现代码，创建 PR |
| PR 创建/更新 | - | `pr-review` | 代码审查 |

## 完整工作流程

```
1. 创建 Issue（Bug 或需求）
        ↓
2. Claude 自动分析，发布技术方案
        ↓
3. 用户评论 "OK" 或 "OK 补充说明..."
        ↓
4. Claude 实现代码，创建 PR
        ↓
5. Claude 自动审查 PR
```

## 项目结构

```
.
├── .claude/skills/
│   ├── issue-analyze/SKILL.md     # Bug 分析
│   ├── feature-plan/SKILL.md      # 需求评估
│   ├── issue-implement/SKILL.md   # 代码实现
│   └── pr-review/SKILL.md         # PR 审查
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.yml         # Bug 报告
│   │   ├── prd.yml                # 需求/PRD
│   │   └── feature_request.yml    # 功能请求
│   └── workflows/
│       ├── issue-analyze.yml      # Issue 分析
│       ├── issue-implement.yml    # Issue 实现
│       └── pr-review.yml          # PR 审查
└── README.md
```

## 快速开始

### 1. 使用模板创建仓库

点击 **Use this template**

### 2. 配置 Secrets

在 **Settings → Secrets → Actions** 添加：

| Secret | 必填 | 说明 |
|--------|------|------|
| `ANTHROPIC_API_KEY` | ✅ | Anthropic API 密钥 |
| `ANTHROPIC_BASE_URL` | ❌ | 自定义 API 端点 |

### 3. 开始使用

1. **创建 Issue**：选择 Bug Report 或 需求/PRD 模板
2. **等待分析**：Claude 自动分析并发布方案
3. **确认实现**：评论 `OK` 或 `OK 需要补充的说明`
4. **等待 PR**：Claude 实现代码并创建 PR
5. **审查合并**：PR 自动触发代码审查

## 确认命令

在 Issue 评论中使用：

| 命令 | 说明 |
|------|------|
| `OK` | 确认方案，开始实现 |
| `OK 使用 TypeScript` | 确认并补充要求 |
| `OK 先只实现核心功能` | 确认并限定范围 |

## 结构化输出

### Issue 实现
```json
{
  "status": "success",
  "branch_name": "feat/issue-123",
  "pr_number": 456,
  "pr_url": "https://github.com/owner/repo/pull/456",
  "files_changed": ["src/auth.ts", "src/utils.ts"],
  "summary": "实现 OAuth 登录功能"
}
```

## 自定义

### 修改 Skill 行为

编辑 `.claude/skills/{name}/SKILL.md`

### 修改确认命令

编辑 `.github/workflows/issue-implement.yml` 中的触发条件：

```yaml
if: startsWith(toLower(github.event.comment.body), 'ok')
```

## 许可证

MIT
