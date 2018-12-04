# Submodule

## 子模组协同模型
应用场景：项目的版本库在某些情况下需要引用其他版本库中的文件，例如需要引用公司积累了很久的共用函数库，此函数库同时被多个项目所调用。显然这个函数库不能放在某一项目的代码中，要为其创建一个独立的代码库，那么其他项目要如何对其进行调用呢？子模组协同模型所解决的就是这样一个问题。

---
## 创建子模组
先尝试创建两个公共函数库(libA.git和libB.git)，以及一个引用函数库的主版本库(super.git)。

    git --git-dir=./libA.git init --bare
    git --git-dir=./libB.git init --bare
    git --git-dir=./super.git init --bare
