# Submodule

## 子模组协同模型
应用场景：项目的版本库在某些情况下需要引用其他版本库中的文件，例如需要引用公司积累了很久的共用函数库，此函数库同时被多个项目所调用。显然这个函数库不能放在某一项目的代码中，要为其创建一个独立的代码库，那么其他项目要如何对其进行调用呢？子模组协同模型所解决的就是这样一个问题。

---
## 创建子模组
先尝试创建两个公共函数库(libA.git和libB.git)，以及一个引用函数库的主版本库(super.git)。

    git --git-dir=./libA.git init --bare
    git --git-dir=./libB.git init --bare
    git --git-dir=./super.git init --bare

向两个公共的函数库中填充些数据.这就需要在工作区克隆两个函数库,提交数据并推送。

克隆libA.git版本库，添加一些数据并推送。

    git clone /path/to/repos/libA.git /path/to/my/workspace/libA
    cd /path/to/my/workspace/libA
    [create a file named libA.c]
    git add .
    git commit -m "add data for libA"
    git push origin master

克隆libB.git版本库，添加一些数据并推送。

    git clone /path/to/repos/libB.git /path/to/my/workspace/libB
    cd /path/to/my/workspace/libB
    [create a file named libB.c]
    git add .
    git commit -m "add data for libB"
    git push origin master

克隆super版本库[准备在其中创建子模组],在其中完成一个提交(可以为空)并提交

    git clone /path/to/repos/super.git /path/to/my/workspace/super
    cd /path/to/my/workspace/super
    git commit --allow-empty -m "initialized"
    git push origin master

现在就可以在super版本库中使用git submodule add 指令了。

    git submodule add /path/to/repos/libA.git lib/libA
    git submodule add /path/to/repos/libB.git lib/libB
    
此时在super工作目录下多了一个.gitmodules 文件

    > cat .\.gitmodules
    [submodule ".\\lib\\libA"]
        path = .\\lib\\libA
        url = ../../libA.git
    [submodule ".\\lib\\libB"]
        path = .\\lib\\libB
        url = ../../libB.git

此时工作区尚未提交

    >git status
    On branch master
    Your branch is up to date with 'origin/master'.

    Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)

        new file:   .gitmodules
        new file:   lib/libA
        new file:   lib/libB    

完成提交后,子模组才算正式在super版本库中创立.运用git push把刚建立了新模组的本地版本库推送到远程版本库。

在提交过程中我们发现，作为子模组方式添加的版本库实际上并没有添加版本库的内容.只是以gitlink的方式添加了一个链接.

## 克隆带子模组的版本库
下面在另外的位置克隆super版本库，会发现lib/libA 和 lib/libB并未克隆。

    git clone /path/to/repos/super.git /path/to/my/workspace/super_clone
    cd /path/to/my/workspace/super_clone

这时如果运行**git submodule status**可以查看子模组状态

    git submodule status
    -b4bfa72bdbe93476cb7778e0c4a90eae47c655c4 lib/lib_a
    -90645c166812e4dc64ee7d10053f3ca6f6eb8783 lib/lib_b

可以看到每个子模组前面都是40位的提交id，最前面是一个减号，表示子模组未被检出。
如果需要克隆出子模组形式的外部引用库，需要首先执行**git submodule init**

    git submodule init
    Submodule 'lib/lib_a' (C:\Users\ZD\project\Github\learngit\submodule\libA.git) registered for path 'lib/lib_a'
    Submodule 'lib/lib_b' (C:\Users\ZD\project\Github\learngit\submodule\libB.git) registered for path 'lib/lib_b'

执行git submodule init操作s实际上修改了.git/config文件，对子模组进行了注册。然后执行**git submodule update**完成子模组的克隆。


## 在子模组中修改和子模组的更新
执行**git submodule update**更新出来的子模组都以某个具体的提交版本进行检出。进入某个子模组目录，会发现其处于非跟踪状态。
    
    git branch
    * (HEAD detached at b4bfa72)
    master

