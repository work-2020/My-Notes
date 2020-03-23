# Git
+ 进入仓库所在文件夹，右键启动Git Bash
+ 查看仓库状态 `git status`
+ 仓库文件发生变化后，命令行前缀出现$\color{red}{*}$，`git status` 也可以看到变化
+ 将文件添加到暂存区，`git add 1.txt`
+ 撤销 `git reset -- 1.txt` 或 `git rm --cached 1.txt`；省略文件名`git reset --`，将暂存区的全部提交撤销
+ 查看工作区被跟踪的文件的修改详情 `git diff`
+ 将修改后的文件提交到暂存区后，使用`git diff --cached`查看暂存区文件的修改详情
+ 查看历史提交记录 `git log`
+ 将暂存区的文件提交到版本区 `git commit -m '注释'`
+ 将本地版本区的文件推送到GitHub远程仓库 `git push`
+ 查看仓库分支信息 `git branch -avv`
+ 版本回退 `git reset --soft HEAD^`，`--soft`表示软退回，`HEAD^`撤销最近一次提交
+ `git push`遇到版本冲突时，使用`git pull`拉取远程仓库到本地仓库，会看到发生冲突的文件，打开发生冲突的文件，会在文件中发现<<<<<<< HEAD ，=======  ，>>>>>>>ae9a0f6b7e42fda2ce9b14a21a7a03cfc5344d61这种标记，<<<<<<< HEAD和=======中间的是你自己的代码，  =======  和>>>>>>>中间的是其他人修改的代码。自己确定保留那一部分代码，最后删除<<<<<<< HEAD ，=======  ，>>>>>>>这种标志，再重新add, commit, push
+ 遇到版本冲突时，也可以`git push -f`，强制推送
+ 本地仓库提交记录`git reflog`显示本地仓库所有分支的每一次版本变化
+ 回退到本地仓库任意一次提交的版本`git reset --hard HEAD@{n}`这里的`n`根据`git reflog`的结果设定自己想要回退的版本号
+ git reset
  - `git reset -- 1.txt` 撤销添加到暂存区的文件1.txt
  - `git reset --soft HEAD^`，`--soft`表示软退回，`HEAD^`撤销最近一次提交，仅仅撤销版本区，不改变暂存区和工作区
  - `git reset --mixed 版本库ID` 撤销已提交的版本区和暂存区，不改变工作区
  - `git reset --hard HEAD@{n}`彻底改变工作区、暂存区和版本区

## Git分支操作
### 添加SSH关联操作
+ 终端运行`ssh-genkey`命令，再按几次回车生成公私钥，公私钥存在主目录下隐藏文件夹.ssh中
+ 将id_rsa.pub文件中的公钥复制出来，在GitHub中-> settings -> SSH and GPG keys -> new sshkeys，粘贴在KEYS中，添加即可

### 分支管理
+ `git branch -avv` 结果中 remotes开头的行就是本地保存的远程分支的信息
+ `git fetch` 刷新保存在本地的远程分支的信息，当GitHub中远程分支发生修改后，该命令可更新本地的远程分支信息
+ `git pull` 拉取GitHub远程分支到本地
+ 首先克隆远程仓库到本地，进入仓库目录，执行`git branch [分支名]`可以创建新的分支
+ 创建完分支后，仍然位于master分支上，执行`git checkout [分支名]`切换分支；先创建分支再切换略显繁琐，执行`git checkout -b [分支名]`创建分支并切换到新分支
+ `git push [主机名][本地分支名]:[远程分支名]`，将本地分支推送到远程分支上，如果远程分支不存在，自动创建。通常本地分支名与远程分支名相同，可以省略冒号及远程分支名，`git push origin dev`
+ 本地分支跟踪远程分支 `git branch -u [主机名/远程分支名] [本地分支名]`，如果是设置当前本地分支跟踪远程分支，可以省略本地分支名
+ `git push -u [主机名] [本地分支名]` 将本地分支推送到远程分支上，并自动跟踪。
+ 撤销本地分支对远程分支的跟踪，`git branch -unset-upstream [本地分支名]`，同样地，如果是撤销当前本地分支的跟踪，可以省略本地分支名
+ 删除远程分支，将本地空白分支推送到远程分支即可 `git push [主机名] :[远程分支名]`
+ 删除本地分支， `git branch -D [本地分支名]`，需要注意的是不能删除当前所在的分支

