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
git reset --hard HEAD~1
#此回退仅将本地分支回退，远程分支并没有回退，所以会造成本地落后于远程，push时需要让你merge，但merge之后原先错误的修改又merge进去了。此操作可以用于commit之后，用到push之后的话有点麻烦
#or git reset --hard commitID 
#要想远程也回到上一版本，需要
git push -f origin master

# 以下指令可以再进行一次提交，比远程提前一分支，也可以实现回退到上一版，可以用于push之后
# 参考网址https://blog.csdn.net/yu870646595/article/details/51443279
git revert commitId # 舍弃commitId这次提交
```



拉取新分支

```shell
# 不需要专门切换到master分支
git fetch
git checkout -b {branch} origin/{branch}

```

