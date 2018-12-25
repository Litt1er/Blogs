### 前言
平常开发中，UIButton是使用频率非常高的控件。除了可点击之外，还因为其能够同时显示文案和图片。默认的UIButton，图片在左，文字在右，且文字是紧紧挨着图片的。大多数情况下，UIButton已经可以满足需求了，然而，总有一些意外……比如，设计让文字和图片不要离得很近；比如，设计让文字在左，图片在右；比如，设计让图片在上，文字在下……

这时候就会用到本文的主角：imageEdgeInsets和titleEdgeInsets了。使用imageEdgeInsets和titleEdgeInsets时，可能或多或少的都会有一种感觉，就是难以捉摸，每次都是尝试多次才能达到一个良好的效果。再彻底会使用imageEdgeInsets和titleEdgeInsets之前，我们先来分析下面对的问题。

### 图片和文字保持间距
首先面临的第一个问题就是图片和文字之间保持一个距离。其实碰到这样的需求解决方法有很多，如果对imageEdgeInsets和titleEdgeInsets不熟悉，完全没必要使用这两个属性。
#### 图片右侧透明
UIButton 默认图片在左，文字在右，且文字和图片紧邻，现在的需求是图片和文字之间留有一定的间距。一种方法是，如果设计提供的图片右侧有一些透明的地方，那么整体给用户的感觉就是图片和文字之间有一定的间距。只不过这种方法需要让设计切图。
#### 文案加空格
其实如果只是想让图片和文字之间有一定间距，不用设计切图，代码完全可以控制，而且特别简单。方法就是在文案的前面加空格，比如说本来要显示的文案是"联系我们",程序中可以设置为"&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 联系我们",这样最终展示给用户的效果图片和文件之间也有一定的间距。这种方法完全是开发人员可控的。
### 图片和文字位置调整
图片和文字的位置调整，主要是指图片在右，文字在左，或者图片在上，文字在下这两种情况，目前还没有碰到过文字在上，图片在下的需求。如果设计的图是图片和文字的位置有了调整，那么仅仅靠增加图片透明度、文案加空格等类似的方法是满足不了需求的。

解决方法有两种，一种是使用UIView，另外就是使用imageEdgeInsets和titleEdgeInsets。
#### 使用UIView解决
使用UIView的思路非常简单。就是使用三个控件，一个UIView作为父控件，一个UILabel用来显示内容，一个UIImageView用来显示图片。由于UILabel和UIImageView都是我们自己写的，其frame我们可以随意控制，图片是在左、在右，还是在上，都是可以控制的。

至于UIButton的点击事件，因为UIView不具备点击事件，我们可以给UIView增加手势UITapGestureRecognizer,用来模拟点击行为。看上去完美解决了这个问题，但是这种方案是有缺点的，看一下：
1. 首先这种方法是使用了3个控件，而如果使用UIButton，只需要使用1个控件。虽然UIButton内部也包含了一个UIImageView和一个UILabel，性能上可能没有太大的差距，但是这种写法麻烦啊。本来使用UIButton5行代码就可以搞定了，结果使用UIView的形式，写了20行代码，实现比较繁琐。
2. 虽然可以给UIView添加手势模拟点击行为，但是，UIView是没有高亮状态的。UIButton默认是有普通状态和高亮状态，点击时会显示默认的高亮状态，这个效果UIView是实现不了的。如果想实现，需要再增加更多的代码。

可以说，使用UIView能够解决问题，但是解决方法不够优雅。

#### imageEdgeInsets和titleEdgeInsets
苹果可能已经考虑到了开发者会有这样的需求，于是提供了imageEdgeInsets和titleEdgeInsets两个属性。然而，由于苹果没有介绍这两个属性的原理，对两个属性的描述又不是特别清晰，导致使用起来难度较大，对于经验不足的开发者，每次使用都要尝试多次。

