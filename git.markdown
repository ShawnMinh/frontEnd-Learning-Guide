# 学习前知识

远程仓库-本地仓库-工作区
                -缓存区
三种状态： 未修改 unmodified, 已修改 modified, 已暂存 staged
通过 git status 查看分支状态

## 起步
git init 初始化本地仓库
git clone xxxx 克隆到本地

## 分支
git branch    查看本地分支
git branch -r 查看远程分支
git branch -a 查看本地和远程分支
git checkout branchName 切换本地分支
git checkout -b branchName 创建本地分支