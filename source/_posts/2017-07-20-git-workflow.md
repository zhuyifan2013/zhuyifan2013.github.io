title: Git Workflow 和分支规范
date: 2017-07-20 21:53:53
tags:
- git
---

## 使用分支说明

`master`**[Must be Protected]** : master 上的每一个 commit 都意味着一次版本的发布，所以普通的 commit 不要提交在这个分支上

`dev` **[Could be Protected]** : 主开发分支，开发过程(非 release 期间)中的 commit 都会提交在 dev 分支上
`feature branch` : 自己开发过程中使用的分支，一般来说在本地开发完成后都会新建一个远程分支来承载此次修改，review 通过后会 merge 到 dev 分支上
`release branch` **[Must be Protected]**: release 期间使用的分支。当产品开发到一定阶段需要发布的时，新建一个基于 dev 的分支，一般命名为  release-x.x.x , 此时进入 release 阶段，**release 期间在 release 分支上的修改不可以是新的 feature**，只能是 bug-fix，新的 feature 只能提交到 dev 分支上，在下次 release 发布，当 bug 修复完毕后，release 分支的代码会**同时 merge 到 master 和 dev 上**，往 dev 上 merge 时建议保留每次 PR 的信息 (git merge --no-ff) ，因为回顾的时候可以看到每次 commit 的信息，而 往 master 上 merge 的时候建议直接合成一个 commit (git merge) , 因为 master 上不需要保留每次 commit 的信息。release 分支使用后删除。
`hot fix branch` : 在 从 release 往 master 上 merge 并发布之后，有严重的 bug 时，需要新建 hot-fix 分支修复问题，最终需要**同时 merge 进 dev 和 master**，使用后删除。

## 工作流程
简单来说，工作流程就是，基于 dev 分支新建 feature 分支，开发完成后 merge 进 dev，到达发布时间之前拉出 release 分支交付测试，测试过程中发现 bug，基于 release 分支新建 bug-fix 分支修复，然后 merge 进 release，稳定后将 release 分支 merge 进 dev 和 master ，然后删除 release。

其中，dev 和 master 分支是一直存在的，其他分支都是按需新建，使用完成后删除。

![git workflow](/img/2017-07-20/git-workflow.png)

## PR(Pull Request) 提交规范
如果使用 Github 之类的代码管理库，建议提交 PR 的时候遵循一定的规范，既方便别人review，也方便团队成员回顾。
每个 PR 的标题要使用以下的 Tag 作为开头：

 - `[Optimize]` 优化代码；去除无用代码；重新组织文档结构等等
 - `[Bug-Fix]` 修改 BUG
 - `[Hot-Fix]` 修复在 Release 之后的重要 BUG，需要同时往 master 分支合并的修复
 - `[iOS]` / `[Android]` 该 PR 与特定平台相关，若都影响则不指定该 Tag
 - `[Feature]` 功能性的代码提交，需要注意控制每个 PR 的大小，在开始做一个模块的时候就要规划好功能的开发递进，避免一次性提交大量代码。

 ## 分支与产品迭代
 以我们的产品开发为例，正常迭代周期为2周一个版本，可以安排在第一周周五从 dev 拉一个 release 的分支，并基于此 release 分支打包测试，测试出来的问题在下周二之前解决完毕，根据解决快慢，可以在剩下的时间里继续新功能的开发，并合入 dev 分支，第二周周四将 release 分支合入 master 并发布正式版本。
 有重要的 Bug 需要第一时间解决(Hot-Fix)，合入 master 与 dev。

![product workflow](/img/2017-07-20/product-workflow.png)





