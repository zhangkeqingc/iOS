# Mac通过Terminal来使用Github



### 1. GitHub的注册

不在赘述



### 2. 安装或更新Git

```php
//查看版本
git --version
  
//显示当前的Git的配置
git config --list
  
//第一次提交需要添加邮箱账号与用户名
git config [--global] user.name "你的名字"
git config [--global] user.email "你的邮箱"
```



### 3. SSH配置

Terminal与GitHub之间使用SSH通信协议进行数据传输，想要将数据传输到github的时候，需要对方信任电脑. 配置流程如下：

1. 检查本机的ssh是否存在，在Terminal中输入命令 `cd ~/.ssh`,如果存在会检索出有哪些，可以将这些ssh备份，或者将新建的ssh生成到另外的目录下.
2. 通过默认参数生成ssh `ssh-keygen -t rsa -C xxxx@gmail.com(注册GitHub时的email)`
3. 在GitHub中选择Settings->SSH and GPG Keys 输入Title和Key,Key中存放生成的id_rsa.pub文件中的内容.
4. 打开Terminal，输入命令 `ssh -T git@github.com` ,测试账号跟GitHub的连接.



> Hi ynchai! You've successfully authenticated, but GitHub does not provide shell 
>
> access



### 4. Git的初步使用

- ##### 1.在GitHub中创建新的Repositor，这一步不具体描述了

- ##### 2.创建本地仓库，并和远程仓库进行连接，主要步骤如下：

```csharp
      cd ...           //到仓库文件夹下
      git init         //初始化本地仓库
      git add <File>   //添加某一个文件
      git add *        //添加文件夹下面的所有文件
      git status       //检查状态 如果都是绿的 证明成功
      git commit -m "commit的相关描述"  //提交到本地仓库，并写一些注释
      git remote add origin 'SSH Key' //连接远程仓库
      git pull origin master --allow-unrelated-histories //合并一些远程仓库的内容
      git push -u origin master  //将本地仓库的东西提交到origin的master分支下面
```

