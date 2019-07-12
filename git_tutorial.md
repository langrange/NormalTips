## git回退大致分如下三条：
+ HEAD指向的版本就是当前版本
+ 使用`git reset --hard commit_id`定位到需求版本
+ `git log`查看提交历史记录
+ 要穿梭到未来，先`git reflog`查看历史命令