显然在这种状态下，修改lib/lib_a下的文件，提交就会丢失。下面介绍如何在检出的子模组下进行修改，以及如何更新子模组。
在子模组下切换到master分支(或者其他分支)后再进行修改。

> (1)切换到master分支，然后在工作区做出一些改动

    git checkout master
    hack...

> (2)执行提交

    git add .
    git commit -m "some comment."

> (3)查看状态，会看到相对于远程分支领先一个提交

    git status
    On branch master
    Your branch is ahead of 'origin/master' by 1 commit.
    (use "git push" to publish your local commits)

    nothing to commit, working tree clean

> (4)先到super_clone版本库查看一下状态，可以看到子模组已经修改，包含了更新的提交。

    git status
    On branch master
    Your branch is up to date with 'origin/master'.

    Changes not staged for commit:
    (use "git add <file>..." to update what will be committed)
    (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   lib/lib_a (new commits)

    no changes added to commit (use "git add" and/or "git commit -a")

> (5)通过**git submodule status**可以看到lib/lib_a子模组指向了新的提交ID(后面有一个加号)，而lib/lib_b子模组状态正常。

    git submodule status
    +25ebf8a49a92eaed05ce50c929994e09fb3c1345 lib/lib_a (heads/master)
    90645c166812e4dc64ee7d10053f3ca6f6eb8783 lib/lib_b (heads/master)

> (6)这时候如果不小心执行了一次**git submodule update**命令，会将lib/lib_a重新切换到就得指向。

    git submodule update
    Submodule path 'lib/lib_a': checked out 'b4bfa72bdbe93476cb7778e0c4a90eae47c655c4'

> (7)执行**git submodule status**查看子模组状态，可以看到lib/lib_a子模组被重置了。

    git submodule status
    b4bfa72bdbe93476cb7778e0c4a90eae47c655c4 lib/lib_a (remotes/origin/HEAD)
    90645c166812e4dc64ee7d10053f3ca6f6eb8783 lib/lib_b (heads/master)

但是刚才的提交其实并没有消失，实际上更新已经提交到了master分支，但是由于此时lib/lib_a的子模组重置为之前的分支，而非master分支，所以使用这种方法对子模组进行修改是行不通的。


----
重新来过！

> (1)进入到lib/lib_a目录下，看到工作区再一次进入分离头指针状态。

    git branch
    * (HEAD detached at b4bfa72)
    master

> (2)重新切换到master分支，找回之前的提交

    git checkout master
    Previous HEAD position was b4bfa72 add data for libA
    Switched to branch 'master'
    Your branch is ahead of 'origin/master' by 1 commit.
    (use "git push" to publish your local commits)

现在如果要将lib/lib_a目录下子模组的改动记录到父项目(super)中，就需要在父项目中进行一次提交才能实现。

> (1)进入父项目根目录(super_clone)查看状态。

    git ststus -s
    M lib/lib_a

> (2)查看差异比较，会看到指向子模组的gitlink有改动。

    git diff
    diff --git a/lib/lib_a b/lib/lib_a
    index b4bfa72..25ebf8a 160000
    --- a/lib/lib_a
    +++ b/lib/lib_a
    @@ -1 +1 @@
    -Subproject commit b4bfa72bdbe93476cb7778e0c4a90eae47c655c4
    +Subproject commit 25ebf8a49a92eaed05ce50c929994e09fb3c1345

> (3)将gitlink的改动添加到暂存区，然后提交。

    git add -u
    git commit -m "submodule lib/lib_a upgrade to new version."

此时还不能急着推送，因为如果执行**git push**将super_clone版本库推送到远程版本库，会引发一个问题。即推送后的远程super版本库的子模组lib/lib_a指向了一个新的提交，而该提交还在本地的lib/lib_a版本库(尚未向上游推送)，这会导致其他人克隆super版本库和更新模组时因为找不到该子模组版本库相应的提交而出错。

为了避免这种可能性的发生，最好先推送lib/lib_a中的提交，然后再向super版本库推送更新的子模组gitlink改动

    cd /path/to/workspace/super_clone/lib/lib_a
    git push
    cd /path/to/workspace/super_clone/
    git push


