# git分支
* 查看本地分支  `git branch` 
* 查看远程分支 `git branch -v`
* 查看所有分支 `git branch -a`
* 本地创建分支，并关联远程分支，并切换到本地分支 `git checkout -b dev origin/dev`
* 创建+切换分支 `git branch -b dev`


# git还原
* 还原至某个版本 `git reset ”commitId“`

# git提交
  git push <远程主机名> <本地分支名> : <远程分支名>
  以下是一些示例
  > git push origin master
  上面的命令表示，将本地master分支推送到orgin 主机的master分支。如果master不存在，则会被新创建
  
* git 提交当前分支，推到code review `git push origin HEAD:/refs/for/master`
