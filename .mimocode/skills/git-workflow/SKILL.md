---
name: git-workflow
description: 解释和执行Git命令，包括添加、提交、推送、分支管理等操作
---

# Git工作流技能

## 目的
解释Git命令并执行Git操作，帮助用户理解和使用Git进行版本控制。

## 适用场景
- 用户询问Git命令和操作
- 需要将更改推送到GitHub
- 需要管理Git分支
- 需要解决Git冲突
- 需要查看Git状态和历史

## 工作流程

### 1. 评估用户需求
- 确定用户想要执行的Git操作
- 评估用户的Git知识水平
- 确定是否需要解释命令

### 2. 执行Git操作
- 使用适当的Git命令
- 处理可能的错误
- 提供操作结果反馈

### 3. 解释命令（如需要）
- 解释每个命令的作用
- 解释命令参数的含义
- 提供最佳实践建议

## 常用Git命令模板

### 基本操作
```bash
# 查看状态
git status

# 添加文件
git add <file>

# 提交更改
git commit -m "提交信息"

# 推送到远程
git push origin <branch>
```

### 分支操作
```bash
# 创建分支
git branch <branch-name>

# 切换分支
git checkout <branch-name>

# 合并分支
git merge <branch-name>

# 删除分支
git branch -d <branch-name>
```

### 远程操作
```bash
# 查看远程仓库
git remote -v

# 拉取最新代码
git pull origin <branch>

# 推送代码
git push origin <branch>
```

## 注意事项
- 始终先解释命令再执行（如用户需要）
- 处理中文路径时使用引号
- 注意分支名称一致性（main vs master）
- 提交信息要清晰描述更改内容

## 示例交互

**用户**: 我想把修改推送到GitHub
**助手**: 我来帮你推送到GitHub。首先检查状态：
```bash
git status
```
然后添加文件、提交、推送：
```bash
git add .
git commit -m "更新游戏功能"
git push origin main
```

**用户**: 这个命令是什么意思？
**助手**: `git add .` 会将当前目录下所有修改过的文件添加到暂存区，准备提交。`.` 表示当前目录。