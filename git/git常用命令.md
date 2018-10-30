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
  
  以下是一些示例:  
  1. `git push origin master`  
      上面的命令表示，将本地master分支推送到orgin 主机的master分支。如果master不存在，则会被新创建
  
  2. `git push origin HEAD:/refs/for/master`  
      上面的命令表示，将当前的分支提交到远程，一般用作code review   
  3. `git push origin : master` 等同于`git push origin --delete master`  
      上面的命令表示，删除origin主机上的master分支  
  4. `git push origin`  
      将当前分支推送到origin主机的对应分支。如果当前分支只有一个追踪分支，那么主机名可以省略  
  5.  `git push`  
       将当前的分支推送origin主机的对应分支，只不过origin省略了  
  6. `git push -u origin master`  
      上面命令表示将本地的master分支推送到origin主机，同时指定origin作为默认主机，后面就可以不加任何参数使用git push了
      
