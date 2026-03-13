# Git 使用笔记

## 1. Git 简介

### 1.1 什么是 Git

Git 是分布式版本控制系统。

### 1.2 集中式 vs 分布式（SVN vs Git）

SVN 和 Git 主要的区别在于历史版本维护的位置：

| 特性 | SVN | Git |
|------|-----|-----|
| 历史库位置 | 中央仓库 | 本地仓库 |
| 离线操作 | 需连接中央仓库 | 支持离线查看历史 |
| 中央仓库故障 | 无法恢复 | 可快速重建中央库 |

**分布式优势**：
- 可以在脱机环境查看开发的版本历史
- 多人开发时如果中央仓库挂了，可以随时创建新的中央仓库并同步恢复

---

## 2. Git 配置

### 2.1 基本配置

```bash
# 设置用户名
git config --global user.name "Your Name"

# 设置邮箱
git config --global user.email "email@example.com"

# 设置大小写敏感（false 表示不忽略大小写）
git config --global core.ignorecase false
```

> `--global` 参数表示这台机器上所有 Git 仓库都会使用这个配置。也可以对某个仓库单独指定不同的用户名和邮箱。

### 2.2 SSH 配置

```bash
# 创建 SSH Key
ssh-keygen -t rsa -C "youremail@example.com" -b 4096
```

### 2.3 屏蔽主机密码验证

在 `.ssh` 目录下新建 `config` 文件，添加：

```
StrictHostKeyChecking=no
```

---

## 3. 创建版本库

### 3.1 初始化仓库

```bash
git init
```

### 3.2 添加文件到仓库

```bash
# 添加单个文件
git add <file>

# 添加所有文件
git add .

# 提交到版本库
git commit -m "description"
```

> `git add` 可以反复多次使用，添加多个文件。`git commit` 一次提交暂存区的所有文件。

### 3.3 快捷提交

```bash
# 相当于 git add . + git commit -m
git commit -am "description"
# 或
git commit -a -m "description"
```

---

## 4. 查看状态与修改

### 4.1 查看工作区状态

```bash
git status
```

### 4.2 查看修改内容

```bash
# 查看工作区和暂存区的区别
git diff

# 查看暂存区和分支的区别
git diff --cached

# 查看工作区和版本库最新版本的区别
git diff HEAD -- <file>
```

### 4.3 查看提交日志

```bash
# 查看完整日志
git log

# 简化日志输出
git log --pretty=oneline

# 查看分支合并图
git log --graph
```

### 4.4 查看命令历史

```bash
git reflog
```

---

## 5. 版本回退

### 5.1 回退到上一版本

```bash
git reset --hard HEAD^
```

| 符号 | 含义 |
|------|------|
| `HEAD` | 当前版本 |
| `HEAD^` | 上一个版本 |
| `HEAD^^` | 上上一个版本 |
| `HEAD~100` | 往上 100 个版本 |

### 5.2 回退到指定版本

```bash
git reset --hard <commit_id>
```

> `commit_id` 是版本号，由 SHA1 计算得出。

### 5.3 撤销 commit

```bash
# 撤销 1 次 commit
git reset --soft HEAD~1

# 撤销 2 次 commit
git reset --soft HEAD~2
```

---

## 6. 工作区、暂存区和版本库

### 6.1 概念说明

| 区域 | 说明 |
|------|------|
| 工作区 | 电脑里能看到的目录 |
| 版本库 | 工作区隐藏目录 `.git` |
| 暂存区 | stage/index，版本库中的重要区域 |

Git 版本库存放了：
- 暂存区（stage/index）
- 自动创建的 `master` 分支
- 指向 `master` 的指针 `HEAD`

### 6.2 命令解释

```bash
# 把文件添加到暂存区
git add <file>

# 把暂存区内容提交到当前分支
git commit -m "description"
```

---

## 7. 撤销修改

### 7.1 丢弃工作区修改

```bash
git checkout -- <file>
```

**两种情况**：
1. 文件修改后未添加到暂存区 → 回到版本库状态
2. 文件已添加到暂存区后又修改 → 回到暂存区状态

### 7.2 丢弃暂存区修改

```bash
# 第一步：把暂存区修改撤销，重新放回工作区
git reset HEAD <file>

# 第二步：撤销工作区修改
git checkout -- <file>
```

### 7.3 小结

| 场景 | 命令 |
|------|------|
| 改乱工作区，想丢弃 | `git checkout -- <file>` |
| 已添加到暂存区，想丢弃 | `git reset HEAD <file>` → `git checkout -- <file>` |
| 已提交到版本库，想撤销 | 版本回退（未推送远程库时） |

---

## 8. 删除文件

### 8.1 删除文件

```bash
# 从暂存区删除，保留工作区文件
git rm --cached -f <file>

# 从暂存区和工作区都删除
git rm <file>

# 删除文件夹
git rm -rf <dir_name>
```

> `git rm <file>` 相当于执行 `rm <file>` + `git add <file>`

