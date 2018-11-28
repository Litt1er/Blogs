## CocoaPods简介
CocoaPods是iOS开发、macOS开发中的包依赖管理工具，效果如Java中的Maven，nodejs的npm。

CocoaPods是一个开源的项目，源码是用ruby写的，[源码地址](https://github.com/CocoaPods/CocoaPods)在GitHub上。

无论是做iOS开发还是macOS开发，都不可避免的要使用到一些第三方库，优秀的第三方库能够提升我们的开发效率。如果不使用包依赖管理工具，我们需要手动管理第三方包，包括但不限于：
1. 将这些第三方库的源码拷贝到项目中
2. 第三方库代码有可能依赖一些系统framework，我们需要把第三方库依赖的framework导入到项目中
3. 当第三方库有更新时，需要将更新过的代码拷贝到项目中

以上工作虽然简单，但是如果项目中的第三方库较多，需要耗费大量的时间和精力。CocoaPods可以将我们从这些繁琐的工作中解放出来。
## 安装CocoaPods
安装CocoaPods比较方便。通常情况下，macOS都安装了ruby，直接使用ruby 的gem命令即可安装CocoaPods。

使用如下命令可以查看有没有安装ruby：
```
// 如果能正确的输出版本号，则说明ruby已经正确安装
ruby --version
```
使用如下命令可以查看gem的版本号：
```
// 该命令会输出gem的版本号
gem --version
```
如果gem的版本号过低，安装CocoaPods可能会失败。所以在安装CocoaPods之前可以升级一下gem，使用如下命令：
```
// 更新gem
sudo gem update --system
```
另外需要注意的是，ruby的软件源https://rubygems.org 使用的是亚马逊云的服务，国内普通网络是不能访问的。如果不能访问，可以将ruby的源换成国内淘宝的源，命令如下：
```
gem sources --remove https://rubygems.org/
gem sources -a https://ruby.taobao.org/
```
操作完后，可以验证下更换源是否成功，命令如下：
```
// 如果只有一个淘宝的源，说明更换源成功
gem source -l
```
以上所有工作都完成之后，现在可以安装CocoaPods了，命令如下：
```
// 安装CocoaPods
sudo gem install cocoapods
```
安装成功后，在使用之前，还需要对CocoaPods初始化，命令如下：
```
// 这一步花费的时间比较久，耐心等待即可
pod setup
```
测试一下CocoaPods有没有安装成功：
```
// 如果能正确显示版本号，说明CocoaPods安装成功
pod --version
```
## 使用CocoaPods
### 使用CocoaPods安装第三方框架
CocoaPods主要是用于iOS项目、macOS项目管理第三方框架，因此在介绍如何使用CocoaPods时，需要结合iOS项目或者macOS项目。这里新建一个iOS项目TestCocoaPods。
1. 进入项目中和.xcodeproj同级的文件夹，如图：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE40e4b5174e067aaec22b6bbe74f3b633/14102)

2. 在该目录下新建一个Podfile文件，可以使用命令：
```
touch Podfile
```
新建Podfile后如下图：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE643f78e6756b18c164e1b3d207e7e6e4/14104)

3. 编辑Podfile文件。Podfile文件有其固定的格式，可以从网上找一个，然后修改里面的内容即可。这里随便贴一个：
```
# Uncomment this line to define a global platform for your project

platform :ios, '8.0'

target 'TestCocoaPods' do
  pod 'SDWebImage', '~> 4.3.2'
end

```
当然可以增加更多的第三方库，上述示例中只增加了1个，是SDWebImage。

Podfile文件中需要写明平台，是iOS还是osx(macOS)，以及第三方库所要支持的系统最低版本号。之后是target，一个Podfile中可以有多个target。比如说插件开发中，主项目和插件项目所依赖的包可能是不同的，就可以写两个target，分别设置依赖的第三方库。需要导入一个第三方库，只需要
```
pod 'package name', 'version number'
```
即可。版本号有多种表示方式，这里简单介绍几种：

（1）'>=1.0'  最低版本号为1.0

（2）'<=1.0'  最高版本号为1.0

（3）'~>1.0'  兼容1.0的版本的最新版本

通常情况下使用 ~> 的方式。
4. 安装所依赖的第三方库。安装使用的方式是命令行，在该目录下执行下述命令即可：
```
pod install
```
安装之后会发现该目录下有较大的变化，如下图：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE2aae7e8383b8400d568aff149f7ce868/14106)

多了Pods目录，且Pods目录里面也是一个单独的工程。

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCEe43f15314a6a6ec03232a0922518bd4e/14110)

多了TestCocoaPods.xcworkspace文件，以后打开TestCocoaPods项目时，需要打开TestCocoaPods.xcworkspace而不是TestCocoaPods.xcodeproj。

打开TestCocoaPods.xcworkspace后，可以发现，里面包含两个工程，分别是TestCocoaPods和Pods。

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCEe2c03eb353b6bb9672c3786fb2789c1f/14112)

在TestCocoaPods中使用第三方库，直接import即可。
5. 当有需求增加或者删除依赖的第三方库时，直接修改Podfile文件即可，修改完毕之后，执行命令：
```
pod install
```
即可。

如果有需求修改依赖的第三方库的版本号，修改完毕之后，执行命令：
```
pod update
```
即可。
### 使用CocoaPods查找第三方框架
在使用CocoaPods时，可以提前检查第三方框架是否在CocoaPods的管理之下，使用的命令是search：
```
pod search 框架名
```
这是我search YYWebImage的结果：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCEb4d4e8c5d6a948ac8006674e8af80926/14133)