先来看下两者的定义。
##### imageEdgeInsets和titleEdgeInsets的定义
imageEdgeInsets和titleEdgeInsets的定义在UIButton.h中，如下：
```
@property(nonatomic)          UIEdgeInsets titleEdgeInsets;                // default is UIEdgeInsetsZero
@property(nonatomic)          UIEdgeInsets imageEdgeInsets;                // default is UIEdgeInsetsZero
```
可以看到，imageEdgeInsets和titleEdgeInsets都是UIEdgeInsets类型，且默认取值是UIEdgeInsetsZero。看一下UIEdgeInsets的定义：
```
typedef struct UIEdgeInsets {
    CGFloat top, left, bottom, right;  // specify amount to inset (positive) for each of the edges. values can be negative to 'outset'
} UIEdgeInsets;
```
UIEdgeInsets是一个结构体，四个值分别是上、左、下、右，值可以为正，也可以为负。

在了解imageEdgeInsets和titleEdgeInsets的使用之前，有三点一定要明白：
1. **无论是imageEdgeInsets还是titleEdgeInsets，都是和原控件的位置相比较的。imageEdgeInsets是和原imageView的位置比较，titleEdgeInsets是和原label的位置比较**。
2. **imageEdgeInsets和titleEdgeInsets中的值为正，则是该方向上的扩张，如果值为负，则是该方向上的缩减。举例来说，对于左侧，扩张是更向左，即frame的x值减小；对于右侧扩张是更向右，frame的width值更大**。
3. **UIEdgeInsets是对称的，左右对称，上下对称。因此，在设置imageEdgeInsets和titleEdgeInsets时尽量也要对称，比如[0,-5,0,5],左右对称，同理上下也要对称。之所以要对称，是为了不拉伸imageView和titleLabel**。

##### imageEdgeInsets和titleEdgeInsets的使用
有了上面的了解和介绍，来看一下使用imageEdgeInsets和titleEdgeInsets如何解决文中最开始提到的问题。

首先是图片和文字之间保持间距。图片和文字保持间距有两种处理方式，一种是图片左移，一种是文字右移。实际上，最开始提到的给图片右侧增加透明度以及文案加空格正是分别对应了图片左移和文字右移。如果使用imageEdgeInsets和titleEdgeInsets，对应的也是这两种处理方式。我们可以调整imageEdgeInsets来使图片左移，同理也可以调整titleEdgeInsets来使图片右移，可以达到相同的效果。

写代码验证一下。首先看一下普通状态的UIButton:

为了后续方便，先定义一些宏：
```
#define     DefaultFont     [UIFont systemFontOfSize:20.0f]
#define     DefaultImage    [UIImage imageNamed:@"mail"]
#define     DefaultText     @"技术支持"
```
生成一个普通的UIButton:
```
- (UIButton *)createBtn
{
    UIButton *btn = [[UIButton alloc] init];
    [btn setTitle:DefaultText forState:UIControlStateNormal];
    btn.titleLabel.font = DefaultFont;
    [btn setImage:DefaultImage forState:UIControlStateNormal];
    [btn setTitleColor:[UIColor blackColor] forState:UIControlStateNormal];
    btn.titleLabel.backgroundColor = [UIColor redColor];
    return btn;
}
```
将普通button显示到屏幕上：
```
UIButton *btn0 = [self createBtn];
[btn0 sizeToFit];
btn0.frame = CGRectMake(100,100,btn0.frame.size.width,btn0.frame.size.height);
[self.view addSubview:btn0];
```
看一下效果：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE6f34ed45fdf52134e49b4386e051e147/15873)

可以看到图片和文字是紧紧贴在一起的。

调整imageEdgeInsets的值，使两者保持一定间距。像上文说的，让图片左移，既然是左移，相当于是左侧扩张，为了不拉伸图片，对应的右侧就要缩减。也就是说左侧的值为负，右侧的值为正，且两者的绝对值要相等。
```
UIButton *btn1 = [self createBtn];
btn1.imageEdgeInsets = UIEdgeInsetsMake(0, -5, 0, 5);
[btn1 sizeToFit];
btn1.frame = CGRectMake(100,200,btn1.frame.size.width,btn1.frame.size.height);
[self.view addSubview:btn1];
```
看一下效果：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE78b838033eb7890855cb56ee076d1700/15875)

