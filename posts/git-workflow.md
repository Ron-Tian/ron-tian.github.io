---
title: Git 工作流最佳实践
date: 2026-06-01
tags: 技术, Git
excerpt: 从 commit 规范到分支策略，打造高效的团队协作 Git 工作流。
cover: linear-gradient(135deg, #89f7fe 0%, #66a6ff 100%)
readingTime: 11
---

# Git 工作流最佳实践

Git 不只是版本控制工具，更是团队协作的语言。

## Commit 规范

### Conventional Commits

```
<type>(<scope>): <subject>

<body>

<footer>
```

**常用 type：**

| Type | 说明 |
|------|------|
| `feat` | 新功能 |
| `fix` | 修复 Bug |
| `docs` | 文档变更 |
| `style` | 代码格式（不影响功能） |
| `refactor` | 重构 |
| `test` | 测试相关 |
| `chore` | 构建/工具变更 |

**示例：**

```
feat(blog): add tag filtering feature

- Support filtering posts by tag on tag page
- Add tag cloud component
- Update navigation to include tags link
```

## 分支策略

### Git Flow（适合发布型项目）

```
main        ────●────────────●──────── (生产环境)
                \            /
develop     ─────●───●───●──●────────── (开发环境)
                  \       /
feature/x  ───────●───●─── (功能分支)
```

### GitHub Flow（适合持续部署）

```
main     ────●───────●───────●─── (主分支)
              \     / \     /
feature  ──────●─●─────●─●─── (功能分支 → PR → 合并)
```

**核心原则：**

1. `main` 分支永远是可部署状态
2. 新功能从 `main` 拉分支
3. 通过 Pull Request 合并
4. 合并前必须通过代码审查和测试

## 常用技巧

### 交互式 Rebase 整理提交

```bash
# 合并最近3个提交
git rebase -i HEAD~3
```

### Stash 暂存

```bash
# 暂存当前修改
git stash

# 恢复
git stash pop
```

### Cherry-pick 选择性合并

```bash
# 将某个 commit 应用到当前分支
git cherry-pick <commit-hash>
```

### Bisect 二分查找 Bug

```bash
git bisect start
git bisect bad          # 标记当前版本有 Bug
git bisect good <hash>  # 标记某个旧版本正常
# Git 会自动切换到中间版本，测试后标记 good/bad
git bisect reset        # 结束
```

## .gitignore 最佳实践

```
# 依赖
node_modules/

# 构建产物
dist/
build/

# 环境变量
.env
.env.local

# 编辑器
.vscode/
.idea/

# 系统文件
.DS_Store
Thumbs.db
```

---

好的 Git 习惯不是约束，而是对团队成员的尊重。清晰的提交历史，是给未来的自己和同事最好的礼物。
