+++
date = '2025-02-08T20:25:00+08:00'
draft = false
title = '再学一遍Git'
image = "navigation.png"
slug = "再学一遍Git"
description = "Git命令快查册或备忘录或半个指南"
categories = ["Code"]
tags = ["Git", "Code", "Tools", "学习", "笔记"]
+++


Git真的是非常重要了！！但我编码不多，用到Git的场景也很少，一段时间不用就会忘掉好多命令和用法QwQ。

按照一般的工作流记录一遍使用Git的工作流（顺便水一篇blog），顺便当个备忘录或教程（？），不要再忘了！


## 配置Git
---

安装Git后，首先需要设置是谁在这台设备上使用Git工作，给自己注册个身份证：
```shell
git config --global user.name "Name"
git config --global user.email "Email@email.com"
# --global 选项为用户级配置，在当前用户目录下(~)生成 .gitconfig
# --system --local 分别为机器级配置和仓库级配置
# 分别生成在	githome 和 repo 下
```

为了方便使用Github或其他Git服务器，我们需要配置SSH。为防止冒名顶替可以配置GPG：
```shell
ssh-keygen -t ed25519 -C "your_email@example.com"
# 若不支持 ed25519 则使用如下命令
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"


gpg --full-generate-key
# 若版本过旧使用如下命令
gpg --default-new-key-algo rsa4096 --gen-key

# 查看现有的GPG密钥
gpg --list-secret-keys --keyid-format=long

# 在Git中设置自己的私钥ID
git config --global user.signingkey SEC_key_id
git config --global commit.gpgsign true

# 导出公钥
gpg --armor --export SEC_key_id
```


一些方便的命令 ^w^ ：
```shell
# 更改git打开的文本编辑器
git config -- system core.editor vim

# 长命令简写
git config -- global alias.diy_name_here command_here

# 一条输出好看commit log的简写命令
git config -- global alias.loog log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit

# 手动编辑config文件
git config -- global --edit
```

## 本地操作

要管理我们的项目，首先得创建一个Git库：
```shell
git init my_proj
# or
mkdir my_proj
cd my_proj
git init
```

从头开始开发，先编写一些项目的配置文件，然后提交，作为最初的版本：
```shell
# 添加所有文件到Stage
git add .

# 将内容从Stage提交
# -m "message" 选项附带简短的提交消息
git commit

# 将当前Stage的内容合并到最近一次的提交，同时修改提交的message
git commit -m "message" --amend
```

之后可以新建一个分支，在该分支上继续开发任务：
```shell
# 查看当前仓库分支
git branch

# 新建分支
git branch dev
# 切换到新分支（两条效果相同）
git switch dev
git checkout dev

# 可以用以下表示新建并切换到新分区
git switch -c dev
git checkout -b dev

# 修改分支名
git branch -M old_name new_name
# 或修改当前分支
git branch -M new_name
```

现在看看在开发时怎么进行版本管理（）：
```shell
# 查看commit记录
# --pretty=oneline  选项简化输出每行显示一条commit
# --oneline         选项简写commit id
# -- file_here      选项显该文件相关的commit
# --graph           选项图形化显示分支信息
git log

# 查看当前仓库的状态
git status
# 查看所有操作日志
git reflog

# 若有文件被修改，查看工作区和Stage的差异
git diff file_name_here
# 查看工作区或Stage与当前HEAD指向版本的差异
git diff HEAD -- file
git diff HEAD --cached

# revert 用于撤销指定commit的更改，并为此次撤销创建commit应用到当前分支
git revert commit_id_here

# reset 用于回退到之前的版本
# HEAD^n  表示上n次提交
# --hard 已提交状态（删除工作区修改）  --soft 未提交状态  --mixed 已添加未提交状态（）
git reset --hard HEAD^
git reset --hard commit_id_here

# 撤销*Stage*中已add的文件，保留工作区中的修改（不改变工作区）
git restore --staged file
# 与上条效果相同
git reset HEAD file

# 撤销*Stage*中已add的文件，并撤销工作区中的修改使其与HEAD（若Stage中存在，则是Stage）一致
git restore file
# 与上条命令效果相同
git checkout -- file

# 删除文件
rm file
git add.
# 与上条效果相同
git rm file


# 用于选取一次commit合并到当前分支
git cherry-pick commit_id


# tag 用于为特定commit添加标签，可用于发布版本
# 当前命令为HEAD指向的commit添加标签
git tag tag_here
# 为指定commit添加标签
git tag tag_here commit_id_here
# -a 指定标签名 -m 指定标签说明
git tag -a tag_here -m "message" commit_id_here

# 撤销打上的标签
git tag -d tag_here


# 查看所有标签
git tag
# 查看特定标签信息
git show tag_here


```

