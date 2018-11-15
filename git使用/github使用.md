macOS下查看隐藏文件可以使用快捷键commond+shift+.

github是一个基于git的代码托管平台。付费用户可以建立私人仓库，一般的免费用户只能使用公共仓库，也就是代码是公开的。

一：

注册github帐号，注册地址：https://github.com/

二：

安装git：https://git-scm.com/downloads  从这里下载git安装。验证是否安装成功方法：在terminal中输入git 命令，如果可以被系统识别，说明git安装成功。

三：

安装GitHub Desktop，操作github的桌面客户端,使用客户端操作非常方便，下载地址https://desktop.github.com/

四:

启动GitHub Desktop，使用注册的账号登录，登录后新建一个Repository(代码仓库)。新建的Repository只是存在于本地，并没有同步到github网站上，点击[Publish repository]按扭将新建的Repository同步到github上。

五：

新建的Repository是空的，之后可以向里面添加工程、文件，添加之后更新到github就可以了。



## gitLab命令
一：克隆项目，在本地生成同名目录，并且目录中会有项目文件
```
git clone git@iZbp1h7fx16gkr9u4gk8v3Z:root/test.git
```
二：上传文件
（1）将文件添加到索引中：
```
git add test.sh
```
（2）将文件提交到本地仓库
```
git commit -m "test.sh"
```
（3）将文件同步到Gitlab服务器
```
git push -u origin master
```

(4) 备用
```
如果你已经使用过git了，那么这一步对你来说可以跳过了。整体来说比较简单的。下面的$project_root代表工程根目录

进入工程目录 cd $project_root
初始化git仓库 git init
添加文件到仓库 git add .
提交代码到仓库 git commit -m 'init commit'
链接到git server git remote add origin git@example.com:namespace/projectname.git
push代码到服务器 git push origin master
```
(5) 设置忽略项
```
有一些文件或文件夹是我们不想要被版本控制的，比如.DS_Store build\ xcuserdata thumbs.db，git提供了一种忽略的方案。

在项目根目录下创建.gitignore文件，然后把需要忽略的文件或文件夹名写进去。这样就可以忽略这些文件受版本控制啦。

svn也提供了这样忽略的方案，svn也可以设置全局忽略。svn的此配置放在~/.subversion/config中global-ignores的值。
```
(6) 查看当前git状态
```
git status
git branch -vv
git checkout -b develop origin/develop

```

### git命令
#### 配置用户信息
使用Git之前需要首先配置个人信息，包括个人的用户名和电子邮件地址。每次Git提交时，都会引用用户名和电子邮件，说明是谁提交了更新。配置用户信息的命令：
```
// 配置用户名
git config --global user.name "Test"
// 配置用户电子邮箱
git config --global user.email 123456@qq.com
```
#### 文本编辑器
当Git需要用户输入一些额外的信息时，会自动调用一个外部的编辑器给用户使用。默认会使用操作系统指定的默认编辑器，比如说Vi或者Vim。如果想设置成其他的编辑器，可以通过git config来设置，命令如下：
```
// 配置编辑器为emacs
git config --global core.editor emacs
```
当然，在设置之前也可以查看当前的编辑器，命令如下：
```
// 查看所有的config配置
git config --list
// 查看user.name的配置
git config user.name
// 查看当前的编辑器
git config core.editor
```
#### 差异分析工具
在提交代码、合并代码、解决冲突时，需要用到差异分析工具。差异分析工具也是可以设置的，命令如下：
```
// 配置差异分析工具为vimdiff
git config --global merge.tool vimdiff
```
Git可以理解vimdiff、gvimdiff、opendiff等工具的输出信息。
#### 新建git仓库
Git中有仓库(repository)的概念，所有的文件应该都在仓库中。可以通过两种方式新建仓库：在本地新建仓库和从远程服务器clone一个仓库。
##### 在本地新建仓库
在本地新建仓库非常容易，只需要在对应的目录下使用git init命令即可。命令如下：
```
// 新建一个git仓库
git init
```
新建仓库之后，后续就是向仓库中添加文件，并提交。添加、提交的命令之后再介绍。
##### 从远程克隆一个仓库
从远程克隆仓库使用的是git clone命令，比如：
```
// 从远程服务器clone一个git仓库，会在当前目录下新建Blogs文件夹
git clone https://github.com/acBool/Blogs.git
```
当然，clone时也可以指定本地仓库的名字，命令如下：
```
// 本地仓库会被命名为myBlog
git clone https://github.com/acBool/Blogs.git myBlog
```
#### 查看当前文件状态
Git仓库下的文件有已修改、未修改、已暂存、未跟踪几种状态，使用status命令可以查看当前文件的状态，命令如下：
```
// 查看当前文件状态
git status
```
git status命令输出的信息可能有些冗余，可以使用-s参数得到一个更为简介的信息：
```
// 查看当前文件状态，信息更为简洁
git status -s
```
#### 添加文件到git仓库
新建仓库后，可以向仓库中添加文件。但是添加的文件处于未被跟踪的状态，如果要改变文件为跟踪状态，需要使用git add命令，如下：
```
// 将hello.c的状态改为跟踪状态
git add hello.c
```
如果add之后的参数是文件夹，会递归的跟踪该目录下的所有文件：
```
// 会递归的将Tempdir目录下的所有文件改为跟踪状态
git add Tempdir
```
#### 忽略文件
通常情况下，总会有一些文件没必要让Git来管理，也不希望这些文件总是出现在未跟踪文件列表，比如说一些日志文件，或者变异过程中生成的临时文件。这种情况下，可以创建.gitignore文件，在.gitignore文件中列出要忽略的文件即可。比如：
```
// 忽略所有以.a或者.o结尾的文件
*.[oa]
// 忽略所有以~结尾的文件
*~
// 忽略所有以.a结尾的文件
*.a
// 除了lib.a文件，使用！取反
!lib.a
// 忽略TODO文件，而不是TODO文件夹
/TODO
// 忽略build文件夹下的所有文件
build/
```
#### 查看文件做了哪些修改
使用git diff命令可以查看文件做了哪些修改，实际上就是和原来的文件做对比，命令如下：
```
// 查看未暂存的文件做了哪些更新
git diff
// 查看已暂存的文件做了哪些更新
git diff --cached
// 查看已暂存的文件做了哪些更新
git diff --staged
```
其中，git diff --staged和git diff --cached的功能是一样的。
#### 提交更新
提交的命令是 git commit,命令如下：
```
// 如果只输入git commit，会弹出文版编辑器让输入这次提交的信息
git commit
// 加上-m参数，直接输入此次提交的信息
git commit -m 'add file'
```
#### 移除文件
从Git中移除某个文件，使用的命令是git rm,命令如下：
```
// 移除a.test文件
git rm a.test
```
需要注意的是，移除之后，还需要使用commit命令来此次的操作提交。另外，移除后，本地磁盘上的a.test文件也会被删除。

