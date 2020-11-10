# git Branch 'master' set up to track remote branch 'master' from 'origin'.   





## 正常情况下（各阶段状态）

1. 说这个之前，先说一下Git的几个区：

   **工作区**：也就是本地文件的区域

   **版本库中暂存区**：就是使用**git add命令之后**，本地工作区的文件加到暂存区

   **版本库当前分支**：也就是使用 **git commit 之后**，暂存区的东西到版本库当前分支。

   而这里出现这个错误的原因就是：暂存区没东西或者东西都提交到版本库当前分支。且工作区中的文件都被git跟踪了（即为都git add了）

   ![Git解决nothing to commit,working tree clean](https://exp-picture.cdn.bcebos.com/32fe25ef354f50b80496646fdc4afa32929c183d.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)

2. 下面给大家演示一下每个阶段的状态。

   第一：创建git版本库，但是目录没有文件。

   就会提示**nothing to commit (create/copy files and use "git add" to track)，**就是**不能提交，希望你复制或新建文件，并且使用add命令**

   ![Git解决nothing to commit,working tree clean](https://exp-picture.cdn.bcebos.com/fb738d9c2cf7dfb26bb10298d01b1edef5dc133d.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)

3. 第二：工作区有文件，但是没有进行add命令。

   就会提示：**nothing added to commit but untracked files present (use "git add" to track)** 

   含义就是**不能提交，但是有没被git跟踪的文件存在**（就是没有进行add命令）,希望你使用add命令。

   ![Git解决nothing to commit,working tree clean](https://exp-picture.cdn.bcebos.com/a1780d1fceecd3d9a9dd3e70679959430501083d.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)

4. 三：使用了add命令之后

   提示**Changes to be committed:  (use "git rm --cached..." to unstage)**。

   即为**缓存区有东西能提交，并提示你可以使用git rm -- cached 命令**将暂存区中的文件删除（不影响本地）

   ![Git解决nothing to commit,working tree clean](https://exp-picture.cdn.bcebos.com/059057299a883913244d205a26bcbe2f46707c3d.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)

5. 第四种：也就是文章标题的这种，**不能提交且工作数里面也是空的**。**nothing to commit, working tree clean** 。

   所以出现这种问题，首先应该想一下自己是不是已经提交过一次了

   ![Git解决nothing to commit,working tree clean](https://exp-picture.cdn.bcebos.com/4e168d5653bbf820be535abfba21056105a36e3d.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)





## Git大小写忽略导致

1. 还有一种情况就是，**我修改了文件，但是我没有改内容，只是改变了大小写**，但是**git设置了忽略大小写**导致git判断我没有更改，从而不能commit。

   第一步，创建git仓库，文件readme.txt添加并提交

   ![Git解决nothing to commit,working tree clean](https://exp-picture.cdn.bcebos.com/586bfdefe0781431391efed8dc6699cf0353623d.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)

2. 然后修改文件名为**Readme.txt**，再次添加并提交**出现**这个错误**nothing to commit,working tree clean**。所以这里是有问题的，应该是能提交的

   ![Git解决nothing to commit,working tree clean](https://exp-picture.cdn.bcebos.com/22c4fe36e29147e8c348f5c6b603bbea3f86583d.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)

3. 最后通过修改当前git项目取消忽略大小写的设置。**git config core.ignorecase false**，然后再添加并提交就可以了

   ![Git解决nothing to commit,working tree clean](https://exp-picture.cdn.bcebos.com/31097f43d7d44831fd433a13d40f822b75ee513d.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)

4. 上面的**git config core.ignorecase false**是修改当前的项目设置为不忽略大小写，**git config --global core.ignorecase false**设置**全局都**不忽略大小写

   