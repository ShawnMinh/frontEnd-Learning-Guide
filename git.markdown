# 学习前知识

远程仓库-本地仓库-工作区</br>
                -缓存区</br>
三种状态：</br> 未修改 unmodified, 已修改 modified, 已暂存 staged</br>
通过 git status 查看分支状态

## 起步
git init 初始化本地仓库</br>
git clone xxxx 克隆到本地 

## 分支
git branch    查看本地分支</br>
git branch -r 查看远程分支</br>
git branch -a 查看本地和远程分支</br> 
git branch -m oldname newname 重命名分支</br> 
git branch -d name 删除本地分支 *删除前先切换到其他分支</br> 
git checkout name 切换本地分支 </br>
git checkout -b name 创建本地分支 </br>
git push origin name 远程仓库添加分支 </br>
git merge name 将分支合并到本分支 </br>
 