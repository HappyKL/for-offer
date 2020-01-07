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



拉取新分支

```shell
# 不需要专门切换到master分支
git fetch
git checkout -b {branch} origin/{branch}

```

