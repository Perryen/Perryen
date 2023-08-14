# git总结

在项目开发过程中，总会用到git来进行代码管理，使用github或者gitee作为远程仓库对代码进行托管。因此，熟练使用git工具和GitHub平台会大大提升项目协同开发效率，对于个人项目的高效管理也有很大好处。那么我们就要捋顺整体代码管理逻辑，理解这样做的目的，而非仅仅停留在clone、push和pull上面。

Git是一个版本控制工具，它可以记录代码内容的变更，方便我们对项目的管理，它有以下几种用途：

\*\*1、代码备份：\*\*当在一个新的程序上迭代新的功能或者改动时，代码备份就显得很重要，通过git切换分支或者版本就可以回到某种状态，而不用事先在本地对源代码进行备份，反复在新副本上修改与增添。

\*\*2、协同编程：\*\*将项目分割为多个小模块分配给不同的人来做，多个功能同时进行，不过不使用git进行版本控制，项目在统合时会出现很大的问题。而通过git可以实现代码共享，多个功能模块相互协调，同时开发，逐步实现全局开发。

***

## git安装

[git安装详细参考资料](https://blog.csdn.net/mukes/article/details/115693833)

在Windows 下可以下载最新版的git，然后具体的安装步骤参考以上链接即可；在linux下使用`sudo apt-get install git`命令进行安装即可。git在windows上和Linux上使用基本是无差别的。

Windows成功安装git后会有多个可执行文件，包括`Git Bash、Git CMD、Git FAQs、Git GUI、Git Release Note`下面介绍一下它们的功能：

### 1、git bash

基于CMD的，在CMD的基础上增添一些新的命令与功能，平时主要用这个，功能很丰富，长这样：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211132334910.png)

### 2、git cmd

不能说和 cmd 完全一样，只能说一模一样，功能少得可怜，两者如下图：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211132334528.png)

### 3、git faqs

就是 Git Frequently Asked Questions（常问问题），访问地址：https://github.com/git-for-windows/git/wiki/FAQ

### 4、git gui

就是 Git 的图形化界面，可以通过它快速`创建新仓库（项目），克隆存在的仓库（项目），打开存在的仓库（仓库）`。如下图：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211132335995.png)

### 5、git release note

就是版本说明，增加了什么功能，修复了什么 bug 之类的。

***

## git免密登录

### 1、区分https clone 和 [ssh](https://so.csdn.net/so/search?q=ssh\&spm=1001.2101.3001.7020) clone

