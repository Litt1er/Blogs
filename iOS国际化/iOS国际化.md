### 国际化简介
国际化，又常常被称为本地化。是根据用户操作系统的语言、或者所在的地区，自动的将应用程序的语言设置为何用户操作系统语言一致。国际化不是必要的，如果应用程序仅仅面对一个国家、地区的用户，那只需要一种语言，是不需要国际化的。如果应用程序要面对多个国家、地区的用户，就要面临国际化的问题。举例来说，微信是在国内的称呼，在海外，app名称是WeChat，这其实就是国际化。

因为最近App中新增了国际化的功能，所以对国际化做一个总结。文章的题目是"iOS国际化"，实际上macOS的国际化也是一模一样的。

App国际化通常包含三部分：App名称国际化，App内的文案国际化，App图标国际化。因为我们项目此次没有涉及到图标国际化，本次只介绍App名称国际化和App内文案国际化。

### 国际化前期准备
首先新建项目LanguageTest,点击项目名，选中PROJECT -> Info -> Localizations，该项下Use Base Internationalization是默认勾选的，保持勾选状态即可。

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCEc2ca3e7961bb47eb21db8bbddef2f6e5/14444)

点击+号，添加想要配置的语言，添加中文简体和中文繁体，加上默认的英文，共3种语言。

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE47cb0800fe551c9f18fe0e8f6e58137e/14446)

接下来就可以设置App名称国际化了。
### App名称国际化
#### 新建InfoPlist文件
App名称支持国际化，必须要新建InfoPlist.strings文件，注意，文件名必须是这个。新建InfoPlist的步骤：
1. commond+N新建文件，或者鼠标右键新建文件
2. 选择Resource下面的Strings File
3. 输入InfoPlist,点击crate,新建成功

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE992d88616c223de107b8e17634ed187f/14448)

一定要注意名称不要写错。新建InfoPlist文件之后：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE7d59053d246dc4bce87d062a20c83b58/14450)

#### 添加InfoPlist语言文件
选中InfoPlist文件，点击Xcode右侧文件检查器，选择Localize

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE3ca2f6bc32cfd19539aa52a2a859aae6/14452)

将英文、简体中文、繁体中文分别勾选上

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE952e6c88e67628c4646f381f94010fb1/14456)

这样，在InfoPlist.strings下面就会多三个文件，分别对应英文，简体中文，繁体中文

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE6f499a5f72acddab534f8360feb2ee47/14458)

#### 分别配置App名称
在上面生成的三个文件中，分别配置App的名称。英文本地化文件中输入：
```
CFBundleDisplayName = "LanguageTest";
```
中文简体本地化文件中输入：
```
CFBundleDisplayName = "语言测试";
```
中文繁体本地化文件中输入：
```
CFBundleDisplayName = "語言測試";
```
注意，这里的key只能是CFBundleDisplayName。到这里，App名称国际化就完成了。设置手机语言，分别看一下不同语言下App显示的名称。

英文环境下：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCEfb3bf2d263db4bdec9893d36066c04e9/14442)

简体中文环境下：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE413ce83803704d80b41b06096f8265fa/14462)

繁体中文环境下：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE20d8e92420020c9c57bbb99d5a08add4/14464)

### App文案国际化
App中不可避免的有很多展示给用户的文案，这些文案也是需要国际化的。App内文案的国际化基本上和App名称国际化类似，只是创建的文件名称不一样。
#### 新建Localizable文件
首先新建Localizable.strings文件，commond+N或者鼠标右键，选择选择Resource下面的Strings File，输入Localizable，点击create

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE98fa7f82d0f55acc00cd8718242cd605/14468)

注意，文件名必须为Localizable，不要写错。
#### 添加Localizable语言文件
选中Localizable文件，点击Xcode右侧文件检查器，选择Localize，将英文、中文简体、中文繁体都勾选上

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE37c806ea4f272bf4a47e73d9d8ce0a99/14470)

同理，项目中的Localizable.strings下面会增加三种语言的文件，分别是英文、中文简体、中文繁体

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCEf2e4c23e705c110d2209116acd5a1972/14472)

