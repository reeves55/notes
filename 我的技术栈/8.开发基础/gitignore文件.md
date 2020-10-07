# git & .gitignore



## git基本原理



![img](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/1090617-20181008211557402-232838726.png)



三个区域：



```工作区```

工作区当中的文件分为tracked和untracked两种文件，tracked是被纳入该项目版本管理的文件，工作区文件可以做任意修改，这些修改都会被作为变更，git status能够显示出来（包括tracked files和untracked files），变更可以添加到暂存区，作为后续提交到版本库的修改的一部分

```shell
# 工作区修改 -> 暂存区（添加）
git add xxx

# 暂存区 -> 工作区（覆盖）
git checkout xxx


```





```暂存区```

暂存区可以原子性地提交从工作区添加到暂存区的修改到版本库，所以，暂存区是通向版本库的唯一通道，如果我们一次性做了多个修改，但是想要分多个版本提交到版本库，那就需要往暂存区当中分批添加文件，多次提交暂存区的内容





```本地版本库```

版本库的作用就是它能够实现版本记录和回滚，.git目录下都有一个HEAD指针，指向目前所在版本库的版本





git add 命令可以把工作区对文件的修改放到暂存区，可以多次执行 git add 把多次修改都增量放到暂存区里面，暂存区

![git-stage](https://www.liaoxuefeng.com/files/attachments/919020074026336/0)





![git-stage-after-commit](https://www.liaoxuefeng.com/files/attachments/919020100829536/0)





## .git文件夹















## .gitignore的作用

用来配置git文件夹当中 ```不会被提交到仓库``` 的文件，但是如果git已经track了一个文件，再配置.gitignore文件是没有用的，不能达到忽略该文件的效果，只能在远程仓库删除，本地也删除，才能忽略该文件。



不管.gitignore文件是否被track，只要.gitignore指定的文件没有被track，这些文件都不会被提交到仓库，但是如果.gitignore文件没有添加到仓库当中，项目其他维护人员就可能向项目添加不被允许的文件，因此：

**只要有.gitignore文件存在，且目标文件没有在.gitignore生成前被tract，就能起到屏蔽效果；**

![image-20200918174111755](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200918174111755.png)



如果已经 git add 到了本地暂存区，如果把应该ignore的文件重新屏蔽❓