不同的克隆方式导致校验方式不同，对应的免秘方式也不一样。 https通过记住账号密码免登，ssh通过校验生成的[密钥](https://so.csdn.net/so/search?q=%E5%AF%86%E9%92%A5\&spm=1001.2101.3001.7020)免登。 通常都用ssh校验。

### 2、https免密配置方法

设置配置 .git/config，**输入一次账号密码后第二次就会记住账号密码。**

```bash
git config --global credential.helper manager
```

### 3、credential.helper

安装 Git 后，我们可以通过下面的指令查询当前凭证存储模式

```bash
git config credential.helper
```

| 凭证存储模式        | 含义                                                                                                                                      |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| cache模式       | 会将凭证存放在内存中一段时间。 密码永远不会被存储在磁盘中，并且在15分钟后从内存中清除。                                                                                           |
| store模式       | 会将凭证用明文的形式存放在磁盘中，并且永不过期。 这意味着除非你修改了你在 Git 服务器上的密码，否则你永远不需要再次输入你的凭证信息。 这种方式的缺点是你的密码是用明文的方式存放在你的 home 目录下。                                |
| osxkeychain模式 | 如果你使用的是 Mac，Git 还有一种 “osxkeychain” 模式，它会将凭证缓存到你系统用户的钥匙串中。 这种方式将凭证存放在磁盘中，并且永不过期，但是是被加密的，这种加密方式与存放 HTTPS 凭证以及 Safari 的自动填写是相同的。           |
| manager模式     | 如果你使用的是 Windows，你可以安装一个叫做 “Git Credential Manager for Windows” 的辅助工具。 这和上面说的 “osxkeychain” 十分类似，但是是使用 Windows Credential Store 来控制敏感信息。 |

推荐使用凭证存储模式 "manager"，该模式下的用户信息展示如下：

```bash
Internet 地址或网络地址：git:https://github.com； 
用户名：PersonalAccessToken
```

手动修改凭证存储模式:

```bash
git config credential.helper 对应模式 
```

重置指令

```bash
git config --unset credential.helper
```

使用的git安装工具（最新版）在安装时会默认帮我们把credential.helper设置成manager，除非我们在安装时下面这个默认勾选的配置被手动取消了：

![install](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211142240548.png)

### 4、git-ssh免密登录

（1）检查本机是否有ssh key设置

```bash
$ cd ~/.ssh
或者
$ cd .ssh
```

如果没有则会提示：

![](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture@master/img/image-20230303154940451.png)

如果有就会进入到.ssh文件夹下，使用ls查看当前文件夹下路径，使用rm\*删除当前文件夹下所有文件。

（2）使用Git Bash生成新的ssh key

```bash
$ cd ~  #保证当前路径在”~”下

$ ssh-keygen -t rsa -C "xxxxxx@yy.com"  #建议填写自己真实有效的邮箱地址

Generating public/private rsa key pair.

Enter file in which to save the key (/c/Users/xxxx_000/.ssh/id_rsa):   #不填直接回车

Enter passphrase (empty for no passphrase):   #输入密码（可以为空）

Enter same passphrase again:   #再次确认密码（可以为空）

Your identification has been saved in /c/Users/xxxx_000/.ssh/id_rsa.   #生成的密钥

Your public key has been saved in /c/Users/xxxx_000/.ssh/id_rsa.pub.  #生成的公钥

The key fingerprint is:

e3:51:33:xx:xx:xx:xx:xxx:61:28:83:e2:81 xxxxxx@yy.com

*本机已完成ssh key设置，其存放路径为：c:/Users/xxxx_000/.ssh/下。

注释：可生成ssh key自定义名称的密钥，默认id_rsa。

$ ssh-keygen -t rsa -C "邮箱地址" -f ~/.ssh/githug_blog_keys #生成ssh key的名称为githug_blog_keys，慎用容易出现其它异常。
```

![image-20230303155746546](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture@master/img/image-20230303155746546.png)

（3）添加SSH key到github

打开id\_rsa.pub文件，复制全部内容（公钥）到github里的SSH增加处，Title自定义

（4）配置SSH登录账户

```bash
$ git config --global user.name “your_username”  #设置用户名

$ git config --global user.email “your_registered_github_Email”  #设置邮箱地址(建议用注册giuhub的邮箱)邮箱地址只要符合格式要求即可，随意设置，设置邮箱都不存在。
```

![image-20230303160835568](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture@master/img/image-20230303160835568.png)

（5）测试SSH是否配置成功

```bash
$ ssh -T git@github.com

The authenticity of host 'github.com (192.30.252.129)' can't be established.

RSA key fingerprint is 16:27:xx:xx:xx:xx:xx:4d:eb:df:a6:48.

Are you sure you want to continue connecting (yes/no)? yes #确认你是否继续联系，输入yes

Warning: Permanently added 'github.com,192.30.252.129' (RSA) to the list of known hosts.

Enter passphrase for key '/c/Users/xxxx_000/.ssh/id_rsa':  #生成ssh kye是密码为空则无此项，若设置有密码则有此项且，输入生成ssh key时设置的密码即可。

Hi xxx! You've successfully authenticated, but GitHub does not provide shell access. #出现词句话，说明设置成功。


```

![image-20230303160850928](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture@master/img/image-20230303160850928.png)

***

## git工作流程（个人开发）

![img](https://pic1.zhimg.com/80/v2-cd9ae639fa7ba273ffc3753037629ee0\_720w.webp)

* 工作区(Workspace)：平时存放项目代码的地方
* 暂存区(Index/Stage)：用于临时存放改动信息
* 本地仓库(Repository)：存放所有提交的版本数据
* 远程仓库(Remote)：托管代码的服务器，比如我们经常用的Github就是个代码托管平台

**git的基本工作流程如下：**

1. 在工作区中添加、修改文件
2. 将工作区中需要进行版本管理的文件放入暂存区
3. 将暂存区的文件提交到git本地仓库
4. (**optional**)将本地仓库推送到远程仓库

***

## git工作流程（团队开发）

![](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211141508517.png)

**注意：无论是团都内部协作开发还是远程团队协作开发，将远程代码克隆到本地后进行开发时，一般都需要创建新的分支来进行开发。除此之外，还要注意分支合并的操作。**

***

## git使用

### 1、git初始化

初始化git，有两种方式：

```bash
# 方式一:本地生成一个git
git init
# 方式二:从远端克隆一个仓库
git clone https://gitee.com/xxxxxx/xx.git
```

克隆时经常需要下载特定版本号的程序：

```bash
git clone #下载源码
git tag　#列出所有版本号
git checkout　+某版本号　
```

### 2、基本配置

配置用户名和邮箱，用户名和邮箱可以是任意的，符合格式就可以。

```bash
# 配置用户名
git config --global user.name "name"
# 配置邮箱
git config --global user.email "name@mail.com"
```

### 3、远程仓库操作

```bash
# 删除远程操作
git remote rm origin
# 添加远程操作
git remote add [shortname] [url]
# shortname通常是origin
git remote add origin https://gitee.com/xxxxxx/xx.git
-----------------------------------------其他命令 ----------------------------------------
# 显示所有远程仓库
git remote -v
# 显示某个远程仓库的信息
git remote show https://gitee.com/xxxxxx/xx.git
# 修改仓库名
git remote rename old_name new_name
```

### 4、推送至远程仓库

（1）将已修改文件添加至暂存区

```bash
git add dir/filename # 添加指定文件
git add . # 添加所有已修改文件
```

（2）将暂存区的改动提交到本地的版本库，使用`git commit`命令我们就会在本地版本库生成一个40位的哈希值，用于版本回退

```bash
git commit -m "message" # message就是本次提交的简要说明
```

（3）本地上传，注意在推送前需要先从远程拉取

```bash
git push -u origin master # master可以更换为其他分支
```

**带上-u 参数其实就相当于记录了push到远端分支的默认值，这样当下次我们还想要继续push的这个远端分支的时候推送命令就可以简写成git push即可。**

### 5、从远程仓库拉取

更新本地：

```bash
git pull origin master # master可以更换为其他分支
```

**git pull 拉取操作其实是两步：pull = fetch + merge**

*   ferch 操作: 只把远程库中的内容下载到本地，但是没有改本地工作区的文件

    ```bash
    git fetch 远程仓库地址别名 远程分支名
    ```
*   merge操作：把远程代码合并到本地代码中

    ```bash
    git merge 远程仓库地址别名/远程分支名
    ```

### 6、分支管理

分支管理是版本控制中一个很重要的内容，git所有分支之间彼此互不干扰，各自完成各自的工作和内容。可以在分支使用完后**合并到总分支(原分支)** 上，安全、便捷、不影响其他分支工作。

**以下命令是基于本地仓库的操作。**

#### **（1）查看当前分支**

```bash
# 查看当前本地仓库分支
git branch
# 返回
# * master
```

Git 的 master 分支并不是一个特殊分支。 它就跟其它分支完全没有区别。 之所以几乎每一个仓库都有 master 分支，是因为 git init 命令默认创建它，并且大多数人都懒得去改动它。

```bash
# 查看所有本地仓库分支和远程分支
git branch -al
# 或者
git branch -a
```

#### **（2）HEAD指针**

> 这里提个问题。对一个项目提交了很多个分支，如果有两个分支指向相同提交历史，Git 又是怎么知道当前在哪一个分支上呢？

很简单，它有一个名为 HEAD 的特殊指针。 请注意它和许多其它版本控制系统（如 Subversion 或 CVS）里的 HEAD 概念完全不同。 在 Git 中，它是一个指针，指向当前所在的本地分支（译注：将 HEAD 想象为当前分支的别名）。

而HEAD所指向的直接关系是当前分支，再找到分支的版本。如下图：

![image-20221114002259148](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211140022209.png)

#### **（3）创建新分支**

git创建新分支。即在当前位置创建一个指针（master本身也是一个指针），比如起名为 `dev`，然后将HEAD指向dev。如下图：

1、分支创建好后的提交都是在dev分支上提交，而之前的总提交master分支的提交位置停留在创建从分支dev的位置。而HEAD会跟随新创建的分支，跟随每一次提交不停的先前移动。保持HEAD指针的在提交的最前沿。 2、在master上新创建的dev分支会继承master分支的所有提交，通过 git log 可以看出来。

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211140028363.jpeg)

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211140027692.jpeg)

