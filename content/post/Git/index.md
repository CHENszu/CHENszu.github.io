---
title: "Git 常用命令整理"
date: 2026-03-08
draft: false
tags: ["Git"]
categories: ["工作必备"]
---

本文整理了一些常用的 Git 命令，看完你将学会版本控制、仓库管理以及协作开发。

## 1. 下载 Git

Git下载链接：

[https://git-scm.com/download/win](https://git-scm.com/download/win)

## 2. 基础配置

首先需要在你的项目路径下初始化 Git 仓库

```bash
cd 你的项目目录
git init
```

在项目根目录下创建一个名为 .gitignore 的文件，用于指定哪些文件或目录在上传的时候应该被 Git 忽略，从而保护隐私和降低文件大小。

```bash
# 忽略所有 .env 文件
*.env
# 编译文件
__pycache__/
# 日志
*.log
```

首次安装后，需要配置你的 github 用户名和邮箱。

```bash
git config --global user.name "YourGitHubUsername"
git config --global user.email "your_email@example.com"
```

上述方式是全局配置，也可以在每个项目路径中单独配置（项目优先级高于全局配置）。

```bash
git config user.name "YourGitHubUsername"
git config user.email "your_email@example.com"
```

## 3. 版本控制

Git 中有三个区域：工作区（Working Directory）【git init】、暂存区（Staging Area）【git add】和版本库（Repository）【git commit】。  
“版本管理”简单来说就是即使我们把当前代码搞砸了，我们也能够恢复到之前的任何状态。

### 核心代码

```bash
# 添加所有文件到暂存区（你也可以指定文件）
git add .
# 提交更改
git commit -m '记录你更新了什么'
```

### 查看日志与回滚

```bash
# 查看简洁的提交日志
git log --oneline
# 回滚到指定版本 (xxx 为 commit hash)
git reset --hard xxx
```

git reset有三个参数，分别是--soft、--mixed、--hard，默认是--mixed。简单来说如果你想要回退工作区的代码，则使用--hard参数。

| 参数 | 版本库 (HEAD) | 暂存区 | 工作区 | 典型用途 |
| :--- | :--- | :--- | :--- | :--- |
| `--soft` | 回滚 | 不变 | 不变 | 撤销提交，保留修改（可重新提交） |
| `--mixed` | 回滚 | 回滚 | 不变 | 撤销提交 + 撤销暂存（默认） |
| `--hard` | 回滚 | 回滚 | 回滚 | 彻底撤销所有修改（不可逆） |

请注意：

+ reset回退是针对于commit命令，所以我们在项目中尽量优化一点就commit一次。  

+ git reset对于未追踪（U）的文件是无效的。

## 4. 管理自己的 GitHub 仓库

### 关联远程仓库

我们需要关联自己的 GitHub 仓库，才能将本地的代码推送到远程仓库。一般默认我们是将本地仓库关联名设置为 origin，当然你也可以自定义关联名称。

```bash
# 添加远程仓库
git remote add origin https://github.com/你的用户名/仓库名.git
```

```bash
# 查看远程仓库关联
git remote -v
```

```bash
# 若需修改远程仓库地址
git remote set-url origin https://github.com/新用户名/新仓库名.git
```

### 克隆与推送

```bash
# 克隆仓库
git clone https://github.com/用户名/仓库名.git

# 推送到远程仓库 (首次推送加上 - u)
git push -u origin main
```

### 下载与更新

```bash
# 拉取远程仓库最新代码
git pull
```

git pull 是把远程仓库（GitHub/GitLab）上最新的代码，拉取并合并到你本地当前的工作分支中，它本质上是 git fetch + git merge 两个命令的组合操作。

如果你的本地分支关联了多个远程仓库，或想拉取指定分支的代码：

```bash
# 格式：git pull <远程仓库名> <远程分支名>
git pull origin feature/xxx

# 如果你是直接拉取别人的文件，直接克隆即可
git clone https://github.com/用户名/仓库名.git
```

## 5. 协作开发

### 创建与切换分支

在公司里我们会多人开发一个项目，并且为了避免生产环境以及开发者的冲突，我们会为自己开发的功能创建一个新的分支。

在开发时创建一个新的分支 `feature/xxx` 来进行开发，开发完成后再合并到主分支 `main`。

```bash
# 查看所有分支
git branch

# 重命名当前分支为 main (如果你的主分支是master，不过目前基本默认为main)
git branch -M main

# 创建并切换到 feature/xxx 分支
git checkout -b feature/xxx

# 开发完成后经过add和commit命令后，就可以将本地的 feature/xxx 分支推送到远程仓库 origin 的 main 分支，等待PR/MR
git push -u origin feature/xxx

# 删除远程仓库关联 (如果需要更换远程仓库)
git remote rm origin
```

下面举一个实际的例子，假设我们有一个项目，项目的主分支是 `main`，我们现在要创建一个新的分支 `feature/xxx` 来开发一个新的功能，你可以检查下列代码中demo.txt的变化。

```bash
# 创建并切换到 feature/xxx 分支
git checkout -b feature/xxx

# 创建一个测试文件
echo "Initial Content" > demo.txt

# 提交更改
git add demo.txt
git commit -m "Add demo.txt"

# 切换回主分支
git checkout main

# 查看当前目录文件，demo.txt 这个时候是不存在的
ls

# 切换回 feature/xxx 分支
git checkout feature/xxx

# 再次查看文件
ls
```

### checkout功能详解

checkout 命令用于切换分支（分支上面讲了）或恢复工作目录。

#### 撤销文件修改

```bash
# 模拟修改文件
echo "Some bad changes" >> demo.txt
cat demo.txt

# 恢复文件到最近一次提交的状态
git checkout -- demo.txt

# 恢复当前目录所有文件
git checkout .
```

#### 新版命令推荐 (Git 2.23+)

为了职责更清晰，Git 2.23 版本引入了 `switch` (切换) 和 `restore` (恢复) 两个新命令来替代 `checkout` 的部分功能。

| 场景 | 旧命令 (checkout) | 新命令 (switch/restore) |
| :--- | :--- | :--- |
| 切换分支 | git checkout branch | git switch branch |
| 创建并切换分支 | git checkout -b branch | git switch -c branch |
| 恢复文件 | git checkout -- file | git restore file |

#### 版本库级 vs 文件级

git reset 和 git restore 都是用于撤销更改的命令，但它们的使用场景和效果不同：

+ git reset 是版本库级别的操作，它会影响整个版本库的历史记录。  
+ 想撤销「文件的修改（工作区 / 暂存区）」→ 用 git restore；
+ git restore 是文件级别的操作，它只影响指定的文件。  
+ 想回滚「分支版本、撤销 commit」→ 用 git reset。
