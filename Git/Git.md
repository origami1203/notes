# Git

### git乱码

```sh
$ git config --global core.quotepath false # 设置 git status utf-8编码

$ git config --global gui.encoding utf-8 # 设置Git GUI界面utf-8编码

$ git config --global i18n.commit.encoding utf-8 #设置commit信息utf-8编码

$ git config --global i18n.logoutputencoding utf-8 # 设置输出 log utf-8 编码
```

### 命令行操作

 ```properties
git软件版本查询 : git --version
 ```

 ```properties
# 首先需要配置名字及邮箱，作为标识
# --global参数，当前用户使用
# --system 当前操作系统的所有用户使用
# 不写只对当前工作目录
git config --global user.name "zhangsan"
git config --global user.email "zhangsan@163.com"

# 查询所有的配置
git config --list
 ```

```properties
# linux操作
cd : 进入某个目录
mkdir : 创建一个文件
pwd : 显示当前的目录路径
```

```properties
git init : 初始化仓库
# 初始化后 目录下生成.git目录
# hooks 钩子，里面有一些钩子设置，比如提交前执行什么命令
# info 	排除文件
# objects 	所有存储内容
# refs 		分支和tag
# config 	配置信息
# description 	仓库描述
# HEAD	分支
# index	暂存区
git add 目录/文件 : 将对象添加到暂存区(添加到暂存区的文件修改后需要再次添加)
git commit -m "更新内容的注释" : 提交暂存区的内容到版本库(不会清空暂存区)
git commit -a : 提交所有已跟踪文件，不需要git add
```

```properties
git status : 查看仓库当前的状态，显示有变更的文件。

# 查看文件的变化
git diff 文件 : 	比较文件的不同，即暂存区和工作区的差异
git diff 文件 --cached/staged(1.6以后版本) : 已暂存，还未提交
git diff 版本hash 文件名 : 与历史版本比较

# 删除
git rm 文件 : 直接commit即可
git rm --cached <file> : 从缓存区取消

# 重命名
mv
git mv
# 日志
git log
git reglog : 获取历史版本号
git log --oneline

# 跳转到指定版本
# 分为--hard --soft --mixed
# --hard 你的stage区和工作目录里的内容会被完全重置为和HEAD的新位置相同的内容。换句话说，就是你的没有commit的修改会被全部擦掉。
# --soft 为commit的数据被保存到缓存区
git reset --hard hash值 : 到指定版本
git reset --hard HEAD^ : 回退到上一个版本
git reset --hard HEAD$ : 到下一版本
```

分支

```properties
# 查看当前分支
git branch : 显示所有本地分支
git branch -a : 显示所有分支(本地和 远程)
git log --oneline --decorate --graph --all : 查看全部的分支日志情况
git branch -v : 查看每一个分支的最后一次提交

# 创建分支
git branch 分支名 : 创建分支
git branch 分支名字 版本hash : 创建一个分支，并将其指向指定的hash的版本(通过log查询hash)	

#删除分支
git branch -d branchName : 删除分支(不能处于删除分支)
git branch -D branchName : 强制删除分支(不能处于删除分支)
git push origin --delete branchName : 删除远程分支


# 切换分支
git checkout 分支名 : 切换分支
git checkout -b 分支名 : 创建并切换到指定分支
# 注意切换分支之前，记得提交当前分支


# 合并分支
git merge 分支名 : 将指定分支(有新内容的分支)与当前分支(比如master分支)合并

####################################################################
# 冲突
# 分支合并，两个分支都出现了修改，会出现分支冲突，需要把冲突文件修改
 8 gggggggg
 9 <<<<<<<HEAD
10 hhhhhhhh edit by hot_fix
11 =======
12 hhhhhhhh edit by master
13 >>>>>>>master
14 11111

# <<<<<<<HEAD 表示以下当前合并版本的冲突代码
# >>>>>>>master 表示以下是被合并版本的冲突代码
# 修改冲突后，使用git add添加，git commit提交即可
```

别名

```properties
git config --global alias.别名 要起别名的命令
# 给git log --oneline --decorate --graph --all命令添加别名allbranch，注意要起别名的命令前部不要加git
git config --glocal alias.allbrchch "log --oneline --decorate --graph --all"
```

github

```properties
git romote -v : 查看远程仓库

# 给远程仓库起别名
git remote add 别名 github的地址
git remote add origin https://github.com/origami-x/first.git

# 加入团队
项目仓库 -> settings -> Manage access -> Invite a collaborator(查询指定的github账号)

# 从远程库克隆下来
# 它不仅克隆了代码，也初始化了.git，并自动配置远程别名
git clone [远程地址] : 克隆到本地

# 推送到远程库
# 此操作只能创建者或被邀请加入团队的人才能推送到
# 如果push的时候，不是基于github的最新版本，不能push，必须先拉取最新版本，若拉取后有冲突，需要解决冲突
git push [远程地址别名] [要推送的分支名]
git push origin master


# 拉取
# pull = fetch + merge
git fetch [远程库地址别名] [远程分支名] : 从远程库拉取下来，有冲突时并没有改变本地的文件内容，自己决定是否合并
git merge [远程库地址别名/远程分支名] : 将远程分支与当前本地分支进行合并
git pull [远程库地址别名] [远程分支名] : 从远程拉取

# fetch 可用于更新远程仓库，如远程创建了dev分支，可以使用fetch进行同步远程仓库，pull不行

# fork及pull request
# github的fork即是复制一整个项目到自己的账号，自己成为仓库的所有者，与原仓库的数据一致
# pull request即是将fork修改后的代码与原仓库进行合并
```

### ssh

```ini
# 生成ssh
ssh-keygen -t rsa -C "xxx"
```