```bash
# 创建并切换到dev分支
git checkout -b dev

# git checkout命令加上-b参数表示创建并切换，相当于以下两条命令
git branch dev		#创建分支
git checkout dev	#切换分支
```

#### **（4）合并分支**

当dev分支工作完成，需要合并到master分支的时候，也只是将master指针指向当前dev的位置，并将HEAD指向master，这时dev分支可以直接删除，也可以不删除，删除的话也只是移除了dev指针，只留下一个master指针，对工作区没有任何的影响，也就是曾经做的所有提交操作都不会有影响。

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211140044419.jpeg)

提交本地分支到远程分支

```bash
git push origin dev:dev 
# 本地的空分支(冒号前面的分支),想要推到远程origin的分支名字(冒号后面的分支)
```

切回主分支

```bash
# 分支切换回主分支master
git checkout master
```

合并分支

当分支切换回主分支的时候，可以将dev的修改提交合并到master分支上，使用：

```bash
# 合并dev到master
git merge dev
```

> 这一次的合并称之为快速合并 `fast-forward`。只是将master的指针指向了dev最后一次提交的位置。

当分支切换回主分支master的时候，就可以删除本地dev分支：

```bash
# 删除远程分支demo
git push origin --delete dev 
# 也可以直接推送一个空分支到远程分支，其实就相当于删除远程分支：
git push origin :demo  //推送本地的空分支(冒号前面的分支)到远程origin的demo(冒号后面的分支)(没有会自动创建)
# 删除本地分支
git branch -d dev
```

