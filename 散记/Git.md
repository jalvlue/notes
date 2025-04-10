# git config
- Git 的配置分为
	- 系统级 `etc/gitconfig`
	- 用户级 `~/gitconfig`
	- 仓库级 `gitrepo/.git/config`
- 配置有层级继承关系, 如果本级没有定义, 那么就会使用上级配置
- 如果本级有配置, 那么就会覆盖上级配置

# git commit -m "commit message"
Git 使用一个有向无环图组织版本信息, 每一次提交 (commit)都会创建一个新的节点, 每个节点代表一个版本


# git branch [branch-name]
git 的分支非常轻量, 只是简单地指向某一个提交,
使用分支相当于基于当前提交以及它所有的 parent 提交进行新的工作


# git checkout [branch-name]
用于切换当前所在分支, 使用 git 进行工作时, 都会处于一个分支中, 使用 checkout 指令可以在分支之间切换
快捷创建新分支并切换到该分支: `git checkout -b [branch-name]`


# git merge [branch-name]
将当前分支与指令中的分支进行合并, 合并会创造一个特殊的提交, 并将当前分支移动到指向这个提交


# git rebase [branch-name]
将当前分支与指令中的分支进行合并, 但 rebase 合并之后创造的提交基于[branch-name], 因此会产生更线性的分支
![[Pasted image 20230924113301.png]]
git rebase main
![[Pasted image 20230924113326.png]]
切换到 main 分支, 执行 git rebase bugFix 会将 main 与 bugFix 合并
![[Pasted image 20230924113432.png]]
此时, 所有分支都统一了

# HEAD
在 git 中, HEAD 是一个特殊的"分支", 它会默认指向某个分支, 并随着分支的提交移动, HEAD 也可以从分支中分离出来, 指向某个提交
git checkout hash
git checkout hash^ 表示到这个提交的父节点
git checkout hash~[num] 表示到这个提交的 num 级父节点

# git branch -f [branch-name] [commit]
分离 HEAD 的作用就是让分支自由地在提交之间移动
使用这个指令可以将分支 branch-name 换到该提交上, 这个提交可以直接使用 hash, 也可以使用 HEAD 

# git reset & git revert
git reset hash 适用于本地提交的撤销
git reset HEAD 用于远程分支中撤销上一次提交