#### 分别配置各语言文案
接下来就是在各个语言文件中配置对应的文案，形式是"key=value;"。因为平常主要使用的是简体中文，所以key是简体中文的文案，value对应英文或者繁体中文。举例来说，项目中有一句文案是"点击我",为了实现多语言，需要在英文、繁体中文中进行配置：

Localizable.strings(English)中输入：
```
"点击我"="click me";
```
Localizable.strings(Chinese(Traditional))中输入：
```
"点击我"="點擊我";
```
为何中文简体不需要配置呢？因为我们在项目中写的文案就是中文简体，如果没有在语言文件中找到对应的value，则默认使用key，即中文简体。
#### 代码中文案使用宏
App中的文案已经配置好了，那么代码中如何使用呢？

不需要国际化时，在定义字符串时，我们通常是这样写的：
```
NSString *title = @"点击我";
```
如果需要国际化，这样写显然是不可以的。幸运的是，系统已经提供了国际化的宏，我们只需要使用系统提供的宏就可以：
```
// 第二个参数传nil即可
NSString *title = NSLocalizedString(@"点击我", nil);
```
看一下NSLocalizedString的定义：
```
#define NSLocalizedString(key, comment) \
	    [NSBundle.mainBundle localizedStringForKey:(key) value:@"" table:nil]
```
使用NSLocalizedString宏，代码在运行时，会根据语言自动去寻找对应应该显示的文案。

如果感觉每次使用NSLocalizedString繁琐，我们也可以定义一个更简单的宏，方便编码：
```
#define _L(key)   NSLocalizedString(key, nil)
```
这样，在写文案时，只需要：
```
NSString *title = _L(@"点击我");
```
可以提高编码效率。

至此，文案的国际化也就完成了。看一下效果：

语言为中文简体：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE82bc29c9c8eb719cb4fc9594471025c2/14482)

语言为中文繁体：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE2297ab0cab8455602852e79b0cfa6dc4/14475)

语言为英文：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCEdceb1b6bc9845c88b8bcc76022a543e1/14477)

#### 快速改变模拟器语言
涉及到国际化时，开发过程中肯定要多次测试。使用模拟器开发时，改变系统语言，可以从设置里面修改。这种方法有个缺点，就是修改系统语言之后模拟器需要重启，等的时间比较久。实际上Xcode提供了更简单的方法:Product -> Scheme -> Edit Scheme -> Run -> Options -> Application Language，中可以选择语言

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE4c88ee972d177549ad7d4da669767ff2/14460)

直接修改这里的语言，然后重新运行就可以。

需要注意的是，这种方法只针对App内文案的国际化有效。App名称的国际化，还是需要到设置里面修改系统语言，重启才能够生效。
### 国际化实现
经过上面的步骤，App名称和App内的文案都已经支持国际化了。在手机上运行时，选择系统语言，然后启动app就可以看到国际化的效果。但是，但是，仅仅这样是不够的。假设产品提了这样的需求，在app内可以让用户选择语言实现国际化，不需要更改系统语言，该怎么做呢？

实际上，App的语言是存储在NSUserDefaults中的，key是AppleLanguages。我们通过代码可以得到AppleLanguages的值：
```
// 直接得到的是一个语言数组，数组中存储了app所支持的语言
NSArray *languages = [[NSUserDefaults standardUserDefaults] valueForKey:@"AppleLanguages"];
// 数组中的第一个语言即是当前设置的语言
NSString *currentLanguage = languages.firstObject;
NSLog(@"currentLanguage = %@",currentLanguage);
```
既然这样，我们可以修改NSUserDefaults中AppleLanguages的值，以达到修改app语言，实现国际化的目的。使用下面的代码：
```
// 设置当前app语言为中文简体
NSArray *languageArray = @[@"zh-Hans"];
[[NSUserDefaults standardUserDefaults] setObject:languageArray forKey:@"AppleLanguages"];
```
设置成功后，重启app，生效，大功告成。

然而，还没有结束。上面也看到了，这种方式必须要重启app才能够生效，这对用户来说是不友好的。有没有方法不重启app就实现国际化的效果呢？看下面。

既然需要重启app才生效，那有没有方法能够模拟app重启呢？还真有。可以在AppDelegate中重新设置rootViewController达到模拟重启的效果，代码如下：
```
/**
 重新设置rootViewController
 */
- (void)resetRootVC
{
    self.window.rootViewController = nil;
    ViewController *rootVC = [[ViewController alloc] init];
    self.window.rootViewController = rootVC;
}
```
运行，修改app语言，还是不行。看来仅仅这样是不够的，继续分析。

