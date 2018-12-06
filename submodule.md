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