---
name: git-ops
description: Git 代码提交助手。自动分析改动内容、按阿里提交规范生成 commit message、执行 add/commit/push 全流程。
---

# Git 提交助手

自动完成代码改动分析、规范化提交和推送。

## 使用方式

直接调用 `/git-ops` 或描述需求如 "提交代码"、"git commit"。

### 1. 分析改动

读取 `git status` 和 `git diff`，列出所有变更文件及修改类型（新增/修改/删除）。

### 2. 生成 Commit Message（阿里规范）

格式要求：

```
<type>(<scope>): <subject>

<body>
```

**type 枚举：**

| 类型 | 说明 |
|------|------|
| feat | 新功能 |
| fix | 修复 bug |
| docs | 文档更新 |
| style | 代码格式调整（不影响功能） |
| refactor | 重构（非 feat/fix 的代码改动） |
| test | 测试相关 |
| chore | 构建/工具/依赖调整 |

**subject 规则：**
- 中文描述，不超过 50 字
- 末尾不加句号
- 使用祈使句（"添加"、"修复"、"更新"）

**body 规则：**
- 每行以 `- ` 开头分点说明
- 说明具体改动内容，不要写空话
- 涉及文件移动/重命名需单独说明

### 3. 执行提交

```bash
git add -A
git commit -m "<生成的 message>"
```

如果存在已暂存但未提交的改动，直接基于已 stage 的内容生成 message 并提交。

### 4. 推送确认

提交成功后询问用户是否需要 push：

> 提交完成（commit: `<hash>`）。是否需要推送到远程仓库？（yes/no）

用户确认后执行：

```bash
git push
```

## 注意事项

- 优先使用已 stage 的改动；若未 stage 则自动 `git add -A`
- message 中的 scope 可选，有明确模块时填写（如 `refactor(core):`）
- 大型重构需拆分多条 commit 时提醒用户
- 提交前如检测到未跟踪的大文件或敏感信息，需警告用户