#### （5）冲突的发生与解决

当同一个文件被两个分支都修改过，想要合并两个分支就会产生冲突，不能快速将dev合并到master上。并且git会提醒“合并过程中产生了冲突，请修正后再提交”。

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211140050422.jpeg)

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211140050141.jpeg)

修正的过程：

1. 将两个分支的文件，进行对比修改，满足两个分支的提交。
2. 使用 git add 和git commit 进行一次新的提交。(此时提交的是master分支)
3. 再次合并

```bash
解决方法：
	1.直接在编辑器打开文件修改，删除多余的符号
		或者
	2.直接在命令行输入vim demo.txt 按i 进入编辑模式 删除多余的符号
```

查看带有冲突解决的日志

```bash
git log --graph -- pretty=oneline
```

#### （6）分支管理策略

通常，合并分支时，如果没有冲突，并且分支是单向一条线路继承下来的，git会使用 fast forword 模式，但是有些快速合并不能成功，但是又没有冲突时，就会触发分支管理策略，git会自动做一次新的提交。

当两个分支对工作区都进行了修改，但是修改的并不是同一个文件，而是两个不同的文件，也就是不会产生冲突。此时合并的时候，不能使用\*\*“快速合并”\*\*，就会弹框需要你输入一段合并说明。使用快捷键 **ctrl+x** 退出

