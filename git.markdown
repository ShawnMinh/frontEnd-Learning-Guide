# 学习前知识
git bash 浏览状态 到尾部按 Q 结束</br>
远程仓库-本地仓库-工作区</br>
                -缓存区</br>
三种状态：</br> 未修改 unmodified, 已修改 modified, 已暂存 staged</br>
通过 git status 查看分支状态

![alt text](image.png)

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
git merge name   合并分支 将name分支合并到本分支 </br>
 
 ## 暂存文件

 git add .  添加所有文件到暂存区</br>

 git commit -m '备注信息'  提交文件到本地仓库</br>
 git commit --amend -m '新的备注' 修改上一次的备注</br>
 git commit --amend --no-edit  提交新的内容到上一次的提交中, 不额外新增 </br>

 git stash 暂存当前内容</br>
 git stash push -m "备注"</br>

 git stash list 暂存的列表</br>
 git stash pop 弹出当前暂存的内容, 删除暂存区内容</br>
 git stash apply 弹出暂存区内容，不删除</br>

 git merge 合并冲突</br>
 git rebase  本地变基,功能分支合并到主分支之前使用git rebase,清晰列表</br>
 ## 远程仓库
 git remote -v 查看远程仓库
 git remote add name url 添加远程仓库
 git remote remove name 删除本地的指定的远程仓库
 git remote rename oldname newname 重命名本地的远程仓库
 git fetch name 从指定的远程仓库获取最新的代码，但不会自动合并到当前分支。它只是将远程仓库的更新下载到本地仓库</br>
 git pull 从指定的远程仓库拉取代码并自动合并到当前分支。相当于先执行git fetch，然后执行 git merge </br>
 git pull <remote_name> <branch_name> </br>
 git push <remote_name> <branch_name> 推送到远程仓库

 git diff 查看本地分支和远程分支的差异，以便更好地了解即将进行的合并操作
 ## 日志
 git log  --oneline</br>
 git show --oneline </br>

## 一般流程
git init / git clone </br>
是否需要切换分支 git checkout </br>
本地工作 </br>
git pull 或 git fetch && git merge </br>
git add . </br>
git commit -m '备注'</br>
git push
