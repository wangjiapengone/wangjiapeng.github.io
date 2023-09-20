---
title: Git基础概念和常用命令
id: git-concept-and-commands
date: 2023-07-24 10:45:35
categories:
  - 工具
tags:
  - Git

---

## 1. 概念和原理

### 1.1 本地的三种工作区域

- 工作区：项目的文件夹
- 暂存区：暂存被改动的文件,位于`.git`下的`index`文件，`git add`后文件都会被添加到暂存区内，`git commit`只提交暂存区中的文件，工作区的文件不提交
- 版本库：`.git`文件夹，保存了项目所有历史的版本记录，`git commit`后文件会被提交到版本库内


## 2. 基础命令

### 2.1 配置

- `/etc/gitconfig`: 系统配置，对应`git config --system`
- `~/.gitconfig`: 用户配置，对应`git config --global`
- `.git/config`: 本地仓库配置，对应`git config --local`
- `local`覆盖`global`覆盖`system`

```Shell
# 查看所有的配置以及它们所在的文件
git config --list --show-origin

# 设置你的用户名和邮件地址
git config --global user.name ""
git config --gloabl user.email ""
```

<!-- more -->

### 2.2 帮助

- 详细信息：`git <verb> --help`
- 简单信息：`git <verb> -h`

```Shell
git config --help
git config -h
```



### 2.3 获取Git仓库

- 已经存在的目录变成git仓库
    - `git init`: 初始化本地仓库，生成`.git`文件夹，同时会创建唯一一个分支`master`，`.git`文件夹拥有所有的版本控制信息
- 克隆现有仓库
    - 拷贝远程仓库：`git clone https://github.com/apache/spark`
- 拷贝然后指定新目录名称：`git clone https://github.com/apache/spark mySpark` 


### 2.4 记录每次更新到仓库

![img](./image/Git基础概念和常用命令/1204574-20180425163547215-1719844952.png)

- 文件状态：未跟踪（untracked），已跟踪（unmodified，modified，staged）
- `git status`: 查看工作区、暂存区文件状态
- `git add`: 跟踪文件，将工作区文件添加到暂存区`git add file.txt`。理解为：将文件添加到下一次提交中

> 对于已经在暂存区的文件，如果再对其进行修改，那么在提交的时候要注意：
>
> 1. 如果此时提交，那么提交的版本是暂存区的版本，而不是最新版
> 2. 如果执行`git add .`，那么暂存区会变成最新版，提交的就是最新版

- `git commit -m "msg"`:暂存区所有文件添加到本地仓库 
- 删除文件：取消跟踪文件，并联带着一起删除：`git rm <file>`
- 移动文件：`git mv <from> <to>`


### 2.5 查看文件的修改内容

- 工作区和暂存区的差异

    ```Shell
    git diff
    ```

- 暂存区和最后一次提交的差异

    ```Shell
    git diff --staged
    ```

### 2.6 查看提交历史

- `git log`: 每个提交的 SHA-1 校验和、作者的名字和电子邮件地址、提交时间以及提交说明
- `git log -p`: 显示每个提交的差异
- `git log -p -2`: 只显示最近两个提交
- `git log stat`: 简单的统计信息
- `git log --pretty=oneline`: 每个提交只展示一行
- `git log --graph`: 图形展示

### 2.7 覆盖上一次提交

- `git commit --amend`: 会将暂存区的文件提交，并修改commit message

### 2.8 文件从暂存区撤回工作区

- `git restore --staged <file>`

### 2.9 取消对于工作区文件的修改

- `git restore <file>`

### 2.10 远程仓库配置

- 默认的远程仓库简写名称：`origin`
- 显示读写的远程仓库：`git remote -v`
- 添加远程仓库：`git remote add <shortname> <url>`
- 拉取远程仓库（只下载，不合并）：`git fetch`
- 抓取并合并到本地分支：`git pull origin <branch>` 
- 推送到远程仓库：`git push origin <branch>`

### 2.11 打tag

- 列出所有标签：`git tag`
- 查找制定模式的标签：`git tag -l "mb2021*"`
- 附注标签的操作：
    - add tag: `git tag -a <tag> -m ""`
    - delete tag: `git tag -d <tag>`
    - add remote tag: `git push origin -f <tag>`
    - delete remote tag: `git push origin -d <tag>`
    - show tag: `git show <tag>`



## 3. 分支命令

### 3.1 新建并切换分支

- `git checkout -b new_branch`
- `HEAD`: 指向当前所在的本地分支
- `git log --oneline --decorate --graph --all`: 查看项目分叉的历史



### 3.2 分支合并

- dev分支合并到master分支

    ```Bash
    # 先checkout到master分支
    git checkout master
    # 再把dev合并进来
    git merge dev
    # 合并完成后，可以删除分支
    git branch -d dev
    ```

