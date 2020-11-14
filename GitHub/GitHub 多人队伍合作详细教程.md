# GitHub 多人队伍合作详细教程



### 文章目录

- - [场景一：队员与主仓版本保持同步](https://blog.csdn.net/sculpta/article/details/104448310#_45)
  - [场景二：队员与主仓版本合并](https://blog.csdn.net/sculpta/article/details/104448310#_116)
  - [场景三：合并时出现冲突](https://blog.csdn.net/sculpta/article/details/104448310#_166)
  - [场景四：版本回退](https://blog.csdn.net/sculpta/article/details/104448310#_216)



![img](https://gitee.com/sculpta/data/raw/master/blog/github-cooperation/1.jpg)

下面以一个二人队伍为例（队长 A 和队员 B），多人队伍的话其他队员操作同队员 B

以队长 A 的仓库为基准，首先队员 B Fork 队长 A 的项目仓库，队长 A 和队员 B 的仓库结构如下所示：

```
队长 A：

 ┌── master ：最终的可上线版本分支
 │
 └── dev ：队长 A 本人开发分支

队员 B：

 ┌── master ：队员 B 经过开发、具有一部分新功能的分支，用于向队长 A 提出 PR
 │
 └── dev ：队员 B 本人开发分支 
1234567891011
```

> 其中，队长 A 的 master 分支是经过充分测试、最终可以部署上线的版本分支，队员 B 的 master 分支是其本人经过开发，完成了一个子功能或子模块并经过简单测试之后，用于向队长 A 提出 PR（Pull Resquest） 的分支

队伍成员职责如下：

- 队长 A：除分配开发任务、负责开发工作外，更重要的工作是 **检查队内成员开发成果，解决合并时的错误，完成项目上线等**
- 队员 B：主要负责开发工作

假设仓库初始状态为：

```
队长 A（主仓）：

 ── master ── project.txt（内容为 "module 1"）

队员 B（Fork 主仓）：

 ── master ── project.txt（内容为 "module 1"）
1234567
```

## 场景一：队员与主仓版本保持同步

假设队长 A 经过一轮合并、添加了子模块，并将项目上线为版本 2，此时主仓内容为：

```
── master ── project.txt（内容为 "module 1, module 2"）
1
```

由于是以队长 A 的仓库为主仓，所以队员在开发新的功能时，须先拉取当前最新版本，以便保持同步

队员 B clone 自己的远程仓库至本地

```shell
git clone https://github.com/member_b/project.git
1
```

添加主仓的远程仓库地址

```shell
git remote add upstream https://github.com/captain/project.git
1
```

查看当前仓库所连接的远程仓库

```shell
git remote -v
1
```

此时，当前仓库所连接的远程仓库为：

```
origin    https://github.com/member_b/project.git (fetch)
origin    https://github.com/member_b/project.git (push)
upstream    https://github.com/captain/project.git (fetch)
upstream    https://github.com/captain/project.git (push)
1234
```

拉取主仓文件

```shell
git fetch upstream
1
```

切换到 master 分支

```shell
git checkout master
1
```

合并

```shell
git merge upstream/master
1
```

至此，新版本的项目文件已经更新到了 master 分支，下一步需要将它合并到 dev 分支，以便后续在 dev 分支上开发

如未创建 dev 分支，则创建并切换

```shell
git checkout -b dev
1
```

合并

```shell
git merge master
1
```

队员 B 就可以在 dev 分支下工作了

## 场景二：队员与主仓版本合并

当队员 B 完成了新模块并经过测试 push 到远程仓库的 master 分支后，需向队长 A 提出合并请求，即 Pull Resquest

假设队员 B 向 `project.txt` 中添加了 “module 3”

队员 B 在自己的 GitHub 仓库中新建 Pull Resquest，如图

![img](https://gitee.com/sculpta/data/raw/master/blog/github-cooperation/2.png)

选择分支，这里选择向主仓的 dev 分支提出 PR

![img](https://gitee.com/sculpta/data/raw/master/blog/github-cooperation/3.png)

填写相关信息后，点击「Create Pull Resquest」，队员 B 任务完成

![img](https://gitee.com/sculpta/data/raw/master/blog/github-cooperation/4.png)

队长 A 在主仓中查看 Pull Resquest，`Commits` 标签下显示队员 B 的提交记录，`Files changed` 标签下显示文件内容变动情况

![img](https://gitee.com/sculpta/data/raw/master/blog/github-cooperation/5.png)

经过检查后就可以选择合并或拒绝合并（即 Close PR）

![img](https://gitee.com/sculpta/data/raw/master/blog/github-cooperation/6.png)

其中，Merge Pull Resquest 有三个选项

- `Create a merge commit` ：表示把这个 PR 作为一个分支合并，并保留分支上的所有提交记录
- `Squash and merge` ：表示只为这次合并保留一个提交记录
- `Rebase and merge` ：找到两个分支共同的祖先，然后在当前分支上合并从共同祖先到现在的所有 commit

对比

- `Create a merge commit` ：不能保持 master 分支干净，但是保留了所有的 commit history，当 PR 中 commit 次数较多时不推荐此方式
- `Squash and merge` ：也可以保持 master 分支干净，但是 master 中 author 都是 maintainer，而不是原 author
- `Rebase and merge` ：可以尽可能保持 master 分支干净整洁，并且易于识别 author

此时，队长 A 的 dev 分支就加入了队员 B 的新模块。随着不断向 dev 分支合并，等到完成一个可上线版本时，队长 A 就可以将 dev 分支合并到 master 分支部署上线了

## 场景三：合并时出现冲突

出现冲突的情况有很多，比如队员 B 偷懒好几天没工作，当他拉取主仓最新版本文件到 master 分支后，发现其他队友们修改了内容并替他修复了 bug，导致队员 B 在合并时产生冲突

对于合并时产生的冲突，需要手动修改、消除冲突

假设仓库状态如下：

```
队长 A（主仓）：

 ┌── master ── project.txt（内容为 "module 1, module 2.5, module 3"）
 │
 └── dev ── project.txt（内容为 "module 1, module 2.5, module 3"）

队员 B（fetch upstream 前）：

 ┌── master ── project.txt（内容为 "module 1, module 2, module 3"）
 │
 └── dev  ── project.txt（内容为 "module 1, module 2, module 3"）
1234567891011
```

当队员 B fetch 后尝试与 master 合并时，产生冲突

![img](https://gitee.com/sculpta/data/raw/master/blog/github-cooperation/7.png)

使用 VS Code 查看 `project.txt`

![img](https://gitee.com/sculpta/data/raw/master/blog/github-cooperation/8.png)

`=======` 上方的是当前分支的内容，下方的是进行合并的分支要传入的内容。此时，须以主仓版本为准，所以需要手动删除其他内容，或点击「采用传入的更改」，修改为：

```
module 1, module 2.5, module 3
1
```

之后，还需要

```shell
git add project.txt

git commit -m "fix conflict"
123
```

这时就消除了冲突，再与 dev 分支合并就可以进行后续工作了

## 场景四：版本回退

有时候当前版本可能有许多 bug，不得不重新从头开始工作，这时候就需要先回退到上一版本

`git reset` 命令可以将当前 branch 回退到之前某一个 commit

回退到上一版本

```shell
git reset --hard HEAD^
1
```

其中，`HEAD^` 表示上一个版本，`HEAD^^` 表示上上个版本，`HEAD~100` 表示前 100 个版本

另外，还可以使用以下命令

```shell
git reset --hard 版本号
1
```

进入需要回退的分支，在日志中查看版本号

```shell
git log --oneline
1
```

**注意**：回退版本后，日志中就会删除回退前版本的记录，如果想撤回回退操作，可以使用如下命令查看版本号

```shell
git reflog
```



原文：[一路是夜幕沉沙](https://me.csdn.net/sculpta) 