```bash
# 合并dev到master，禁止快速合并模式，同时添加说明
git merge --no-ff -m '' dev
```

#### （7）bug分支

使用场景：当在某个分支上正在工作，突然有一个紧急的bug需要修复，此时可以使用 `stash`功能，将当前正在工作的`现场存储起来`，等bug修复之后，在返回继续工作。

* **修复bug时，通过创建新的bug分支进行修复，然后合并，最后删除。**
* **当手头工作没有做完时，先把工作现场git stash一下，然后去修复bug，修复后，再git stash pop，恢复工作现场。**

操作顺序：

```bash
# 1、对当前现场进行存储
git stash
# 2、切换到bug出现的分支上，比如bug出现在 master分支
git checkout master
# 3、新添加一个bug临时分支
git checkout -b bug001
# 4、对代码进行修复
# 5、切换回master分支
git checkout master
# 6、合并bug分支到主master上
git merge --no-ff -m '合并bug分支到master' bug001
# 7、删除bug001分支
git branch -d bug001
# 8、回到之前的工作现场所在的分支
git checkout dev
# 9、查看当前分支保存那些工作现场(之前封冻存储的工作现场)
git stash list
# 10、恢复存储的现场
git stash pop
```

### 7、打标签

使用Git可以给指定提交打上标签，用来突出显示这个提交，比如将提交标记为v1.0、v2.0，等等

（1）列举标签

```bash
# 列出所有标签
git tag
# 列出包含指定字符的标签
git tag -l "v1.*"
```

（2）创建标签

```bash
git tag -a v1.0 -m "version 1.0"
# 可为当前提交创建一个标签，标签名为v1.0，-m选项后就是该标签的附注信息
```

（3）推送标签

只使用git push命令在默认情况下不会将标签推送到 远程仓库，在创建标签后需要执行如下命令将指定标签推送到远程仓库：

```bash
# 推送指定标签
git push origin <tagname>
# 推送所有标签
git push origin --tags
```

（4）删除标签

使用git tag的-d选项即可删掉**本地仓库**上的指定标签，如下：

```bash
git tag -d <tagname>
```

但是该指令不会删除**远程仓库**中的标签 ，**还需要**使用如下命令来更新远程仓库：

```bash
git push remote :refs/tags/<tagname>
```

（5）切换标签、

使用git checkout 指令即可将git仓库的HEAD指针指向标签所在的提交，如下：

```bash
git checkout v1.0
```

### 8、设置忽略文件

```bash
在仓库目录下创建 .gitignore (内容：要忽略的文件名)

git add .

git commit -m "have ignore"

git push (提交后不包括.gitignore里的文件)
```

***

## IDE中的git使用

### 1、VSCode中的git使用

#### （1）本地仓库操作

当对仓库已经被跟踪的文件进行修改的时候，会有三种文件状态。如图：