### 8.2 恢复误删文件

**场景一**：执行了 `rm text.txt` 误删

```bash
git checkout -- text.txt
```

**场景二**：执行了 `git rm text.txt`

```bash
git reset HEAD text.txt
git checkout -- text.txt
```

**场景三**：真的想从版本库删除

```bash
git commit -m "delete text.txt"
```

---

## 9. 远程仓库

### 9.1 关联远程仓库

```bash
# 添加远程仓库
git remote add origin https://github.com/username/repositoryname.git

# 重设远程仓库地址
git remote set-url origin https://github.com/username/repositoryname.git

# 查看远程库信息
git remote -v
```

### 9.2 推送到远程仓库

```bash
# 第一次推送（关联分支）
git push -u origin master
# 或
git push --set-upstream origin master

# 后续推送
git push origin master
```

### 9.3 从远程克隆

```bash
git clone https://github.com/username/repositoryname.git
```

### 9.4 从远程抓取

```bash
# 抓取远程更新
git pull

# 如果推送失败，先抓取远程新提交
git pull
```

### 9.5 同步其他远程仓库

将远程 A 的 master 分支同步到本地 dev 分支：

```bash
git fetch git@github.com:username/repo.git master:dev
```

---

## 10. 分支管理

### 10.1 创建与查看分支

```bash
# 创建分支
git branch <branchname>

# 查看分支
git branch

# 创建并切换分支
git checkout -b <branchname>

# 切换分支
git checkout <branchname>
```

### 10.2 合并分支

```bash
# 合并某分支到当前分支
git merge <branchname>

# 普通模式合并（保留合并记录）
git merge --no-ff -m "description" <branchname>
```

### 10.3 删除分支

```bash
# 删除已合并的分支
git branch -d <branchname>

# 强制删除未合并的分支
git branch -D <branchname>

# 删除远程分支
git push origin --delete <branch-name>

# 清除已删除的远程分支引用
git fetch --prune
```

### 10.4 修改远程分支名称

```bash
git branch -m <branch_old> <branch_new>
git push --delete origin <branch_old>
git push --set-upstream origin <branch_new>
```

### 10.5 本地与远程分支关联

```bash
# 创建本地分支并与远程分支对应
git checkout -b <branch-name> origin/<branch-name>

# 建立本地分支和远程分支的关联
git branch --set-upstream <branch-name> origin/<branch-name>
```

### 10.6 推送分支

```bash
# 从本地推送分支
git push origin <branch-name>
```

---

## 11. Stash（保存工作现场）

```bash
# 保存工作现场
git stash

# 查看工作现场
git stash list

# 恢复工作现场
git stash pop
```

---

## 12. 标签管理

### 12.1 创建标签

```bash
# 新建标签（默认为 HEAD）
git tag <tagname>

# 指定 commit id 创建标签
git tag <tagname> <commit_id>

# 指定标签信息
git tag -a <tagname> -m "description" <branchname>

# PGP 签名标签
git tag -s <tagname> -m "description" <branchname>
```

### 12.2 查看标签

```bash
# 查看所有标签
git tag
```

### 12.3 推送标签

```bash
# 推送单个标签
git push origin <tagname>

# 推送全部未推送过的标签
git push origin --tags
```

### 12.4 删除标签

```bash
# 删除本地标签
git tag -d <tagname>

# 删除远程标签
git push origin :refs/tags/<tagname>
```

---

## 13. 常见问题

### 13.1 升级 Git 版本

```bash
git update-git-for-windows
```

### 13.2 Android Studio 终端 git log 乱码

**原因**：Android Studio 使用 `en_UTF8` 编码，Windows 使用 `C_UTF8` 编码

**解决方案**：新建系统环境变量

| Key | Value |
|-----|-------|
| `LC_ALL` | `C.UTF-8` |

### 13.3 Git 进程锁定问题

**错误信息**：
```
Another git process seems to be running in this repository...
```

**原因**：Git 操作时崩溃，部分被上锁资源未释放

**解决方案**：
1. 进入项目文件夹下的 `.git` 目录
2. 删除 `index.lock` 文件

```bash
rm .git/index.lock
```

### 13.4 优化 .git 文件夹大小

参考文档：https://www.cnblogs.com/lout/p/6111739.html

---

## 14. 命令速查表

| 功能 | 命令 |
|------|------|
| 初始化仓库 | `git init` |
| 克隆仓库 | `git clone <url>` |
| 添加文件 | `git add <file>` |
| 提交 | `git commit -m "msg"` |
| 查看状态 | `git status` |
| 查看日志 | `git log` |
| 版本回退 | `git reset --hard HEAD^` |
| 创建分支 | `git branch <name>` |
| 切换分支 | `git checkout <name>` |
| 合并分支 | `git merge <name>` |
| 删除分支 | `git branch -d <name>` |
| 推送远程 | `git push origin <branch>` |
| 拉取远程 | `git pull` |