在该分支上完成开发任务后，可以合并到master分支：
```shell
git switch master

# 合并分支
# 默认 Fast-forward模式 合并
git merge dev
# 合并后可删除分支
git branch -d dev
# 如果分支在合并时太乱，可以Rebase成一条线
git rebase

# 显示图形化的分支信息
git log --graph
```

## 协作开发

在本地使用Git进行版本管理进行得差不多了，现在看看如何使用Git借助远程仓库与别人协助工作。

首先看看怎么进行远程仓库的一些相关操作：
```shell
# 查看当前仓库的远程仓库信息
git remote -v
# 为本地仓库添加远程仓库
git remote add remote_name remote_repo_uri

# 推送选定分支到远程仓库
# --tags 选项推送本地所有tag
git push remote_name branch_name
# 将远程仓库作为选定本地分支的上游，简化push
git push -u remote_name branch_name

# 从远程仓库拉取 会merge到工作区
git pull remote_name
# 从远程仓库拉取 只拉取仓库，不改变工作区，需要手动merge
git ferch remote_name

# 删除当前仓库和远程仓库的绑定信息
git remote rm remote_name

# 从已有的远程仓库开始开发
git clone remote_name
```

多分支下与远程仓库的操作：
```shell
# 查看远程仓库的分支
# -a 选项查看本地和远程的所有分支
git branch -r


# 创建分支
# 将本地分支推送到远程。远程若没有则会创建
# -u 选项将绑定upstream，简化push
git push remote_name branch_name
# 一次推送本地所有分支
git push --all remote_name

# 在本地创建分支，并与已存在的远程仓库关联
git checkout -b branch_name remote_name/remote_branch

# 删除远程分支
git push remote_name --delete branch_name
# 删除本地分支
git branch -d branch_name
```

## Patch和Submodule

Patch用于记录代码的更改及应用，可以使用它生成最近几次的commit记录的patch，也可以很方便地把patch的改动合并：
```shell
# 生成patch
# 通过最近的n次commit生成n个patch
git format-patch -n

# 应用patch，但不会生成新的commit
git apply patch_name.patch
# 应用patch，但会生成新的commit
git am patch_name.patch

# 可使用send-email发送patch
git send-email --to="email" patch_name.patch
```

Submodule用于管理大型项目，尤其是一个项目是由多个子项目构成时。例如Rust开发中使用 workspace 与 submodule 配合进行项目管理：
```shell
# 添加submodule
# 添加本地的git仓库
git submodule add ./submodule_name
# 添加远程仓库
git submodule add url submodule_path

# 克隆带有 submodule 的仓库
git clone --recurse-submodules repo_url
# 或克隆仓库后再获取submodule
git submodule update --init --recursive

# 为子模块commit后，包含它作为submodule的仓库也需要提交commit

# submodule的远程仓库更新后，拉取最新代码
git submodule update --remote
# 若只需要更新某个子模块
cd module_name
git pull
cd ..
git add . && git commit -m "message"

# 删除submodule
git submodule deinit -f module_name
git rm -f module_name
rm -rf .git/modules/module_name
git commit -m "message"
```
