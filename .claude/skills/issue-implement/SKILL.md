---
name: issue-implement
description: 根据 Issue 分析结果实现代码，创建分支并提交 PR。当用户在 Issue 评论中确认方案后使用。
---

# Issue 实现

根据已确认的技术方案实现代码。

## 实现流程

### 1. 获取上下文

```bash
# 查看 Issue 及之前的分析评论
gh issue view {issue_number} --comments
```

从分析评论中提取：
- 问题类型（bug/feature）
- 技术方案
- 需要修改的文件
- 实现步骤

### 2. 创建分支

| 类型 | 分支命名 |
|------|----------|
| Bug | `fix/issue-{number}` |
| Feature | `feat/issue-{number}` |

```bash
git checkout -b fix/issue-123
# 或
git checkout -b feat/issue-123
```

### 3. 实现代码

按照分析方案中的步骤逐一实现：

1. 阅读相关文件，理解现有代码
2. 编写/修改代码
3. 确保代码风格一致
4. 添加必要的错误处理

### 4. 提交代码

使用 Conventional Commits 格式：

```bash
# Bug 修复
git commit -m "fix: 修复xxx问题 (#123)"

# 新功能
git commit -m "feat: 添加xxx功能 (#123)"

# 重构
git commit -m "refactor: 重构xxx模块 (#123)"
```

### 5. 推送分支

```bash
git push -u origin fix/issue-123
```

### 6. 创建 PR

```bash
gh pr create \
  --title "fix: 修复xxx问题 (#123)" \
  --body "## 概述
修复了 #123 中描述的问题。

## 改动
- 文件1: 改动描述
- 文件2: 改动描述

Closes #123"
```

### 7. 回复 Issue

```bash
gh issue comment 123 --body "✅ 已创建 PR #456 来解决此问题。

**改动文件：**
- \`path/to/file1.ts\`
- \`path/to/file2.ts\`

请查看 PR 进行代码审查。"
```

## 输出格式

### 结构化输出 Schema

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

## 注意事项

- 所有输出使用**简体中文**
- 确保代码**可运行**，不要留下语法错误
- 如果遇到阻塞问题，设置 `status: "blocked"` 并在 Issue 评论中说明
- PR 标题和 commit message 保持一致
- Body 中必须包含 `Closes #issue_number` 以自动关联