最开始配置的App名称多语言文件、App文案多语言文件，经过编译之后，会被打包成一个个.lproj文件，如en.lporj,zh-Hans.lproj，看一下示例程序包中的内容：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE4c78af264597cd98f431156b1669a83a/14658)

再看一下chrome浏览器中关于国际化的文件：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE549bf07243089a6a108c6832819b59f5/14656)

App在启动时，会根据NSUserDefaults中设置的语言，选择对应的.lproj文件夹，以该文件夹中的文字作为国际化的资源。这也就解释了，为何在app内修改app语言后，必须重启才生效，因为.lproj文件夹已经确定了，即使改变了语言，也没有任何作用。

上面提到过，用于国际化的NSLocalizedString是一个宏，再来看一下该宏定义：
```
#define NSLocalizedString(key, comment) \
	    [NSBundle.mainBundle localizedStringForKey:(key) value:@"" table:nil]
#define NSLocalizedStringFromTable(key, tbl, comment) \
	    [NSBundle.mainBundle localizedStringForKey:(key) value:@"" table:(tbl)]
```
可以看到，无论是NSLocalizedString还是NSLocalizedStringFromTable最后调用的都是NSBundle.mainBundle的localizedStringForKey方法。如果新定义一个子类继承自NSBundle，并且重写了localizedStringForKey方法，在该方法中，根据设置的语言，选择对应的bundle，这样就可以了。

假设我们新建LanguageBundle类继承自NSBundle，LanguageBundle中实现两个方法。重写父类的localizedStringForKey以及判断对应的.lproj文件夹是否存在，代码如下：
```
/**
 重写父类的方法
 */
- (NSString *)localizedStringForKey:(NSString *)key value:(NSString *)value table:(NSString *)tableName
{
    // 如果有对应的语言资源文件夹，则使用对应的；否则使用mainBundle的语言资源文件夹
    if ( [LanguageBundle languageMainBundle] ) {
        return [[LanguageBundle languageMainBundle] localizedStringForKey:key value:value table:tableName];
    } else {
        return [super localizedStringForKey:key value:value table:tableName];
    }
}

/**
 判断是否有对应的lproj文件夹

 @return bundle
 */
+ (NSBundle *)languageMainBundle
{
    if ( [NSBundle currentLanguage].length ) {
        NSString *path = [[NSBundle mainBundle] pathForResource:[NSBundle currentLanguage] ofType:@"lproj"];
        if ( path.length ) {
            return [NSBundle bundleWithPath:path];
        }
    }
    return nil;
}
```
但是现在还有个问题是，宏定义中最后调用的是NSBundle的localizedStringForKey方法，我们虽然重写了localizedStringForKey方法，但是没效果，因为执行的不是LanguageBundle的localizedStringForKey方法。

解决方法就是使用runtime。使用object_setClass方法，将NSBundle替换为LanguageBundle。这样，localizedStringForKey方法的调用者就会成为LanguageBundle,而不是NSBundle。

新建一个NSBundle的category，并在其+load方法中实现上述的逻辑：
```
+ (NSString *)currentLanguage
{
    NSArray *array = [[NSUserDefaults standardUserDefaults] objectForKey:@"AppleLanguages"];
    return array.firstObject;
}

+ (void)load
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        object_setClass([NSBundle mainBundle], [LanguageBundle class]);
    });
}
```

重新运行程序，修改app语言后，app文案能够直接改动，无需重启app，大功告成。

注意：重写Bundle的NSLocalizedString方法需要和上面提到的模拟重启方法结合使用才能达到最终的效果。还有另外一种方法，不需要模拟重启也可以达到效果，重写Bundle的NSLocalizedString方法还是需要的，就是发送通知。

修改App语言后，发送对应的语言改变的通知，在所有涉及到文案国际化的地方接收通知，收到通知后，重新设置文案。很明显，这种方法比较繁琐，如果项目立项开始没有确定使用这种方法，文案分散在各个位置，修改起来的工作量是巨大的。因此推荐使用重新设置rootViewController的方法。