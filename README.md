# Claude Code Automation Template

基于 [claude-code-action](https://github.com/anthropics/claude-code-action) 和 [Claude Code Skills](https://code.claude.com/docs/en/skills) 的 GitHub 自动化模板。

## 功能

| 触发器 | Issue 类型 | Skill | 功能 |
|--------|-----------|-------|------|
| Issue 创建 | `bug` 标签 | `issue-analyze` | 分析 Bug，定位根因，自动修复 |
| Issue 创建 | `enhancement` 标签 | `feature-plan` | 技术评估，方案设计，工作量预估 |
| PR 创建/更新 | - | `pr-review` | 代码审查 |

## 项目结构

```
.
├── .claude/skills/
│   ├── issue-analyze/SKILL.md    # Bug 分析
│   ├── feature-plan/SKILL.md     # 需求评估
│   └── pr-review/SKILL.md        # PR 审查
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.yml        # Bug 报告 → 触发 issue-analyze
│   │   ├── prd.yml               # 需求/PRD → 触发 feature-plan
│   │   └── feature_request.yml   # 功能请求 → 触发 feature-plan
│   └── workflows/
│       ├── issue-analyze.yml     # Issue 分析（根据标签分发）
│       └── pr-review.yml         # PR 审查
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

- **报告 Bug**：New Issue → Bug Report
- **提交需求**：New Issue → 需求/PRD
- **代码审查**：提交 PR 自动触发

## 工作流程

```
Issue 创建
    │
    ├─ 标签含 "bug" ──────→ issue-analyze skill
    │                           ├─ 问题分类
    │                           ├─ 根因分析
    │                           └─ 自动修复（low/medium）
    │
    └─ 标签含 "enhancement" ─→ feature-plan skill
                                ├─ 技术方案
                                ├─ 工作量预估
                                └─ 爆炸半径评估
```

## 结构化输出示例

### Bug 分析
```json
{
  "issue_type": "bug",
  "severity": "medium",
  "root_cause": "src/utils.ts:42 未处理空值",
  "auto_fixed": true,
  "branch_name": "fix/issue-123"
}
```

### 需求评估
```json
{
  "requirement_summary": "添加 OAuth 登录",
  "scope": "medium",
  "blast_radius": "contained",
  "implementation_steps": [
    {"step": "安装依赖", "complexity": "simple"},
    {"step": "创建 OAuth 模块", "complexity": "moderate"}
  ]
}
```

## 自定义

### 修改 Skill 行为

编辑 `.claude/skills/{name}/SKILL.md`

### 添加新 Skill

```bash
mkdir -p .claude/skills/my-skill
cat > .claude/skills/my-skill/SKILL.md << 'EOF'
---
name: my-skill
description: 描述功能和使用场景
---

# Skill 内容
EOF
```

## 许可证

MIT
