title: git使用方法
tags:
  - git
categories:
  - git
description: 有关git的入门配置
author: zonzie
date: 2017-12-24 17:56:00
---
##### 准备工作

- 首先需要安装git命令行工具gitbash
- 需要注册一个github账号

##### 初始配置
- 安装完毕后,第一次使用:
    1. 设置用户名和邮箱,设置自己的用户名和邮箱
    ```shell
        git config --global user.name "zonzie"
        git config --global user.email "yaobq13@gmail.com"
    ```
    2. 生成ssh-key,之后可以不用每次提交代码时都输入账号密码
    ```
    	ssh-keygen -t rsa -C "yaobq13@gmail.com"
    ```
    3. 之后会有提示:这时可以什么都不输入,三个回车
    ```
            Enter file in which to save the key (/c/Users/zonzie/.ssh/id_rsa): 
            Enter passphrase (empty for no passphrase):
            Enter same passphrase again:
        
    ```

        - 然后会生成公钥和私钥在用户根目录下的.ssh目录下id_rsa是私钥, id_rsa.pub是公钥
    4. 使用 ssh-agent 管理公钥
       - eval "$(ssh-agent -s)"
       - ssh-add id_rsa.pub
    5. 登录github账户,将公钥添加到 settings/SSH and GPG keys 中
    6. 在gitbash中登录github
        - ssh -T git@github.com
        
##### 常用命令

- git的常用命令
    1. 先建一个目录 `mkdir myRepo`
    2. 变成仓库 `git init`    --- 显示隐藏文件  `ls -ah
    3. 新建一个文件readme.txt,要在myRepo目录下或者子目录下
    4. 添加文件到仓库  `git add readme.txt`
    5. 然后提交  `git commit -m "some message"`
    6. 查看一段时间内文件的修改内容 `git diff `
    7. 查看当前的状态,是否有未提交的内容 `git status `
    8. 提交修改的内容和添加新文件方法一致 ``git add <file>`,`git commit -m "some msg"`
    9. 查看操作的历史记录  `git log`  可以使用参数 `--pretty=oneline`  使输出信息变得简洁
    10. 版本回退:   
        1. 回退到上一个版本 `git reset --hard HEAD^`
        2. 回退几个版本就加几个 `^`
        3. 可以用数字代替  回退100次  `git reset --hard HEAD~100`
    11. 回到未来的版本 需要知道提交时的commit id  然后使用 `git reset --hard "commit id"`
    12. 如果没有了commit id  可以使用 `git reflog` 查看历史的每一次记录
    13. 撤销修改
        - 撤销工作区的修改  `git checkout -- file`  例如: `git checkout -- readme.txt`
        - 撤销暂存区的修改  `git reset HEAD file` 就会撤销暂存区的修改,但是内容会回到工作区,需要再撤销工作区的修改
    14. 删除修改
		1. 直接区目录下删除  或者用命令 `rm file` 
		2. `git rm file` 
		3. `git commit`  就删除了
		4. 如果删错了,使用  `git checkout -- readme.txt`  就可以了
		5. `git checkout` 其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”
 
##### 密钥管理

- 密钥管理工具  ssh-agent 的使用
    1. `ssh-add -D` 删除所有管理的密钥
    2. `ssh-add -d` 删除指定的
    3. `ssh-add -l` 查看现在增加进去的指纹信息
    4. `ssh-add -L` 查看现在增加进去的私钥
    5. 如果重启之后，会发现需要重新load一下ssh-agent
        - `ssh-add -K` 将指纹加到钥匙串里面去
        - `ssh-add -A` 可以把钥匙串里面的私钥密码，load进ssh-agent

##### 推送到github

- 在github端创建一个新的仓库,也在本地建一个仓库 `git init`
- 将本地的仓库和远程仓库关联起来 <br />`git remote add origin git@github.com:zonzie/zonzie.github.io.git`
- 添加后，远程库的名字就是origin，这是Git默认的叫法，也可以改成别的，但是origin这个名字一看就知道是远程库。
- 将本地库的内容推送到远程库 `git push -u origin master`  实际上是把当前分支master推送到远程
- 由于远程库是空的，我们第一次推送master分支时，加上了`-u`参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令
- 现在起 只要本地做了提交,就可以通过命令,把把本地master分支的最新修改推送至GitHub   `git push origin master`

##### 克隆

- 使用  git clone 命令   `git clone 项目地址`
`git clone git@github.com:zonzie/zonzie.github.io.git`
- 修改后推送到远程仓库 
`git push`
- 更新本地仓库
`git pull`

##### 分支管理
- 创建一个分支
`git checkout -b dev` 加上-b表示创建并且切换到新的分支,相当于以下两条命令
 `git branch dev`,`git checkout dev`
- 查看所有的分支`git branch`
- 合并分支 `git merge xxx` 合并xxx分支到当前分支  是快进模式,合并速度非常快
- 删除分支 `git branch -d xxx`  删除xxx分支
- 删除远程分支 xxx `git push origin --delete xxx`
- 解决冲突
    - `git merge xxx` 合并分支后,可能出现冲突
    - 需要手动修改冲突文件,再提交--冲突解决
    - 使用`git log`查看分支的合并情况或者`git log --graph`
- 合并分支,禁用fast forward模式(默认)--删除分支后,任然保留分支信息
    - 合并时使用命令 `git merge --no-ff xxx`  表示禁用fast forward模式,合并xxx分支到当前分支
- 推送分支
    - 命令 `git push origin master`
    - 推送到dev分支,就改成 `git push origin dev`
- 放弃本地修改,强制拉取更新
    - `git fetch` 指令是下载远程仓库最新内容，不做合并 
    - `git reset` 指令把HEAD指向master最新版本
    - 步骤: 
        1. `git fetch --all`
        2. `git reset --hard origin/master`
        3. `git pull` //可以省略
- 新建远程分支
	- 先创建一个本地的分支
        `git checkout -b newBranch`
	- 推送本地分支到远程,远程分支也叫newBranch,可以是别的
        `git push origin newBranch:newBranch`
- 删除远程分支
	- 可以推送空的分支到远程分支,就可以删除远程分支
		`git push origin :newBranch`
	- 也可以直接删除远程分支
		`git push origin --delete newBranch`