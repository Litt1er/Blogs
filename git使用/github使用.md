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