### git部分常用命令
      git init 
      git remote add origin [远程仓库]
      git add *
      git commit -m "message"
      git push [远程仓库] [分支名]
      git pull [远程仓库] [分支名]

      git checkout -b [分支名]  //创建分支切换过去
      git checkout master //切换主分支
      git branch -d [分支名] //删除分支
      git push origin [分支名] // 推送分支到远程
      git merge [分支名] //合并分支到当前分支
      git add [fliename] //冲突文件解决后标记合并成功文件
      丢失本地修改和提交，获取远程最新分支
      git fetch orign
      git reset --hard origin/master

#### 几个commit合并成一个(用户多次提交后变成一次提交记录，或者编辑修改)

      用于查询commitId
      git log 
      
      进入vi模式进行选择pick和累加squash 
      git rebase -i commintId 
      
            pick就是说保留该commit, 也可以用缩写p. (黄色)
            squash, 使用该commit但合并到前一个老的commit去(常用). 可以用缩写s代替 (绿色).
            reword, 和pick类似, 但可以修改commit时的提交信息(中间会弹出来让你修改commit).可以用缩写r代替 (紫红色).
            edit, 使用commit, 但停下来进行修改, 可能用于merge冲突.可以用缩写e代替.（当前结论 只能修改备注？无法修改提交中的某个文件）
            fixup, 和squash类似, 但会舍弃commit信息. 可以用缩写f (红色)
            exec, 执行shell命令.可以用缩写x
            drop,剔除此次commit.可以用d代替
  
      部分出现错误后的命令
      git rebase --abort来忽略之前的rebase尝试,并恢复HEAD到开始的分支.
      git rebase --continue就继续上次修改, 一般是rebase中间处理merge冲突后使用.
      git rebase --skip是重新开始rebase并跳过现在所进行的处理.

      强制上传到远程服务器

      git push -u orign master -f 
      git push -u（upsteam） [远程仓库] [本地仓库] -f(force)

#### 分支的一些命令
      git merge --abort 取消分支合并
      git reset --merge 重置分支
      git reset --mixed 默认git reset使用这个选项，回归到指定版本，当前源码不变化，源码与目标版本的差异未打入缓存区
      git reset --soft  回归到指定版本，当前源码不变化，源码与目标版本的差异打入缓存区
      git reset --hard  回归到指定版本，当前源码和缓存区都将回归到当前版本，次版本之后的commit都将消失
  
  
  
