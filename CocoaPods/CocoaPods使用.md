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

## 开源项目支持CocoaPods