- 遇到合并冲突

    - 先用`git status`找到`Unmerged paths`里面的文件，手动解决冲突
    - 然后`git commit`完成提交

### 3.3 查看分支

- `git branch`: 列出所有分支
- `git branch -v`: 列出所有分支，及对应最后一次提交
- `git branch --merged`: 列出已经合并到**当前**分支的分支
- `git branch --merged master`: 列出已经合并到**master**分支的分支
- `git branch --no-merged`: 列出还没有合并到**当前**分支的分支
- `git branch --no-merged master`: 列出还没有合并到**master**分支的分支

### 3.4 删除分支

- `git branch -d dev`: 可以删除已经合并过的分支，即`git branch --merged`列出的分支
- `git branch -D dev`: 强制删除任意分支（master不能删除）

### 3.5 变基（另一种merge分支的方法）

- 作用：使提交历史更简洁

- 何时不能用：分支已经提交到远程仓库，别人可能用这个分支进行开发

    ```Bash
    # 变基
    git checkout test
    git rebase master
    # 合并
    git checkout master
    git merge test
    # 删除
    git branch -d test
    ```

## 4. 子模块

### 4.1 克隆项目的同时下载所有子模块

- 默认在克隆项目时，子模块不会自动下载

    ```bash
    git clone git@github.com:wangjiapengone/wangjiapengone.github.io.git --recurse-submodules
    ```

- 子项目下载后，可能不是最新的`commit`，需要手动更新

    ```bash
    git submodule foreach git pull origin master
    ```

- 子项目`head`默认状态为`detached`，需要手动切换为`master`分支

    ```bash
    git submodule foreach git checkout master
    ```

## 5. 工作中遇到的问题

### 5.1. 项目首次从远端拉取代码失败

- 报错：`fatal: 'origin' does not appear to be a git repository `

    ```
    fatal: Could not read from remote repository.
    ```

- 解决办法：把项目地址加入到origin配置中

    ```
    git remote add origin git@github.com:wangjiapengone/wangjiapengone.github.io.git
    ```

### 5.2. git remote add origin配置失败 / git pull 取消密码

- 报错：`fatal: remote origin already exists.`

- 解决办法：报错的原因是origin地址已经存在，需要把地址修改一下

    ```
    git remote set-url origin git@github.com:wangjiapengone/wangjiapengone.github.io.git
    ```

### 5.3. `GitLab`免密操作

- 简单做法

    ```Bash
    # 在机器上生成新的秘钥
    ssh-keygen -t rsa -C "jiapeng@ubuntu-22" -P ''
    # 把公钥信息复制到GitLab页面中
    cat ~/.ssh/id_rsa.pub
    # 注意配置用户名和邮箱
    git config --local user.name wangjiapeng
    git config --local user.email wangjiapeng@xxx.com
    # 更改远程url
    git config --local remote.origin.url git@github.com:wangjiapengone/wangjiapengone.github.io.git
    ```

- 如果是多人合作，可以生成自己单独的秘钥

    ```Bash
    # 在新文件中生成秘钥
    ssh-keygen -t rsa -C "wangjiapeng@xxx.com" -P '' -f /home/rd/.ssh/id_rsa_wjp
    # 把公钥信息复制到GitLab页面中
    cat ~/.ssh/id_rsa.pub
    # 注意配置用户名和邮箱
    git config --local user.name wangjiapeng
    git config --local user.email wangjiapeng@xxx.com
    # 更改远程url
    git config --local remote.origin.url git@github.com:wangjiapengone/wangjiapengone.github.io.git
    # 重要：git默认不会识别除了id_rsa以外的文件需要在config里面设置
    vim ~/.ssh/config
    # 添加如下配置到config文件中
    Host github.com
      PreferredAuthentications publickey
      IdentityFile ~/.ssh/id_rsa_wjp
    ```

### 5.4. 代码上传到`GitHub`

- 在`GitHub`上创建一个空的仓库
- `git init`初始化本地仓库
- 秘钥配置参考上文
- 注意，`GitHub`的主分支为`main`，本地分支需要名称一致，如果远程仓库有初始化的文件，需要先`git pull origin main --rebase --allow-unrelated-histories`
- 最后`git push --set-upstream origin main`

### 5.5 打补丁

- 希望测试开源项目中没有合到`master`的代码
- 打开`pull requests`，对应的地址为：https://github.com/apache/superset/pull/24368  在后面添加`.patch`
- 网页会跳转为：
    `https://patch-diff.githubusercontent.com/raw/apache/superset/pull/24368.patch`
- 下载页面内容：
    `wget https://patch-diff.githubusercontent.com/raw/apache/superset/pull/24368.patch`
- 应用补丁：`git apply 24368.patch`
- 最后`git status`看一下，补丁会转换为未提交的代码
