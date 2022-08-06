

# git 为什么要先commit，然后pull，最后再push？而不是commit然后直接push？

情况是这样的，现在远程有一个仓库，分支就一个，是master。然后我本地的仓库是从远程的master上clone下来的。大家都是clone下来，再在自己本地改好，再commit然后pull然后push，大家都是这么做的。那么现在问题来了：

1，那我本地这个也算是个分支？还是就是一个本地仓库？

答：本地和远程的关系相当于两个分支,你感觉一样是因为你`git pull` 的时候已经自动给绑定好对应关系了, set-upstream..balbala

2，如果我在远程新建了个分支，然后我pull了下来，那我本地到底有分支这个说法吗？我本地的分支是不是就是那个远程新建的分支？

答：你远程新建了一个分支拉到本地的道理是一样的,属于复制了一份,但是本地分支和远程分支已经是两个东西了

3，本地仓库和本地分支有什么区别？

答：本地分支属于本地仓库里,是包含关系,一个仓库里可以有很多分支

4，commit是提交到本地仓库，然后push，这个push是把所有代码推到远程仓库，还是只是把commit的地方推到远程仓库？

答：肯定不会全量推送到远程的,是通过对比 commit 的记录,如果本地高于远程就直接把多出来的`commit` 给怼上去,如果本地分支的最新版本和远程的 `commit` 有冲突，就需要解决冲突。

**5，那为什么要先commit，然后pull，然后再push，我pull了，岂不是把自己改的代码都给覆盖掉了嘛，因为远程没有我改的代码，我pull，岂不是覆盖了我本地的改动好的地方了？那我还怎么push？**

**答：这个先 commit 再 pull 最后再push 的情况就是为了应对多人合并开发的情况,**

1. **`commit` 是为了告诉 git 我这次提交改了哪些东西,不然你只是改了但是 git 不知道你改了,也就无从判断比较;**
2. **`pull`是为了本地 commit 和远程commit 的对比记录,git 是按照文件的行数操作进行对比的,如果同时操作了某文件的同一行那么就会产生冲突,git 也会把这个冲突给标记出来,这个时候就需要先把和你冲突的那个人拉过来问问保留谁的代码,然后在 `git add && git commit && git pull` 这三连,再次 pull 一次是为了防止再你们协商的时候另一个人给又提交了一版东西,如果真发生了那流程重复一遍,通常没有冲突的时候就直接给你合并了,不会把你的代码给覆盖掉**
3. **出现代码覆盖或者丢失的情况:比如A B两人的代码pull 时候的版本都是1,A在本地提交了2,3并且推送到远程了,B 进行修改的时候没有`commit` 操作,他先自己写了东西,然后 `git pull` 这个时候 B 本地版本已经到3了,B 在本地版本3的时候改了 A 写过的代码,再进行了`git commit && git push` 那么在远程版本中就是4,而且 A 的代码被覆盖了,所以说所有人都要先 commit 再 pull,不然真的会覆盖代码的**

6，两个分支A和B，A合并B和B合并A，有区别吗？

答：两个互相合并的唯一区别就是 A->B 的时候 B 分支上会产生一个 merge_commit ，被改变的分支是 B ；如果现在**没有发生任何改动**执行 B->A ，则A和B两分支才会完全相同。





# Git提交时过滤某些文件

**`.gitignore`文件，这个文件的作用就是告诉Git哪些文件不需要添加到版本管理中**

* **如果没有改文件使用`touch .gitignore`新建一个**

  ![image-20220412224417067](https://blog.zhaobincode.cn/blogimages/202204122244108.png)

**忽略规则：**

```
target          # 忽略这个target目录
angular.json    # 忽略这个angular.json文件
log/*           # 忽略log下的所有文件
css/*.css       # 忽略css目录下的.css文件
/Library/    # /文件名/的意思就是当前路径下的Library文件夹，都不提交

bin   #所有路径下的Bin都不提交 

!/Assets/    # 和上面一句对比，这里加了个！，这就是说，这个Assets文件夹要被提交

/Logs/*.bak  # Logs下面所有的.bak结尾的文件，不被提交

!debug.bak    # debug.bak文件除外（不会被忽略）

!/Packages/*.h # Packages下面的所有.h文件，要被提交

Temp/version.txt  # 忽略Temp目录下的version.txt文件

/Temp/*
!/Temp/var/
这两句都写，就是不提交Temp文件夹,但是提交Temp里面的var文件夹，这种骚操作都可以.
```



`.gitignore`只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。

##### 解决方案：

```js
git rm -r --cached .	//清理本地git缓存，注意后面那个点
git add .
git commit -m "update .gitignore"
```



# 删除GitHub上文件

在GitHub上不能删除文件,只能从GitHub仓库拉到本地操作,操作完成之后在上传到GitHub仓库

[如何删除gitee仓库的文件 - 编程猎人 (programminghunter.com)](https://www.programminghunter.com/article/65122115403/)

