---
title: git_操作
date: 2020-07-23 08:47:58
categories: 网站部署
tags: 
- git 
- gitbub
---

# git和github

## （一）Git基础

### Git基本工作流程

> {% asset_img git_操作流程.jpg git操作流程 %}
>
> 主要涉及到四个关键点：
>
> 1. 工作区：本地电脑存放项目文件的地方，比如my_work文件夹；
> 2. 暂存区（Index/Stage）：在使用git管理项目文件的时候，其本地的项目文件会多出一个.git的文件夹，将这个.git文件夹称之为版本库。其中.git文件夹中包含了两个部分，一个是暂存区（Index或者Stage）,顾名思义就是暂时存放文件的地方，通常使用add命令将工作区的文件添加到暂存区里；
> 3. 本地仓库：.git文件夹里还包括git自动创建的master分支，并且将HEAD指针指向master分支。使用commit命令可以将暂存区中的文件添加到本地仓库中；
> 4. 远程仓库：不是在本地仓库中，项目代码在远程git服务器上，比如项目放在github上，就是一个远程仓库，通常使用clone命令将远程仓库拷贝到本地仓库中，开发后推送到远程仓库中即可；

<!--more-->

- 关键在于**几个核心存储区的交互命令**

| 工作目录        | 暂存区               | git 仓库             | 远程仓库     |
| --------------- | -------------------- | -------------------- | ------------ |
| 被Git管理的项目 | 临时存放被修改的文件 | 目录用于存放提交记录 | 远程代码仓库 |
| `git init`      | `git add`            | `git commit`         | `git push`   |

### Git使用前的配置命令

在使用前告诉git你是谁：

1. > 第一次使用git，配置用户信息

   1. 配置用户名：`git config --global user.name "your name"`;
   2. 配置用户邮箱：`git config --global user.email "youremail@github.com"`;

2. > 查询配置信息

   1. 列出当前配置：`git config --list`;
   2. 列出repository配置：`git config --local --list`;
   3. 列出全局配置：`git config --global --list`;
   4. 列出系统配置：`git config --system --list`;


### 工作区上的操作命令

#### 提交步骤

1. `` git init `` 初始化git仓库

   > > 新建仓库
   >
   > 1. 将工作区中的项目文件使用git进行管理，即创建一个新的本地仓库：`git init`；
   >
   >    - 会生成一个.git隐藏文件
   >
   > 2. 从远程git仓库复制项目：`git clone ssh地址 ` 
   >
   >    eg：`git clone git@github.com:xiaoming403/my_work.git` 
   >    
   > 3. 关联远程仓库：
   >
   >    `git remote add origin SSH`   #本地文件远程关联github仓库

2. `` git status `` 查看文件状态

   > > 查新信息
   >
   > 1. 查询当前工作区所有文件的状态：`git status`;
   > 2. 比较工作区中当前文件和暂存区之间的差异，也就是修改之后还没有暂存的内容：git diff；指定文件在工作区和暂存区上差异比较：`git diff `;

3. ` git add 文件/文件列表 ` 提交到暂存区

   > > 提交
   >
   > 1. 提交工作区所有文件到暂存区：`git add .`
   > 2. 提交工作区中指定文件到暂存区：`git add   ...`;
   > 3. 提交工作区中某个文件夹中所有文件到暂存区：`git add [dir]`;

4. `` git commit -m 提交信息 `` 向仓库提交代码

   > > 提交文件到版本库
   >
   > 1. 将暂存区中的文件提交到本地仓库中，即打上新版本：`git commit -m "commit_info"`;
   > 2. 将所有已经使用git管理过的文件暂存后一并提交，跳过add到暂存区的过程：`git commit -a -m "commit_info"`;
   > 3. 提交文件时，发现漏掉几个文件，或者注释写错了，可以撤销上一次提交：`git commit --amend`;
   > 4. 上传：首次要加`-u` ->  `git push -u origin master `

5. `` git log `` 查看提交记录

   > > 查看信息
   >
   > 1. 比较暂存区与上一版本的差异：`git diff --cached`;
   > 2. 指定文件在暂存区和本地仓库的不同：`git diff  --cached`;
   > 3. 查看提交历史：git log；参数`-p`展开每次提交的内容差异，用`-2`显示最近的两次更新，如`git log -p -2`;

- **提示：**每天上班第一件事：git pull 拉取线上最新版本；

  ​			下班前要做的事：git push 本地代码上传线上仓库

**版本回溯**

- git reset --hard c72ac3a6e1

## （二）Git分支

### 分支

生成副本，避免影响开发主线

#### 分支细分

1. 主分支（master）：第一次向git仓库提交更新记录时自动产生的一个分支。
2. 开发分支（develop）：作为开发的分支，基于master分支创建。
3. 功能分支（feature）：作为开发具体功能的分支基于开发分支创建。

#### 分支命令 

- ` git branch `    查看分支
- ` git branch 分支名称 `   创建分支
- ` git checkout 分支名称 `   切换分支
  - 对于新分支，使用`git checkout -b 分支名`  新建一分支并切换到该分支
- ` git merge 被合并的分支名`   合并分支
- ` git branch -d 分支名称 ` 删除分支（分支合并后（退出该分支）才允许被删除）（-D 大写强制删除）
  - `git push origin :branch-name` : 远程仓库同步删除掉的分支