可以看到图片和文字之间有了一定的间距。

接下来再来看一下，使用imageEdgeInsets和titleEdgeInsets，让文字在左，图片在右，为达到这种效果，imageEdgeInsets和titleEdgeInsets的值都需要调整。先看下图：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCEe8cf673316d32de2f8b6e8ec4d4d2c47/15903)

默认图片在左，文字在右，现在要将文字放到左侧，图片放到右侧。从上图可以看出，最终文字是向左移了，图片是向右移了。根据前面的介绍，文字向左侧移动，那么titleEdgeInsets中的left字段应该为负，right字段应该为正，那么应该移动多少呢？从上面的图也很清晰的看到，移动的距离就是图片的宽度。再看图片，图片整体右移，因此imageEdgeInsets中的left字段应该为正，right字段应该为负，右移的距离是多少呢？正好是文字的宽度。

看一下实现代码：
```
UIButton *btn2 = [self createBtn];
UILabel *label = [[UILabel alloc] init];
label.font = DefaultFont;
label.text = DefaultText;
[label sizeToFit];
btn2.imageEdgeInsets = UIEdgeInsetsMake(0, label.frame.size.width, 0, label.frame.size.width * -1);
btn2.titleEdgeInsets = UIEdgeInsetsMake(0, DefaultImage.size.width * -1, 0, DefaultImage.size.width);
[btn2 sizeToFit];
btn2.frame = CGRectMake(100,300,btn2.frame.size.width,btn2.frame.size.height);
[self.view addSubview:btn2];
```
效果如下：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCEf207d77887b5aec6c001fdfa67cfb6f7/15877)

如果想让两者之间有间距，也很简单，移动时只需要把值调大一些即可：
```
UIButton *btn3 = [self createBtn];
btn3.imageEdgeInsets = UIEdgeInsetsMake(0, label.frame.size.width, 0, label.frame.size.width * -1);
btn3.titleEdgeInsets = UIEdgeInsetsMake(0, DefaultImage.size.width * -1 - 5, 0, DefaultImage.size.width + 5);
[btn3 sizeToFit];
btn3.frame = CGRectMake(100,400,btn3.frame.size.width,btn3.frame.size.height);
[self.view addSubview:btn3];
```
让两者之间保持间距为5，效果如下：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE6a07fd54d508bc15fde7fe30cd877a63/15879)

再来看最后一种效果，即图片在上，文字在下。为达到这样的效果，我们可以将图片向上移，文字的y值保持不变；也可以图片保持不变，文字的位置下移；可以达到相同的效果。如下图：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE0ee4e7b75485a2318f7e35282cb59370/15933)

当然，也可以图片上移一部分，文字下移一部分，只是没有这个必要。

我们以图片上移为例。图片上移，说明imageView的edgeInsets的top值扩张，即top为负，那么bottom值为正。移动的距离正好是图片的高度。
```
btn4.imageEdgeInsets = UIEdgeInsetsMake(-DefaultImage.size.height, 0, DefaultImage.size.height, 0);
```

为了保持美观，我们还需要让titleLabel和imageView的中心点对齐,即titleLabel左移。titleLabel左移，说明titleLabel的edgeInsets的left扩张，值为负，right值为正。titleLabel左移的距离需要计算。从上图可以看出，titleLabel原来的x值正好是imageView.width，现在的x值是(imageView.width - titleLabel.width) * 0.5,因此，titleLabel需要左移的距离是：
```
CGFloat diff5X = DefaultImage.size.width - (DefaultImage.size.width - label.frame.size.width) * 0.5;
```
通常情况下，titleLabel的高度和imageView的高度是不同的，如下图：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCEac3ad17ce37c71abe67b86442a97945a/15953)

