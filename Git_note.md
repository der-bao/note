# 前提工作

## 0. 参考

参考视频：[一小时Git教程](https://www.bilibili.com/video/BV1HM411377j?spm_id_from=333.788.videopod.sections&vd_source=26b39f0d20bfb02f04f447a625efe7be)

## 1. 安装

Ubuntu安装

```
# 安装 Git
sudo apt install git -y
# 配置身份信息
git config --global user.name "你的名字"
git config --global user.email "你的邮箱@example.com"
# 查看配置信息
git config --global --list
```

# 一、本地仓库

## 1. Git环境搭建

### （2）用户配置

```
git config --global user.name "你的名字"
git config --global user.email "你的邮箱@example.com"
# 查看配置信息
git config --global --list
```

参数说明：

- `--global`:全局配置，所有仓库生效
- `--system`：系统配置，对所有用户生效
- `(省略)` :本地配置，只对本地仓库生效

## 2. Git核心概念

### （1）工作区域

- 工作区（Working Directory）
- 暂存区（Staging Area）
- 本地仓库（Local Repository）

### （2）文件状态

- 未跟踪（Untrack）
- 未修改（Unmodified）
- 已修改（Modified）
- 已暂存（Staged）

### （3）指针

- HEAD指针：指向你当前工作区正在使用的分支 / 提交，全局一个
- 分支指针：指向该分支的最新提交（分支的 “终点”）
- 一般情况下，HEAD指针依附在分支指针上。因此当执行 reset 回退时，分支指针回到之前的某个提交节点，HEAD指针也会回到对应指针。
- 但是当执行查看操作（checkout）时，只是将HEAD指针移到对应节点（工作区文件会发生对应变化）。

## 3.核心操作

### （1）初始化仓库

```
git init
```

### （2）暂存

将文件从工作区 => 暂存区

```
git add filename	// 单个文件
git add *.txt		// 某个类型的文件，如txt文件
git add . 			// 某个指定的文件夹，如"."代表当前的工作目录
```

### （3）提交

将文件从暂存区 => 仓库（使用如下命令时，只会将暂存区的文件打包提交，不会提交工作区的文件	）

```
git commit -m "note" 	// note:本次提交的笔记
# 如果没有输入-m 参数，会进入Vim编辑器进行交互
# "i"键进入编辑，输入本次提交的笔记
# "Esc"键回到指令模式
# 输入":wq"保存退出

git log					// 可以查看commit记录，按下q退出分页器
git log -n 5 --oneline 	// -n 显示提交记录条数，--online：保留提交id前7位

git commit -a -m "note" 	// 对于文件状态不是新创建的情况，可以同时完成暂存和提交两个操作。
git commit --amend -m "新的提交备注信息"		// 修改最新一次提交的备注
```

### （4）gitignore忽略文件

对于一些系统生成的环境或日志文件之类，我们可以忽略更改（不出现在工作区）。
在".gitignore"文件中添加如下字段

```
# 注释
*.log		#  忽略所有日志文件
temp/ 		# 忽略指定文件夹，记得以"/"结尾
doc/**/*.txt	# 忽略doc文件下及所有子目录下的txt文件
# 采用Blob模式进行匹配，具体需求可以ai
# 匹配规则从上往下匹配
```

**注意**：
对于已经被添加到版本库的文件，".gitignore"文件并不会生效。
如果不小心提交了不需要的文件可以通过以下指令将其从版本库中移除

```
git rm --cached filename 	// 从 Git 的暂存区（索引）中移除文件，但保留本地工作区的文件
git commit -m "note"		// 移除以后还需要再提交一次，才可以彻底地停止跟踪
```

### （5）查看某版本代码

如果想查看某个版本的代码，可以采用如下指令查看：

```
git checkout <commit_id>				// 此时工作区代码就会变成本次提交的状态
git branch branch_name					// 查看分支后，可以切换分支，就会变成对应分支最新提交的状态

# 如果此时想要更改代码(如想采用一种新的方法实验)，可以在该节点创建一个新分支， 用来保存修改
git checkout -b <branch_name> 			// 创建新分支并切换
git commit -m "note" 					// 然后再提交
```

### （6）版本回退

当想要回退到之前某次提交的版本，可以采用如下指令：

```
git reset --mixed commit_id			//一般情况下，默认参数是--mixed
git reset --mixed HEAD^ 			// HEAD^ :回到上一次提交
git reset HEAD~n 					// HEAD-n:最新提交的前n个版本
```

 参数说明：

| 参数    | 工作区文件 | 暂存区文件 |
| ------- | ---------- | ---------- |
| --hard  | 不保存     | 不保存     |
| --mixed | 保存       | 不保存     |
| --soft  | 保存       | 保存       |

`--mixed`和 `--soft`一般用于几次提交没有多大的更新，可以回退后再打包提交，区别在于采用 `--mixed`还要将工作区的文件提交暂存区。
`--hard`:用于舍弃之前的提交。误操作可以采用 `git reset --hard commit_id`进行回退。

**注意**:版本回退会将分支指针和头指针（HEAD）移动到指定分支。此时如果新提交，以往的旧提交会丢失，可以通过上述方法回退，或者创建一个新的分支指针指向旧提交 `git checkout -b <branch_name> <commit_id> `将以往的提交绑定。

# 二、远程仓库

## 1.基础概念

## 2. 远程仓库连接方式

- HTTPS方式：无需配置，每次操作输令牌（适合新手临时使用）
- SSH方式：一次配置，终身免密（重点讲生成密钥、上传公钥步骤，解决“SSH连接失败”痛点）

```
git remote -v				// 查看本地仓库关联的远程仓库信息
```

SSH配置
链接：[SSH配置](https://www.bilibili.com/video/BV1HM411377j?spm_id_from=333.788.videopod.sections&vd_source=26b39f0d20bfb02f04f447a625efe7be&p=11)

```
# 1. 在"C:/用户/用户名下"，输入cmd 打开命令提示符，输入如下指令：
ssh-keygen -t rsa -b 4096
# 输入后会在目录下多了一个".ssh"文件夹，文件夹下有"id_rsa"和"id_rsa.pub"文件
# 将"id_rsa.pub"文件复制到GitHub中
```

## 3.本地与远程仓库同步

### （1）clone仓库

```
git clone  URL(远程仓库的地址)
```

### （2） 推送（push）

```
  # 首次推送
git remote add origin URL					// 添加一个远程仓库，“origin”是远程仓库的别名
git push -u origin branch_name				// "-u":upstream，建立分支关联；把本地对应分支推送到 origin 远程
  													仓库对应的分支（不存在时自动创建）
```

### （3）拉取（pull）

`git fetch`是一个从远程仓库下载最新数据到本地仓库的命令，但它不会自动合并到你的当前工作分支。

```
git fetch origin branch_name		// 获取远程仓库的特定分支
```

而 `git pull`则是 `git fetch`和 `git merge` 的合并，会下载数据并尝试合并到当前分支，可能产生冲突

```
  git pull origin main 
  git pull --rebase origin main		//
```

### （4）版本回退

对于远程仓库的版本回退采用 `git revert`,本质是创建一个新的「反向提交」，抵消指定提交的所有修改，而非删除原有提交。

```
# 0. 暂存未提交的修改（如果有）
git stash

# 1. 确保本地代码和远程同步（先拉取最新代码）
git pull origin main  # main替换为你的分支名（如dev）

# 2. 生成反向提交
# HEAD 代表最后一次提交，HEAD~1 代表倒数第二次，以此类推
# 或者是commit_id
# 如果是连续的，还可以使用“<id1>^..<id2>”,左开右闭
git revert HEAD

# 3. 此时Git会自动打开编辑器，让你填写反向提交的备注，保存并退出即可
#    如果想跳过编辑器，直接用默认备注：
# git revert HEAD --no-edit

# 4. 将反向提交推送到远程，完成远程分支的“回退”
git push origin main
```

## 4. *分支

在Git中，分支是并行开发的核心。每个分支代表一个独立的开发线，允许您在不影响主分支的情况下进行工作，然后可以在适当的时候将更改合并回主分支或其他分支。

分支一般可以分为以下类型：

- 主分支（master/main）：通常用于生产环境的**稳定代码**。
- 开发分支（develop）：用于集成各个功能分支，进行**整体测试**。
- 功能分支（feature branches）：开发新功能时从develop分支创建，完成后合并回develop。
- 发布分支（release branches）：准备发布新版本时创建，用于修复bug和版本准备。
- 热修复分支（hotfix branches）：生产环境出现紧急bug时，从master分支创建，修复后合并回master和develop。

### （1）查看分支

```
# 查看本地分支
git branch

# 查看远程分支
git branch -r

# 查看所有分支（包括远程）
git branch -a

# 查看分支及其最后提交
git branch -v

# 查看已合并到当前分支的分支
git branch --merged

# 查看未合并到当前分支的分支
git branch --no-merged
```

### （2）创建分支

```
# 创建新分支（但不切换）
git branch <branch-name>

# 创建并切换到新分支
git checkout -b <branch-name>

# 从特定提交创建分支
git branch <branch-name> <commit-hash>
```

### （3）切换分支

```
# 切换到已有分支
git checkout <branch-name>

# 切换到上一个分支
git checkout -

# 使用 switch 命令（Git 2.23+）
git switch <branch-name>
git switch -c <new-branch>  # 创建并切换
```

### （4）删除分支

```
# 删除已合并的分支
git branch -d <branch-name>

# 强制删除未合并的分支
git branch -D <branch-name>

# 删除远程分支
git push origin --delete <branch-name>
# 或
git push origin :<branch-name>
```

### （5）重命名分支

```
# 重命名当前分支
git branch -m <new-name>

# 重命名特定分支
git branch -m <old-name> <new-name>

# 如果已推送到远程，需要：
git branch -m <new-name>
git push origin -u <new-name>
git push origin --delete <old-name>
```

### （6）分支合并

```
# 类型
git merge dev -m "messge"					// 将dev分支合并到当前的工作分支
git rebase main						 		// 将当前工作分支从共同提交节点以后的所有提交记录剪切移动到指定分支的顶端

# 当分支合并后，副分支并不会消失
git branch -D branch_name					// 彻底取消某个分支

# 分支合并冲突（不同分支修改了同一个文件的同一部分内容）
# 1. 继续合并
（1）手动解决冲突（必须删除 <<<<<<<、=======、>>>>>>> 这些标记，记得修改后保存！！）
（2）将解决冲突后的文件加入暂存区
			git add test.txt
（3）完成合并提交（Git 会自动生成合并提交的备注，也可自定义）
			git commit -m "note"
		
# 2. 不继续合并
			git merge --abort
```

# 三、协同开发

git工作流模型（多人协同开发规范）

## 1. 主流分支模型

```
# Git Flow（经典模型）
main (master)     - 稳定生产版本
develop           - 集成开发分支
feature/*         - 功能分支
release/*         - 发布分支
hotfix/*          - 热修复分支

# GitHub Flow（简化模型）
main              - 主分支（可部署）
feature/*         - 功能分支

# GitLab Flow（环境分支）
main
production
staging
feature/*
```

## 2. 分支命名规范

```
# 功能分支
feature/login-authentication
feature/add-user-profile

# 修复分支
bugfix/fix-login-error
hotfix/security-patch

# 发布分支
release/v1.2.0

# 重构分支
refactor/cleanup-codebase
```

## 3. 多人协作标准流程

`开发流程：主分支保护 → 创建功能分支 → 开发 → 推送 → PR/MR → 代码审查 → 合并 → 删除分支`

### （1）同步最新代码

```
git checkout main		# 确保在主分支

git pull --rebase origin main	# 将本地修改 “嫁接到” 远程最新代码之上
```

### （2）创建功能分支

```
git checkout -b feature/your-feature-name

# 分支命名规范：
# feature/add-user-auth    # 新功能
# bugfix/fix-login-error   # 修复bug
# hotfix/critical-security # 紧急修复
# refactor/cleanup-api     # 重构
# chore/update-deps        # 维护任务
```

### （3）开发提交

```
# 开发代码...

# 添加更改
git add .
# 或分阶段添加
git add file1.js file2.js

# 提交（遵循提交规范）
git commit -m "feat(auth): 添加用户登录功能

- 实现邮箱/密码登录
- 添加JWT令牌验证
- 完善错误处理 "
```

### （4）推送分支到远程

```
# 提交前的同步,解决冲突
git fetch origin
git rebase origin/main 

# 第一次推送
git push -u origin feature/your-feature-name

# 后续推送（如果已设置上游）
git push
```

### （5）合并并删除分支

```
git branch -d feature/your-feature-name      # 删除本地
git push origin --delete feature/your-feature-name  # 删除远程
```

# 四、其他

日常协作命令清单

```
git log --graph --online --decorate --all			# 当没有图形化界面时，可以采用指令
git status                          # 查看状态
git fetch --all                     # 获取所有远程更新
git diff origin/main..HEAD          # 比较本地与远程差异
git stash                           # 暂存当前修改
git stash pop                       # 恢复暂存
git cherry-pick <commit-hash>       # 选择性应用提交
git reflog                          # 查看操作历史
```