如果只想移除仓库中的a.test文件，而保留本地磁盘上的a.test文件，该如何操作呢？实际上，这样的应用场景是存在的，比如说忘记加.gitignore文件，将一些不必要的文件加入到仓库中，这时候就会有这样的问题。我们想把仓库中没用的文件删除，但是本地还想要保留，git对这种情况是支持的。命令如下：
```
// 从仓库中移除a.test，但保留在本地磁盘
git rm --cached a.test
```
#### 重命名文件
Git中重命名文件可以使用mv命令，如下：
```
// 将a.test文件重命名为b.test
git mv a.test b.test
```
#### 查看提交日志
使用git log命令可以查看一个仓库的提交日志，命令如下：
```
// 默认不带参数，会按照提交时间列出所有的更新，包括提交人昵称，邮箱，最新的提交在最上面
git log
// 显示每次提交的差异，即具体更新了哪些内容
git log -p
// 显示最近两次提交的差异，限制了日志数量
git log -p -2
// 显示每次提交简略的统计信息
git log --stat
```
#### 撤消对文件的修改
如果不小心改了一个文件，但是不想修改该文件，也不想把修改过的文件放入暂存区，提交，Git提供了撤消修改文件的命令：
```
// 撤消修改文件.DS_Store
git checkout -- .Ds_Store
```
#### 将文件从暂存区移除
当想将一个文件从暂存区移除时，可以使用reset命令，如下：
```
// 将.DS_Store文件从暂存区移除
git reset HEAD .DS_Store
```

目前为止，我们所介绍的命令都是和本地仓库相关，提交、添加等，操作的都是本地仓库，那么，如何和远程仓库交互呢？如何将本地仓库的修改推送到远程仓库呢？
#### 从远程仓库拉取
从远程仓库中获取数据，可以使用git fetch或者git pull命令，如下：
```
// 从远程仓库test拉取数据，需要注意的是，使用这种方式拉取的数据，并不会合并到本地仓库
// 还需要手动add、commit之后，才会合并到本地仓库
git fetch test
// 从远程仓库test拉取数据，这种方式拉取的数据，会尝试合并到本地仓库
git pull test
```
#### 推送到远程仓库
推送到远程仓库使用git push命令即可，如下：
```
// 推送到远程仓库test
git push test
```
运行完这条命令后，可能会让输入用户名和密码，以验证是否有推送的权限。
#### 添加远程仓库
添加远程仓库使用的命令是git remote add命令，如下：
```
// 添加一个远程仓库，远程仓库的url是https://github.com/***/***，远程仓库的名称是test
git remote add test https://github.com/***/***
```
#### 移除远程仓库
移除远程仓库使用git remote remove命令即可：
```
// 移除远程仓库test
git remote remove test
```
#### 重命名远程仓库
重命名远程仓库使用rename命令，命令如下：
```
// 把远程仓库a重命名成b
git remote rename a b
```
#### 查看Git标签
Git提供了标签的功能，可以在某些重要的节点增加标签，比如版本上线，可以打上标签。查看已有标签的命令是：
```
// 会列出所有的标签
git tag
// 只列出v1.1.0的标签
git tag -l 'v1.1.0'
```
#### 创建Git标签
创建Git标签使用-a命令，如下：
```
// 创建标签，标签名为v1.4,标签信息为 version 1.4
git tag -a v1.4 -m 'version 1.4'
```