# 刚安装git

```shell
# 产生公钥
ssh-keygen -t rsa -C "your_email@youremail.com"

# 设置用户名邮箱
git config --global user.name "xxx"
git config --global user.email "xxx@163.com"
```



# 新建分支并提交到远程

```shell
git checkout -b dev
git push --set-upstream origin dev


#本地分支和远程分支建立关联
git branch --set-upstream-to=origin/dev dev
```



# 删除分支

```shell
# 删除远程分支
git push origin --delete dev

# 删除本地分支
git branch -d dev
# 强制删除本地分支
git branch -D dev
```



# 切换分支

```shell
# git pull = git fetch + git merge
# 当git pull时出现merge冲突，然后git checkout时无法切换，可以执行git reset --merge
git reset --merge


# 当自己所在分支修改未完成，又需要切换到新的分支时，无法切换，此时可以使用git stash进行隐藏
git stash
git status # 可以看到没有内容需要提交
# 当需要继续修改时，切到该分支，git stash apply
git stash apply #呈现之前修改的
git stash drop #删除当时的保存记录

# 参考网址：https://blog.csdn.net/asheandwine/article/details/79003270


```



# 回滚

```shell
# 修改之后未add时撤回修改
git checkout -- <file>

# add之后的回退
git reset HEAD <file>

git reset <--hard/soft/mixed(默认)> HEAD~1
## --mixed 
#不删除工作空间改动代码，撤销commit，并且撤销git add . 操作
#这个为默认参数,git reset --mixed HEAD^ 和 git reset HEAD^ 效果是一样的。
## --soft  
#不删除工作空间改动代码，撤销commit，不撤销git add . 
## --hard
#删除工作空间改动代码，撤销commit，撤销git add .

#此回退仅将本地分支回退，远程分支并没有回退，所以会造成本地落后于远程，push时需要让你merge，但merge之后原先错误的修改又merge进去了。此操作可以用于commit之后，用到push之后的话有点麻烦
#or git reset --hard commitID 
#要想远程也回到上一版本，需要
git push -f origin master

```



# 拉取新分支

```shell
# 不需要专门切换到master分支
git fetch
git checkout -b {branch} origin/{branch}

# 专门拉取特定分支
git clone -b {branch} {url}
```



# fork之后根据进行更新

```sh
git checkout master
git remote -v
git remote add upstream https://github.com/Alan-kang/LeetCode.git
git remote -v
git branch
git fetch upstream
git branch -a
git merge upstream/master
git status
git push
```



# 解决冲突

参考链接：https://www.liaoxuefeng.com/wiki/896043488029600/900004111093344

```bash
git status # 查看冲突文件,对文件进行修改

git add {conflict-file}
git commit -m "fix conflict"

```



# git rebase使用

参考链接：

使用：https://www.jianshu.com/p/f7ed3dd0d2d8

解释：https://www.jianshu.com/p/4079284dd970

- 同一分支rebase

```bash
# 当前提交分支对应的远程分支已经被人修改，需要pull后再提交，但又不想提交历史分叉，通过rebase可以解决，将自己的修改添加到最后
git pull --rebase

## 与上面等价
git pull
git rebase
```

- 分支之间rebase

```bash
git add .
git commit -m "add feature"

git checkout master
git pull
git checkout feature
git rebase master

## 如果出现冲突，先解决冲突，打开冲突文件修改，修改完成后
git add conflict.file
git rebase --continue

git log --oneline --graph # 查看提交图

```



# git cherry-pick

可用于将另一个分支某个提交 添加到当前分支

参考链接 https://www.ruanyifeng.com/blog/2020/04/git-cherry-pick.html

```shell
git cherry-pick <commitId>

# 发生冲突解决 可以参考链接(我没遇到过)
git status # 查看冲突文件,对文件进行修改

git add {conflict-file}
git cherry-pick continue
```



 查看commit内容  

 git show commit_id



# tag

```shell
# 切换到相应分支打tag
git checkout <branch_name>
git tag <tagname>
git tag # 查看tag

# 创建标签都只存在本地，不会自动推送到远程,以下命令可以推送到远程
git push origin <tag_name>

# 查看提交历史
git log --pretty=oneline --abbrev-commit
git tag <tag_name> <commitID>

# 查看tag对应的提交信息
git show <tag_name>

# 带有说明的标签
git tag -a <tag_name> -m "comment" <commitID>

# 删除标签(本地)
git tag -d <tag_name>

# 创建标签都只存在本地，不会自动推送到远程,以下命令可以推送到远程
git push origin <tag_name>
# 推送全部本地未推送的标签
git push origin --tags

# 删除远程标签
git push origin :refs/tags/<tag_name>
```



将已有的目录推送到github

在github新建一个项目，然后进入推送的目录，执行以下操作

```shell
cd <对应的目录>
git init
git remote add origin <git地址>
git add .
git commit -m "init"
git push -u origin master
```





拉取远程已经有的分支

```shell
git branch -r # 查看远程分支
git checkout origin/feature # 切换到远程分支
git checkout -b feature # 给分支命名
```



 修改远程地址由https为ssh

```bash
git remote set-url origin git@****.git
```