![img](https://pic3.zhimg.com/80/v2-ef4f2f4e1453c1311cf6ffb0f045057a\_720w.webp)

* M(Modify)，表示该文件存在修改
* D(Delete)，表示该文件被删除
* U(Update)，表示该文件是新添加的

选中文件即可查看已进行的修改：

![img](https://pic1.zhimg.com/80/v2-21bd732c279178de83fa5d46fd6092a4\_720w.webp)

接下来可以对这些更改进行处理，可以选择放弃修改或者保存修改，选择放弃修改的话，该文件就会回退到上次保存的版本：

![img](https://pic4.zhimg.com/80/v2-cf629edd3ff909f59eb01649d91caf6b\_720w.webp)

也可以点击上面的图标对所有更改进行处理：

![img](https://pic1.zhimg.com/80/v2-99bad335d95735ca5b3bdcbd411bff30\_720w.webp)

我们选择保存所有修改，所有已修改文件就会保存到暂存区，对应的git命令为`git add .`

接下来将暂存区的改动提交到本地的版本库，点击上方的“√”，对应git命令`git commit`，然后添加message即可：

![img](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211141546531.png)

这时候所有的修改就已经处理完毕了：

![img](https://pic2.zhimg.com/80/v2-557dcf3b8c86daa348daa9fca6f8c5d1\_720w.webp)

#### （2）推送到远程仓库

将本地仓库上的修改推送到远程仓库，对应git命令`git push`

![img](https://pic3.zhimg.com/80/v2-7f0d54b382afc8f6059c83de4d7c7022\_720w.webp)

#### （3）从远程仓库拉取

与推送类似，如图，对应git命令`git pull`：

![img](https://pic1.zhimg.com/80/v2-5742bd638076376dea0eeea548850f34\_720w.webp)

#### （4）分支管理

VScode可以直接在左下角创建/切换分：

![img](https://pic3.zhimg.com/80/v2-675d844f01c011c1a0a422379dd42f6e\_720w.webp)

合并分支：

![img](https://pic3.zhimg.com/80/v2-9447b8ffaeca8879a8c819f7834c1ec2\_720w.webp)

如果待合并的分支上的修改和master没有冲突，就可以直接合并。但是在多人协作时常常会出现两个分支存在不同修改的情况，这时候就要对这些冲突进行处理：

![img](https://pic3.zhimg.com/80/v2-98ef03a3554f080504c3f125179d3306\_720w.webp)

#### （5）好用插件

使用VScode自带的git支持对于个人开发来说已经足够了，但是在应对团队协作时的文件冲突时还略显不足，这时候我们可以借助VScode中的GitLens插件，使用方法详见[git源代码管理插件GitLens](https://link.zhihu.com/?target=https%3A//www.jianshu.com/p/a91cb8a2e55d)

### 2、IDEA中的git使用

#### （1）在IDEA中配置git

如果Git安装在默认路径下，那么idea会自动找到git的位置，如果更改了Git的安装位置则需要手动配置下Git的路径。选择文件→设置，打开设置窗口，搜索git选择下面的git选项：

![](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211142157231.png)

点击测试按钮,现在执行成功，配置完成.

#### （2）在IDEA中操作git

场景：本地已经有一个项目，但是并不是git项目，我们需要将这个放到远程仓库里，和其他开发人员继续一起协作开发。

> 注意，这里的.gitignore里面写的是（因为不想让.git和.idea也上传）
>
> ```
> .git
> .idea
> ```

**①创建项目远程仓库**

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211142200383.png)

**②初始化本地仓库**

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211142201948.png)

**③设置远程仓库**

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211142201693.png)

**④提交到本地仓库**

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211142202780.png)

**⑤推送到远程仓库**

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211142203442.png)

**⑥查看git log版本相关内容**

![](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211142211120.png)

**⑦查看代码的不同处**

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211142213959.png)

**⑧克隆远程仓库到本地**

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211142214635.png)

**⑨创建分支**

一般使用第一种方式较多，因为它可以在任意依次提交的节点上增加分支

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211142214728.png)

**⑩解决冲突**

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Perryen/Typora\_Picture/img/202211142217982.png)

## 参考资料

[Git操作详解以及在VScode中的使用 - 知乎](https://zhuanlan.zhihu.com/p/276376558)

[如何在 GitHub 下载某个程序的特定版本（代码）？ - 知乎](https://www.zhihu.com/question/20080881)

[git 新建分支并同步到远程](https://blog.csdn.net/ezconn/article/details/85250585)

[如何使用Git进行团队协作开发？ - 掘金](https://juejin.cn/post/6844904178146361352)

[Github 使用经典问题：如何同步 fork 项目原仓库的更新 - 知乎](https://zhuanlan.zhihu.com/p/291845721)

[Git 详细安装教程](https://blog.csdn.net/mukes/article/details/115693833)

[Git版本控制及Goland使用Git教程](https://blog.csdn.net/qq\_42956653/article/details/121613703#4\_IDEAGit\_338)