- ##### 3. .gitignore的创建和使用

  - .gitignore文件比较重要，一般在进行项目第一次push之前进行创建和使用
  - 在本地仓库的文件夹下面 `touch .gitignore` 创建`.gitignore`文件,然后`open .gitignore`，写入忽略目录.
  - 具体忽略目录配置请参考 [github/gitignore](https://github.com/github/gitignore) ，也可以在 [gitignore.io](https://www.gitignore.io) 输入你需要配置的语言，生成配置.
  - 如果设置 `.gitignore`之前已经进行了push，可以通过以下命令，在提交:

  ```csharp
    git rm -r --cached .
    git add .
    git commit -m 'update .gitignore'
  ```

  - ignore相关语法

  ```bash
    # 此为注释 – 将被 Git 忽略
    *.a       # 忽略所有 .a 结尾的文件
    !lib.a    # 但 lib.a 除外
    /TODO     # 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
    build/    # 忽略 build/ 目录下的所有文件
    doc/*.txt # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
  ```



### 5. Git branch 相关操作

```csharp
# 本地分支的相关操作
# 建立一个本地自己的分支
git branch mybranch  //mybranch自己分支的名字
# 查看本地有多少分支
git branch
# 切换到新建的分支
git checkout mybranch  //此时进行修改只是在mybranch上，不会影响master,此时可以进行修改、add、commit操作
# 切换回原来的分支
git checkout master
# 将之前在mybranch上的修改合并到master
git merge mybranch
# 删除无用的mybranch分支
git branch -d mybranch
# 将本地master的修改push到远程
git push 
## 新建和切换branch一次完成
git checkout -b branchname
```



```csharp
# 远程分支的相关操作
# 查看远程分支
git branch -r
# 现在本地创建一个远程没有的分支
git checkout -b branchname
# 将本地分支链接到远程分支
git push origin branchname//将本地分支链接到远程，同时在远程上也创建同一分支,origin远程仓库名字, branchname本地创建的分支名字
# 将修改后的branchname push到远程分支branchname
git push --set-upstream origin branchname
# branchname代码没有问题，合并到主分支
git checkout master
git merge branchname
git push
# 删除远程分支
git push origin :branchname
## 1.7版本之后，也可以使用以下语句
git push origin --delete branchname
# 删除本地分支
git branch -d branchname
```

### 6. Git撤销git add 文件 、撤销 commit 文件

- 撤销 git add 文件

```csharp
# 将修改提交至stage区
git add xx.file
# 重置HEAD指针，xx.file 变为 unstaged，git diff和git status又可以查看
git reset HEAD xx.file
# 修改错误后撤销修改
git checkout --xx.file
```



- 撤销 git commit 文件

```csharp
# 将修改或者添加从stage提交到本地仓库
git commit -m "xxxxx"
# 把commit文件的状态更改为staging
git reset --soft HEAD^ //之后再用git status，提示有修改的内容可以commit
# 重新修改commit
git commit --amend -m "new message" //git log可以看到这条commit记录，之前的那条没有了，如果只是想修改commit的message，不用中间更改状态的直接用这一句就可以，但是如果有别的修改，就需要有中间那一句
# 撤销上一次的commit和all changes
git reset --hard HEAD^
# 撤销上二次的commit和all changes
git reset --hard HEAD^^
```



### 7. Git tag标签相关操作

> tag是对历史一个提交id的引用，如果理解这句话就明白了. 使用git checkout tag即可切换到指定tag，例如：git checkout v0.1.0切换到tag历史记录会处在分离头指针状态，这个是的修改是很危险的，在切换回主线时如果没有合并，之前的修改提交基本都会丢失，如果需要修改可以尝试git checkout -b branch tag创建一个基于指定tag的分支，例如：git checkout -b tset v0.1.0  这个时候就会在分支上进行开发，之后可以切换到主线合并

```csharp
# 打标签
git tag -a v1.0.1 -m "Release version 1.0.1"  //git tag 是打标签命令, -a是添加标签,其后跟上新标签号, -m后面是相关描述
# 提交标签到远程仓库, git push不会将标签传送到远端服务器上
## 提交单个标签
git push origin [tagname]
## 提交所有标签
git push [origin] --tags
### 如果以上命令不起作用，请在Git控制台上确认你的账号是否有权限推送Tag
# 删除标签
git tag -d v1.0.1 //-d表示删除，后面跟上要删除标签的名字
# 删除远程标签
git push origin  :refs/tags/v1.0.1
# 查看标签
git tag / git tag -l
# 获取tag对应的代码,先clone整个仓库,然后
git checkout tagname
# 获取tag对应的代码，并在这个基础上进行修改
git checkout -b branch_name tag_name  //从tag创建一个分支，后续操作和普通的git操作一样
```



### 8. Git常用命令

```csharp
git init  //git 初始化(进入本地目录以后)
git remote add origin url  //连接远程仓库origin(url)
git diff  //看看有什么修改的,检测unstaged文件的差异
git diff --staged  //查看staged文件的差异
git status  //检查状态
git add .  //本目录下所有修改和新的文件添加至Stage(暂存区)
git add *  //同上面的注释
git commit -m "message"  //将更新和修改从Stage提交到本地仓库
git pull origin master  //远程仓库相对本地的更新
git push origin master  //将本地仓库的修改提交至远程仓库
git rm "文件"  //删除文件
git clone url  //下载工程 url 是远程的url
git log  //获取版本记录状态
```



附上一张 [w3c](https://www.w3cschool.cn/git/git-cheat-sheet.html) 的git常用命令速查表：

![img](https:////upload-images.jianshu.io/upload_images/1417873-31f229ae41b309b2.jpg?)



#### 参考网址

1. [在Mac（OS X）中使用GitHub的超详细攻略（20170706）](https://blog.csdn.net/baimafujinji/article/details/74533992)
2. [Mac下使用GitHub](https://www.jianshu.com/p/3012896932a0)
3. [Mac下使用github](https://blog.csdn.net/u012716788/article/details/51202944)
4. [MAC下GitHub命令操作](https://www.cnblogs.com/snowlove/p/6095673.html)
5. [关于使用终端terminal对GitHub项目进行管理](https://blog.csdn.net/u013664733/article/details/67633301)
6. [mac下使用git向GitHub提交项目代码](https://www.jianshu.com/p/5a1d9a248ea4)
7. [Git的一些常用命令，及.gitignore的配置](https://blog.csdn.net/zxyudia/article/details/67633321)
8. [Git分支与多人协作-Git使用总结!](https://blog.csdn.net/kkkkkxiaofei/article/details/41483039)
9. [Git查看、删除、重命名远程分支和tag](https://blog.zengrong.net/post/1746.html)
10. [git reset soft,hard,mixed之区别深解](https://www.cnblogs.com/kidsitcn/p/4513297.html)
11. [Git中tag的用法](https://blog.csdn.net/jeffasd/article/details/49863335)
12. [Git教程-分支和tag管理](https://blog.csdn.net/top_code/article/details/52336221)
13. [git学习--GitHub上如何进行PR(Pull Request)操作](https://blog.csdn.net/qq_33429968/article/details/62219783)
14. [Pull Request的正确打开方式（如何在GitHub上贡献开源项目）](https://blog.csdn.net/zhangdaiscott/article/details/17438153)
15. [Which remote URL should I use?](https://help.github.com/articles/which-remote-url-should-i-use/)
16. [真正理解 git fetch, git pull 以及 FETCH_HEAD](https://www.cnblogs.com/ToDoToTry/p/4095626.html)



作者：ynchai
链接：https://www.jianshu.com/p/2a8559bffc1a
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。