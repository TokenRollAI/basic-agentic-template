---
name: issue-analyze
description: 分析 GitHub Issue，判断类型和严重程度，定位 Bug 根因，自动修复简单问题。当 Issue 被创建或需要分析问题时使用。
---

# Issue 分析

自动分析 GitHub Issue，提供问题分类、根因定位和修复建议。

## 分析流程

### 1. 问题分类

根据 Issue 内容判断类型：

| 类型 | 描述 |
|------|------|
| `bug` | 代码缺陷，功能异常 |
| `feature` | 新功能请求 |
| `question` | 使用问题或疑问 |
| `documentation` | 文档相关问题 |

### 2. 严重程度评估（仅 Bug）

| 级别 | 描述 | 处理方式 |
|------|------|----------|
| `critical` | 系统崩溃或数据丢失 | 仅分析，需人工处理 |
| `high` | 核心功能无法使用 | 仅分析，需人工处理 |
| `medium` | 功能受损但有变通方案 | 可自动修复 |
| `low` | 轻微问题或样式问题 | 可自动修复 |

### 3. 根因分析

对于 Bug 类型：

1. 使用 `Grep` 搜索错误信息相关代码
2. 使用 `Read` 查看相关文件上下文
3. 分析调用链和数据流
4. 确定问题根本原因

### 4. 自动修复

**仅在以下条件满足时执行：**
- 问题类型为 `bug`
- 严重程度为 `low` 或 `medium`
- 修复方案明确且风险可控

**修复步骤：**

```bash
# 1. 创建修复分支
git checkout -b fix/issue-{issue_number}

# 2. 实现修复代码
# 使用 Edit 工具修改文件

# 3. 推送分支
git push -u origin fix/issue-{issue_number}
```

## 输出格式

### Issue 评论模板

```markdown
## 🔍 Issue 分析报告

### 📋 基本信息
- **类型**: {bug/feature/question/documentation}
- **严重程度**: {critical/high/medium/low/N/A}

### 🔎 根因分析
{问题原因详细说明}

**相关代码位置：**
- `path/to/file.ts:123` - {问题描述}

### 💡 解决方案
{修复建议或已完成的修复}

### 🔗 相关链接
{如果创建了修复分支，提供 PR 创建链接}

---
🤖 由 Claude 自动生成
```

### 结构化输出 Schema

```json
{
  "issue_type": "bug|feature|question|documentation",
  "severity": "critical|high|medium|low|N/A",
  "root_cause": "问题根因分析",
  "affected_files": ["file1.ts", "file2.ts"],
  "fix_complexity": "simple|moderate|complex",
  "auto_fixed": true|false,
  "branch_name": "fix/issue-123",
  "pr_url": "https://github.com/owner/repo/compare/main...fix/issue-123",
  "summary": "简要总结"
}
```

## 注意事项

- 所有输出使用**简体中文**
- 给出具体的**文件名和行号**
- 对于 `critical`/`high` 级别问题，**只分析不修复**
- 使用 `gh issue comment` 发布分析结果
