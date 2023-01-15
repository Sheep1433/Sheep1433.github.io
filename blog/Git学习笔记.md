# git核心

![Screenshot_20220416_171404_tv.danmaku.bili](D:\Huawei Share\Huawei Share\Screenshot_20220416_171404_tv.danmaku.bili.jpg)

# git登录 

查看当前登录账号：

```shell
git config user.name
```

查看当前登录邮箱

```shell
git config user.email
```

修改用户名和邮箱：

```shell
git config --global user.name "Your_username"
git config --global user.email "Your_email"
```

初始化一个Git仓库，使用`git init`命令。

添加文件到Git仓库，分两步：

1. 使用命令`git add <file>`，注意，可反复多次使用，添加多个文件；
2. 使用命令`git commit -m <message>`，完成。

- 要随时掌握工作区的状态，使用`git status`命令。
- 如果`git status`告诉你有文件被修改过，用`git diff`可以查看修改内容。

# 版本切换

- `HEAD`指向的版本就是当前版本，HEAD^表示上一个版本，HEAD^^表示上上个版本，HEAD~100表示往上100个版本，因此，Git允许我们在版本的历史之间穿梭，使用命令`git reset --hard commit_id`，commit_id版本号不需要写全，只需要写前几个。
- 穿梭前，用`git log`可以查看提交历史，以便确定要回退到哪个版本。
- 要重返未来，用`git reflog`查看命令历史，以便确定要回到未来的哪个版本。

git把本地仓库推到github上

```
git remote add origin https://github.com/Sheep1433/learngit.git
```

![image-20211223210005615](C:\Users\DESKTOP\AppData\Roaming\Typora\typora-user-images\image-20211223210005615.png)

```
git checkout -- file 撤销对file的修改
```

在本机上创建ssh key，然后在主目录的.ssh文件夹中将公钥复制到github或者gitee中即可。

```
ssh-keygen -t rsa -C "youremail@example.com"
```

然后在本地仓库目录下运行以下命令，将本地仓库和远程库关联

```
git remote add origin git@github.com:Sheep1433/learngit.git
或者git remote add origin https://github.com/Sheep1433/learngit.git
# git支持多种协议，如ssh和https,默认的git://使用的是ssh协议，速度快
```

将本地仓库内容推送到远程库

```
git push -u origin master
```

第一次推送master分支时，加上-u参数，会同时将本地master和远程库的master关联起来，以后的推送或拉取可以简化命令

如

```
git push 或者 git push origin master
```

**解除本地仓库和远程库的链接**

查看远程库信息

```
git remote -v
```

然后删除

```
git remote rm origin
```

# 分支管理

创建dev分支并且切换

```
git checkout -b dev		等同于 git branch dev 加上git checkout dev也等同于git switch -c dev
```

使用git branch查看分支

在dev分支中完成代码工作后，并使用git add和git commit 提交后，切换至master分支，是无法看到dev分支的工作的，通过在master分支中使用git merge dev命令合并分支。		

```
删除分支	git branch -d dev
创建分支并切换		git checkout -b dev
查看分支	git branch
查看日志	git log
合并分支	git merge dev
禁用Fast-Forward模式进行合并分支	git merge --no-ff -m "merge with no-ff" dev
git stash命令的场景假设：
	保留工作现场		git stash
	切换master分支 	 git checkout master
	创建新分支		git checkout -b issue001
	git add 和git commit -m提交修复
	切回master分支	 git switch master
	将issue分支合并入master分支	git merge --no-ff -m "merge with no-ff" issue001
	切回dev分支		git switch dev
	查看工作现场		git stash list
	恢复现场并删除		git stash apply和git stash drop等同于git stash pop
恢复指定的stash	git stash apply stash@{0}
删除还没有合并的分支	git branch -D feature-vulcan
从本地推送分支		git push origin branch-name
如果推送失败，先用git pull抓取
在本地创建和远程分支对应的分支		git checkout -b branch-name origin/branch-name
建立本地分支和远程分支的管理		git branch --set-upstream-to branch-name rigin/branch-name
查看日志	git log
查看日志	git log --pretty=oneline --abbrev-commit
打上标签	git tag v1.0
查看标签	git tag
针对指定commit id打上标签	git tag v0.9 f52c633

```

将推送的远程库名字改成不同的，就可以将一个本地库推送到多个平台

```
git remote add github git@github.com:Sheep1433/learngit.git
git remote add gitee git@github.com:Sheep1433/learngit.git
```

删除github上的文件夹，在本地仓库使用git rm -r --cached 文件夹名，再使用git commit和git push

下载源码时切换版本	git clone 源码， git tag 列出所有版本，git checkout 版本号

参与到别人的开源项目中

在github中fork项目，然后git clone自己的仓库，这样才能提交推送，如果想让官方接受修改，在github上pr(pull requests)