titleLabel和imageView竖直方向的中心点一致，但是两者的高度是不同的。如果titleLabel的y值不调整，那么图片和文字之间会有一定的间距，这个间距对于开发者来说不好控制。因此，最好的办法是将titleLabel向上调整，和imageView紧邻。调整的距离是多少呢？根据上图也很容易看出来
```
CGFloat diffY = (DefaultImage.size.height - label.frame.size.height) * 0.5;
```
完整代码如下：
```
UIButton *btn4 = [self createBtn];
btn4.imageEdgeInsets = UIEdgeInsetsMake(-DefaultImage.size.height, 0, DefaultImage.size.height, 0);
CGFloat diff = DefaultImage.size.width - (DefaultImage.size.width - label.frame.size.width) * 0.5;
CGFloat diffY = (DefaultImage.size.height - label.frame.size.height) * 0.5;
btn4.titleEdgeInsets = UIEdgeInsetsMake(-diffY, diff * -1, diffY, diff);
[btn4 sizeToFit];
btn4.frame = CGRectMake(20,550,btn4.frame.size.width,btn4.frame.size.height);
[self.view addSubview:btn4];
```
效果图如下：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE6691ffe8db931cd43f1eecf0e24cafa7/15881)

如果想让文字和图片之间保持间距，只需要把文字向下移动一些即可，代码如下：
```
UIButton *btn5 = [self createBtn];
btn5.imageEdgeInsets = UIEdgeInsetsMake(-DefaultImage.size.height, 0, DefaultImage.size.height, 0);
CGFloat diff5X = DefaultImage.size.width - (DefaultImage.size.width - label.frame.size.width) * 0.5;
CGFloat diff5Y = (DefaultImage.size.height - label.frame.size.height) * 0.5;
btn5.titleEdgeInsets = UIEdgeInsetsMake(-diffY + 5, diff * -1, diffY - 5, diff);
[btn5 sizeToFit];
btn5.frame = CGRectMake(200,550,btn5.frame.size.width,btn5.frame.size.height);
[self.view addSubview:btn5];
```
效果图如下：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE72e2d3dad87824d5832d2d3f23f87c37/15884)

### 封装
了解了imageEdgeInsets和titleEdgeInsets如何使用后，之后再碰到文中开头提到打需求，相信大家已经可以从容应对了。然而，imageEdgeInsets和titleEdgeInsets实在难以理解，一段时间不用，可能就忘了应该如何调整偏移量……每次使用之前都研究一下其偏移量的原理，好像又有点浪费时间。

为了方便日常开发，我封装了ACEdgeInsetsBtn，ACEdgeInsetsBtn是UIButton的子类，主要目的就是解决UIButton中文字、图片有间隔、或者位置调整的问题。使用也很简单，只需要传入image,文案，字体，imageView和titleLabel的间距，类型即可，如下：
```
/**
 初始化方法

 @param image btn所要显示的image
 @param text btn所要显示的text
 @param font btn的字体信息
 @param edgeInsetsType 图片的位置类型
 @param space image和文字之间的间距
 @return    返回实例button对象
 */
- (instancetype)initWithImage:(UIImage *)image text:(NSString *)text font:(UIFont *)font edgeInsetsType:(ACEdgeInsetsBtnType)edgeInsetsType space:(CGFloat)space;
```
ACEdgeInsetsBtnType是一个枚举类型，目前包含了图片左文子右、图片右文字左、图片上文字下三种类型，如下：
```
/**
 btn类型，以图片的位置为标准
 */
typedef NS_ENUM(NSInteger, ACEdgeInsetsBtnType){
    // 普通状态，即图片在左，文字在右
    ACEdgeInsetsBtnTypeNormal = 1,
    // 图片在右，文字在左
    ACEdgeInsetsBtnTypeRight ,
    // 图片在上，文字在下
    ACEdgeInsetsBtnTop
};
```
传入对应的参数，直接生成期望的Btn:
```
ACEdgeInsetsBtn *btn = [[ACEdgeInsetsBtn alloc] initWithImage:DefaultImage text:DefaultText font:DefaultFont edgeInsetsType:ACEdgeInsetsBtnTypeNormal space:5];
[btn setTitleColor:[UIColor blackColor] forState:UIControlStateNormal];
btn.frame = CGRectMake(50,80,btn.frame.size.width,btn.frame.size.height);
[self.view addSubview:btn];
```
项目放在了github上，[地址](https://github.com/acBool/ACEdgeInsetsBtn)。