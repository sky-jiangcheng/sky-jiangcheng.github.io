---
title: Git 团队协作工作流最佳实践
date: 2026-05-20 10:00:00
categories:
  - tutorial
tags:
  - Git
  - 团队协作
  - DevOps
description: 分享在团队开发中如何规范使用 Git，包括分支策略、Commit Message 规范、Code Review 流程和常见问题的处理方案。
---

## 前言

在多年的后端开发经历中，团队协作的规范化程度直接影响项目的交付质量。本文总结了在实际项目中经过验证的 Git 工作流实践。

## 1. 分支策略

推荐基于 GitFlow 简化版的分支模型：

```
master         o--o----------------o--------- (生产)
                \                 /
develop          o--o--o--o--o--o---------- (开发主线)
                     \    \    /
feature/login        o--o--o
feature/payment           o--o--o
release/v1.0                       o--o
```

分支命名规范：

| 分支类型       | 说明                         |
| -------------- | ---------------------------- |
| master         | 生产环境分支                 |
| develop        | 开发主线                     |
| feature/xxx    | 功能开发分支，从 develop 切出|
| hotfix/xxx     | 紧急修复分支，从 master 切出 |
| release/x.x.x  | 发布准备分支，从 develop 切出|

## 2. Commit Message 规范

采用 Conventional Commits 格式：

```
<type>(<scope>): <subject>

<body>

<footer>
```

常用的 type 类型：

| type     | 说明                           |
| -------- | ------------------------------ |
| feat     | 新功能                         |
| fix      | Bug 修复                       |
| docs     | 文档更新                       |
| style    | 代码格式(不影响功能)           |
| refactor | 重构                           |
| perf     | 性能优化                       |
| test     | 测试相关                       |
| chore    | 构建过程或辅助工具的变动       |

示例：

```bash
git commit -m "feat(user): 添加用户登录接口

实现基于 JWT 的用户登录功能，包含:
- 用户名密码校验
- Token 生成与刷新
- 登录失败次数限制

Closes #123"
```

## 3. Code Review 流程

1. 开发者在 feature 分支完成开发后，提交 PR 到 develop
2. 至少一位团队成员进行 Review
3. Review 通过后合并，删除 feature 分支

Review Checklist：

- 代码逻辑是否正确
- 是否有单元测试
- 是否遵循项目编码规范
- 是否有潜在的性能问题
- 错误处理是否完善

## 4. 合并策略

推荐使用 `git merge --no-ff` 保留分支历史：

```bash
git checkout develop
git merge --no-ff feature/user-login
```

## 5. 常见问题处理

### 合并冲突

```bash
# 从 develop 拉取最新代码
git checkout develop && git pull origin develop

# 在 feature 分支 rebase
git checkout feature/xxx && git rebase develop

# 解决冲突后继续
git add . && git rebase --continue
```

### 回滚已推送的提交

```bash
# 使用 revert 创建反向提交(安全，推荐)
git revert HEAD

# 使用 reset 强制回滚(危险，需确认无人基于此提交)
git reset --hard HEAD~1
git push --force-with-lease
```

## 总结

规范的 Git 工作流不是负担，而是团队协作的基础设施。清晰的分支策略降低合并冲突，规范的 Commit Message 提升可追溯性，严格的 Code Review 保证代码质量。建议将这些规范写入团队的 CONTRIBUTING.md 文档中。
