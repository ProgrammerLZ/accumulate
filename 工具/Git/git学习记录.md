## 最佳实践

<img src="git%E5%AD%A6%E4%B9%A0%E8%AE%B0%E5%BD%95/webp" alt="img"  />

## Tips

1. 在A分支修改了代码，这个时候有其他紧急任务插入进来因此新建了B分支，然而B分支也能看到A分支修改的代码，这是为什么？

   答：这种情况，需要把A分支的代码暂存一下，然后在切换到其他分支，就没有上述的情况了。在B分支上干完活之后，再切回到A分支可以 把之前暂存的代码再取出来。

2. 我每次提交merge request之前，都要把dev的代码合并到我自己的分支上来吗？

   答：是的。如果不合并，提交merge request，管理dev分支的人在合并你的代码的时候，就有可能产生冲突。先合并，再提交merge request就不会有这种情况发生。