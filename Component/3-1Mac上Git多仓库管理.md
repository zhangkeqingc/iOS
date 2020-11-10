# Mac上Git多仓库管理（GitHub上有多个账号多个仓库如何配置ssh）

[Mac github 的使用](https://www.jianshu.com/p/7edb6b838a2e)

[关于Git基础](https://www.cnblogs.com/wood-life/p/10342964.html)

###  Git多仓库管理

创建新的SSH密钥，并添加到ssh-agent

创建密钥 [注解](https://blog.csdn.net/weixin_33775582/article/details/93798019)

```
ssh-keygen -t rsa -b 4096 -C "[your_email@example.com](mailto:your_email@example.com)"
```

输入保存密钥的绝对路径和文件名，如/Users/Steve/.ssh/new_id_rsa 
Enter a file in which to save the key (/Users/you/.ssh/id_rsa): [Press enter]

两次输入确认密钥的密码 
Enter passphrase (empty for no passphrase): [Type a passphrase] 
Enter same passphrase again: [Type passphrase again]

把密钥添加到ssh-agent
把ssh-agent在后台启动
eval "$(ssh-agent -s)"

一个账号不用配置config 
配置~/.ssh/config文件，如果没有该文件，通过touch config命令创建。注意HostName github.com
Host github.com
 HostName github.com
 AddKeysToAgent yes
 UseKeychain yes
 IdentityFile ~/.ssh/id_rsa

把私钥添加到ssh-agent并存入keychain，执行命令会要求你输入密码
ssh-add -K ~/.ssh/id_rsa

### 添加SSH公钥到你的GitHub账户

把公钥复制到剪贴板
pbcopy < ~/.ssh/id_rsa.pub

进入Github账户找到Settings，点击进入后选择SSH and GPG keys，点击New SSH key。在Title框内填入标题，在``框粘贴刚才复制的公钥。最后点击Add SSH key。

测试命令
ssh -vT [git@github.com](mailto:git@github.com)

### 如果有两个GitHub账户，如何配置SSH密钥并使用？

假设现在有两个GitHub账户，对应两个SSH密钥old_id_rsa和new_id_rsa。如果还没有密钥，分别按上面的步骤创建和添加。

修改config文件如下
\# 配置多个id_rsa 

\# github
Host github.com *# Host是别名*
HostName github.com *# HostName是远程仓库的域名*
IdentityFile ~/.ssh/github_rsa
AddKeysToAgent yes
\# PreferredAuthentications publickey
User yourname@gmail.com



\# 公司项目
Host labs.oa.com
HostName labs.oa.com
IdentityFile ~/.ssh/id_rsa
AddKeysToAgent yes
User yourname@qq.com



\# fastlane打包证书
Host app.cert.com
HostName labs.oa.com
IdentityFile ~/.ssh/app_rsa
AddKeysToAgent yes
User yourname@163.com

 

\# gitlab仓库备份
Host gitlab.com
HostName gitlab.com
IdentityFile ~/.ssh/gitlab_rsa
AddKeysToAgent yes
User yourname@live.com

 

注意：labs.oa.com.app 和 labs.oa.com.cert 在新版本macOS中添加秘钥时会全部识别为labs.oa.com，建议让HostName完全不同

配置完~/.ssh/config后添加秘钥到ssh-agent中，可以不用再输入密码

ps -ef|grep ssh 查找已经启动的ssh-agent进程，如果进程存在则干掉 kill -9 pid 

删除~/.ssh/known_hosts文件，rm -rf ~/.ssh/known_hosts

依次测试能否连通，ssh -T 会自动将秘钥添加到ssh-agent中

ssh -T github.com 

ssh -T gitlab.com

ssh -T labs.oa.com

ssh -T app.cert.com 

 

如果报错了，可以使用下面命令查看详细的报错信息

ssh -vT [git@new.github.com](mailto:git@new.github.com)
 

使用需要注意，git@后要改为对应账户的别名。
 如new_id_rsa密钥对应的GitHub账户上有个仓库test.git，且你的GitHub用户名是username，使用下面命令克隆

git clone [git@app.cert.com:username/test.git](mailto:git@new.github.com:username/test.git) 设置了别名就不能用原来域名labs.oa.com

避免git错用密钥，把git全局的用户名和邮箱删除

git config --global --unset user.email
git config --global --unset user.name
 

删除后，以后进入每个仓库都要指定该仓库局部的user.mail和user.name。

git config user.email "[you@example.com](mailto:you@example.com)"
git config user.name "Your Name"
 

 

还没完~

如果想要每次启动电脑都自动启动ssh-agent，就不用输入密码了 

自动操作神器登场~~~~~~

![img](https://img2018.cnblogs.com/blog/1571670/201903/1571670-20190308162450513-1428229366.png)

1、新建应用程序

![img](https://img2018.cnblogs.com/blog/1571670/201903/1571670-20190308162536577-1499258399.png)

2、选择 shell 脚本类型

 ![img](https://img2018.cnblogs.com/blog/1571670/201903/1571670-20190308162615706-1061001217.png)

3、记得先在控制台执行一下这些脚本，因为需要输入密码，否则会报错

![img](https://img2018.cnblogs.com/blog/1571670/201903/1571670-20190308163004795-1544397145.png)

 点击右上角的运行按钮执行脚本保证正确性

![img](https://img2018.cnblogs.com/blog/1571670/201903/1571670-20190308163133518-1081190280.png)

4、保存为应用程序

![img](https://img2018.cnblogs.com/blog/1571670/201903/1571670-20190308163242618-279097261.png)

5、设置开机启动运行脚本 

系统-偏好设置-用于与群组，点击 + 添加应用程序

![img](https://img2018.cnblogs.com/blog/1571670/201903/1571670-20190308163408222-2082816026.png) 

完美~

 

## 参考

 [Connecting to GitHub with SSH
](https://link.jianshu.com/?t=https%3A%2F%2Fhelp.github.com%2Farticles%2Fconnecting-to-github-with-ssh%2F) [Generating a new SSH key and adding it to the ssh-agent
](https://link.jianshu.com/?t=https%3A%2F%2Fhelp.github.com%2Farticles%2Fgenerating-a-new-ssh-key-and-adding-it-to-the-ssh-agent%2F) [Adding a new SSH key to your GitHub account
](https://link.jianshu.com/?t=https%3A%2F%2Fhelp.github.com%2Farticles%2Fadding-a-new-ssh-key-to-your-github-account%2F) [Multiple GitHub Accounts & SSH Config](https://link.jianshu.com/?t=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F3225862%2Fmultiple-github-accounts-ssh-config)