### CocoaPods的大概工作原理
CocoaPods的使用相对来说是比较简单的。那么CocoaPods是如何完成这些工作的？以及为何生成了一个Pods工程？

实际上，CocoaPods是将所有依赖的第三方库都放到了Pods项目中

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCEd2be0dbcf4b9f592813a347a34110bd7/14152)

所有的源码管理工作从住项目转移到了Pods项目中。

Pods项目最终会编译成一个libPods-项目名.a的文件，主项目只需要依赖这个.a文件即可。

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCEd5d274119a6bea4ddb22cedea5dc0efa/14154)

对于libPods-TestCocoaPods.a这个文件，可以将其理解为各个第三方库的.a文件的集合。在本例中,libPods-TestCocoaPods.a就是libPureLayout.a和libSDWebImage.a的集合。

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE50f19c5d27540fa1c7a2c845c97e5318/14157)
## 开源项目支持CocoaPods
下面介绍一下如何让自己的开源项目支持CocoaPods。
### gitHub上新建仓库
首先，需要在gitHub上新建仓库，新建仓库时记得选择开源协议，通常选择MIT，另外就是设置成项目为public。这里新建一个仓库ACMoreResponseButton。

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE6cdf6a1501045afa298ea1bc82af656d/14170)

记得选择开源协议为MIT：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE258e0ece133a7e1b5bedbb41290f78d6/14225)

#### 各种开源协议
开源协议有多种，如MIT、BSD等，常见的有6种，关于这6种开源协议的区别，网上有一张图描述的是非常清楚的，这里贴一下：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE9508e4c043594d9842a912f187d0f850/14099)

可以看到，MIT许可证是要求最不严格的许可证，可以给其他开发者更大的空间。这也是为何多数开源框架都使用MIT许可证的原因。
### 将gitHub上的仓库克隆到本地
使用git clone命令，将gitHub上的仓库克隆到本地：
```
git clone https://github.com/acBool/ACMoreResponseButton.git
```
克隆完之后，在本地仓库上新建项目，并完成对应的功能。之后，使用git add、git commit、git push命令，将本地的修改提交，并且推送到远程仓库，这些步骤不再详细介绍。
### 新建podspec文件
凡是支持CocoaPods的开源库，都需要具备podspec文件，podspec文件可以理解成是对该开源库的描述，包括作者信息，项目主页等。新建podspec文件使用下述命令：
```
pod spec create ACMoreResponseButton
```
新建podspec之后:

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE85bdbc6c9e54b27e9b17570e433ed38f/14231)

### 编辑podspec文件
podspec文件新建之后，里面会有一些信息，可以看做是一个模板，我们只需要稍微对podspec的文件做改动即可。这里贴一下我修改之后的podspec文件：
```
s.name         = "ACMoreResponseButton"
s.version      = "1.1.0"
s.summary      = "This is a moreResponseArea Button"
s.homepage     = "https://github.com/acBool/ACMoreResponseButton"
s.license      = "MIT"
s.author       = { "wmn" => "acbool@163.com" }
s.platform     = :ios, "9.0"
s.source       = { :git => "https://github.com/acBool/ACMoreResponseButton.git", :tag => "1.1.0" }
s.source_files  = "MoreResponseButtonExample/MoreResponseButtonExample/ACMoreResponseButton/*"
s.exclude_files = "UIKit"
s.requires_arc = true
```
podspec文件中可以做更多的配置，如果想要了解更多，可以参考gitHub上一些比较好的开源库，看下podspec文件是怎么写的。
### 分支新建tag，并推送到远程仓库
在上面的podspec文件中注意到，tag值为1.1.0，因此我们需要在分支上新建一个tag，并且将该tag推送到远程仓库，命令如下：
```
// 新建tag
git tag 1.1.0
// 将本地的tag推送到远程仓库
git push --tags
git push
```
这里的tag值需要和podspec中写的保持一致。
### 验证podspec文件
在将开源库提交至CocoaPods之前，我们需要验证一下podspec文件，验证命令如下：
```
pod spec lint ACMoreResponseButton.podspec
```
如果验证不通过，会提示有几个警告，有几个error，且警告信息，error信息都会标识出来。需要注意的是，无论是警告还是error，都需要解决。

如果验证通过，会提示如下图：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE602b9d73666b2da1d4ce11163c8eddba/14234)

### 提交至CocoaPods
注意，只有podspec文件验证通过后，才能将开源库提交至CocoaPods，否则即使提交了也不会成功。提交CocoaPods的命令如下：
```
pod trunk push ACMoreResponseButton.podspec
```
提交成功之后如下图：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE0e90ca88ffac2cb93a737ac0f6578340/14232)

#### 注册CocoaPods
注意，如果之前没有提交过开源库到CocoaPods，需要先注册一下。注册的命令为：
```
// 注意将邮箱名和昵称替换
pod trunk register test@163.com '昵称名' --description='描述'
```
执行完毕后，CocoaPods会给对应的邮箱发送一封确认邮件，点击邮件中的确认链接即可。注册成功后，再执行上面的提交步骤。
## 私有库支持CocoaPods
在公司项目中，有时一些通用的功能会封装成框架，这些框架也是可以支持CocoaPods的。所不同的是，我们希望这些框架只为公司内部使用，并不是开源的，可以称之为私有库。

私有库支持CocoaPods的步骤和公有库基本一致，区别就是不需要提交至CocoaPods，也就是验证podspec文件通过后就可以了。

另外就是，使用私有库时，Podfile文件的写法也有细微区别。Podfile文件中引入私有库时的写法：
```
// 注意替换私有git域名
pod 'ProjectName',git=>"https://XXX.git"
```
