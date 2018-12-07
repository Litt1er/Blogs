## @property介绍
相信做过iOS开发的同学都使用过@property，@property翻译过来是属性。在定义一个类时，常常会有多个@property，有了@property，我们可以用来保存类的一些信息或者状态。比如定义一个Student类：
```
@interface Student : NSObject

@property (nonatomic, copy) NSString *name;

@property (nonatomic, copy) NSString *sex;

@end
```
Student类中有两个属性，分别是name和sex。

在程序中使用时，可以使用
```
self.name = @"xxx";
self.sex = @"xxx";
```
那么，为什么可以这样用呢？self.name是self的name变量嘛？还是其他的什么？属性中的copy代表什么？nonatomic呢？下面来看一下这些问题的答案。
## @property 本质
@property到底是什么呢？实际上@property = 实例变量 + get方法 + set方法。也就是说属性
```
@property (nonatomic, copy) NSString *name;
```
代表的有实例变量，get方法和set方法。如果大家用过Java，相信对set方法和get方法应该很熟悉，这里的set、get方法和Java里面的作用是一样的，get方法用来获取变量的值，set方法用来设置变量的值。使用@property生成的实例变量、get方法、set方法的命名有严格的规范，实例变量的名称、get方法名、set方法名稍后再介绍。

这里需要注意的是，包括实例变量、get方法和set方法，不会真的出现在我们的编辑器里面，使用属性生成的实例变量、get方法、set方法是在编译过程中生成的。下面介绍一下set方法、get方法以及自动生成的实例变量。

### setter方法
set方法也可以称为setter方法，之后看到setter方法直接理解成set方法即可。同理，get方法也被称为getter方法。

还是以上面的属性：
```
@property (nonatomic, copy) NSString *name;
```
为例，属性name生成的setter方法是
```
- (void)setName:(NSString *)name;
```
该命名方法是固定的，是约定成束的。如果属性名是firstName，那么setter方法是：
```
- (void)setFirstName:(NSString *)firstName;
```
项目中，很多时候会有重写setter方法的需求，只要重写对应的方法即可。比如说重写name属性的setter方法：
```
- (void)setName:(NSString *)name
{
    NSLog(@"rewrite setter");
    _name = name;
}
```
关于_name是什么，后续会介绍。
### getter方法
以属性
```
@property (nonatomic, copy) NSString *name;
```
为例，编译器自动生成的getter方法是
```
- (NSString *)name;
```
getter方法的命名也是固定的。如果属性名是firstName，那么getter方法是：
```
- (NSString *)firstName;
```
重写getter方法：
```
- (NSString *)name
{
    NSLog(@"rewrite getter");
    return _name;
}
```
如果我们定义了name属性，并且按照上面所述，重写了getter方法和setter方法，Xcode会提示如下的错误：
```
Use of undeclared identifier '_name'; did you mean 'name'?
```
稍后我们再解释为何会有该错误，以及如何解决。先来看一下_name到底是什么。
### 实例变量
既然@property = 实例变量 + getter + setter，那么属性所生成的实例变量名是什么呢？根据上面的例子，也很容易猜到，项目中也经常使用，实例变量的名称就是_name。实例变量的命名也是有固定格式的，下划线+属性名。如果属性是@property firstName，那么生成的实例变量就是_firstName。这也是为何我们在setter方法和getter方法，以及其他的方法中可以使用_name的原因。

这里再提一下，无论是实例变量，还是setter、getter方法，命名都是有严格规范的。正是因为有了这种规范，编译器才能够自动生成方法，这也要求我们在项目中，对变量的命名，方法的命名遵循一定的规范。
### 自动合成
定义一个@property，在编译期间，编译器会生成实例变量、getter方法、setter方法，这些方法、变量是通过自动合成（autosynthesize）的方式生成并添加到类中。实际上，一个类经过编译后，会生成变量列表ivar_list，方法列表method_list，每添加一个属性，在变量列表ivar_list会添加对应的变量，如_name，方法列表method_list中会添加对应的setter方法和getter方法。
### 动态合成
既然有自动合成，那么相对应的就要有非自动合成，非自动合成又称为动态合成。定义一个属性，默认是自动合成的，默认会生成getter方法和setter方法，这也是为何我们可以直接使用self.属性名的原因。实际上，自动合成对应的代码是：
```
@synthesize name = _name;
```
这行代码是编译器自动生成的，无需我们来写。相应的，如果我们想要动态合成，需要自己写如下代码：
```
@dynamic sex;
```
这样代码就告诉编译器，sex属性的变量名、getter方法、setter方法由开发者自己来添加，编译器无需处理。

那么这样写和自动合成有什么区别呢？来看下面的代码：
```
Student *stu = [[Student alloc] init];
stu.sex = @"male";
```
编译，不会有任何问题。运行，也没问题。但是当代码执行到这一行的时候，程序崩溃了，崩溃信息是：
```
[Student setSex:]: unrecognized selector sent to instance 0x60000217f1a0
```
即：Student没有setSex方法，没有属性sex的setter方法。这就是动态合成和自动合成的区别。**动态合成，需要开发者自己来写属性的setter方法和getter方法**。添加上setter方法：
```
- (void)setSex:(NSString *)sex
{
    _sex = sex;
}
```
**由于使用@dynamic，编译器不会自动生成变量**，因此除此之外，还需要手动定义_sex变量，如下：
```
@interface Student : NSObject
{
    NSString *_sex;
}

@property (nonatomic, copy) NSString *name;

@property (nonatomic, copy) NSString *sex;

@end
```
现在再编译，运行，执行没有错误和崩溃。
### 重写setter、getter方法的注意事项
上面的例子中，重写了属性name的getter方法和setter方法，如下：
```
- (void)setName:(NSString *)name
{
    NSLog(@"rewrite setter");
    _name = name;
}

- (NSString *)name
{
    NSLog(@"rewrite getter");
    return _name;
}
```
但是编译器会提示错误，错误信息如下：
```
Use of undeclared identifier '_name'; did you mean 'name'?
```
提示没有_name变量。为什么呢？我们没有声明@dynamic，那默认就是@autosynthesize，为何没有_name变量呢？奇怪的是，倘若我们把getter方法，或者setter方法注释掉，gettter、setter方法只留下一个，不会有错误，为什么呢？

还是编译器做了些处理。对于一个可读写的属性来说，**当我们重写了其setter、getter方法时，编译器会认为开发者想手动管理@property，此时会将@property作为@dynamic来处理**，因此也就不会自动生成变量。解决方法，显示的将属性和一个变量绑定：
```
@synthesize name = _name;
```
这样就没问题了。如果一个属性是只读的，重写了其getter方法时，编译器也会认为该属性是@dynamic，关于可读写、只读，下面会介绍。这里提醒一下，当项目中重写了属性的getter方法和setter方法时，注意下是否有编译的问题。
### 修改实例变量的名称
使用自动合成时，针对
```
@property (nonatomic, copy) NSString *name;
```
属性，生成的变量名是_name。倘若，不习惯使用下划线开头的变量名，能否指定属性对应的变量名呢？答案是可以的，使用的是上面介绍过的@synthesize关键字。如下：
```
@synthesize name = stuName;
```
这样，name属性生成的变量名就是stuName,后续使用时需要写stuName,而不是_name。如getter、setter方法：
```
- (void)setName:(NSString *)name
{
    NSLog(@"rewrite setter");
    stuName = name;
}

- (NSString *)name
{
    NSLog(@"rewrite getter");
    return stuName;
}
```
注意：**虽然可以使用@synthesize关键字修改变量名，但是如无特殊需求，不建议这样做**。因为默认情况下编译器已经为我们生成了变量名，大多数的项目、开发者也都会遵循这样的规范，既然苹果已经定义了一个好的规范，为什么不遵守呢？
### getter方法中为何不能用self.
有经验的开发者应该都知道这一点，在getter方法中是不能使用self.的，比如：
```
- (NSString *)name
{
    NSLog(@"rewrite getter");
    return self.name;  // 错误的写法，会造成死循环
}
```
原因代码注释中已经写了，这样会造成死循环。这里需要注意的是：self.name实际上就是执行了属性name的getter方法，getter方法中又调用了self.name， 会一直递归调用，直到程序崩溃。通常程序中使用：
```
self.name = @"aaa";
```
这样的方式，setter方法会被调用。

## @property修饰符
当我们定义一个字符串属性时，通常我们会这样写：
```
@property (nonatomic, copy) NSString *name;
```
当我们定义一个NSMutableArray类型的属性时，通常我们会这样写：
```
@property (nonatomic, strong) NSMutableArray *books;
```
而当我们定一个基本数据类型时，会这样写：
```
@property (nonatomic, assign) int age;
```
定义一个属性时，nonatomic、copy、strong、assign等被称作是关键字，或者是修饰符。
### 修饰符种类
修饰符有四种：
1. 原子性。原子性有nonatomic、atomic两个值，如果不写nonatomic,那么默认是atomic的。如果属性是atomic的，那么在访问其getter和setter方法之前，会有一些判断，大概是判断是否可以访问等，这里系统使用的是自旋锁。由于使用atomic并不能绝对保证线程安全，且会耗费一些性能，因此通常情况下都使用nonatomic。
2. 读写权限。读写权限有两个取值，readwrite和readonly。声明属性时，如果不指定读写权限，那么默认是readwrite的。如果某个属性不想让其他人来写，那么可以设置成readonly。
3. 内存管理。内存管理的取值有assign、strong、weak、copy、unsafe_unretained。
4. set、get方法名。如果不想使用自动合成所生成的setter、getter方法，声明属性时甚至可以指定方法名。比如指定getter方法名：
```
@property (nonatomic, assign, getter=isPass) BOOL pass;
```
属性pass的getter方法就是 
```
- (BOOL)isPass;
```

### 默认修饰符
声明属性时，如果不显示指定修饰符，那么默认的修饰符是哪些呢？或者说未指定的修饰符，默认取值是什么呢？如果是基本数据类型，默认取值是：
```
atomic,readwrite,assign
```
如果是Objective-C对象，默认取值是：
```
atomic,readwrite,strong
```

### atomic是否是线程安全的
上面提到了，声明属性时，通常使用nonatomic修饰符，原因就是因为atomic并不能保证绝对的线程安全。举例来说，假设有一个线程A在不断的读取属性name的值，同时有一个线程B修改了属性name的值，那么即使属性name是atomic，线程A读到的仍旧是修改后的值，可见不是线程安全的。如果想要实现线程安全，需要手动的实现锁。下面是一段示例代码：

声明name属性，使用atomic修饰符
```
@property (atomic, copy) NSString *name;
```
对属性name赋值。同时，一个线程在不断的读取name的值，另一个线程在不断的设置name的值：
```
stu.name = @"aaa";
    
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    for(int i = 0 ; i < 1000; ++i){
        NSLog(@"stu.name = %@",stu.name);
    }
});
    
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    stu.name = @"bbb";
});
```
看一下输出：
```
2018-12-06 15:42:26.837215+0800 TestClock[15405:175815] stu.name = aaa
2018-12-06 15:42:26.837837+0800 TestClock[15405:175815] stu.name = bbb
```
证实了即使使用了atomic,也不能保证线程安全。

### weak和assign区别
经常会有面试题问weak和assign的区别，这里介绍一下。

weak和strong是对应的，一个是强引用，一个是弱引用。weak和assign的区别主要是体现在两者修饰OC对象时的差异。上面也介绍过，assign通常用来修饰基本数据类型，如int、float、BOOL等，weak用来修饰OC对象，如UIButton、UIView等。
#### 基本数据类型用weak来修饰
假设声明一个int类型的属性，但是用weak来修饰，会发生什么呢？
```
@property (nonatomic, weak) int age;
```
Xcode会直接提示错误，错误信息如下：
```
Property with 'weak' attribute must be of object type
```
也就是说，weak只能用来修饰对象，不能用来修饰基本数据类型，否则会发生编译错误。
#### 对象使用assign来修饰
假设声明一个UIButton类型的属性，但是用assign来修饰，会发生什么呢？
```
@property (nonatomic, assign) UIButton *assignBtn;
```
编译，没有问题，运行也没有问题。我们再声明一个UIButton,使用weak来修饰，对比一下：
```
@interface ViewController ()

@property (nonatomic, assign) UIButton *assignBtn;

@property (nonatomic, weak) UIButton *weakButton;

@end
```
正常初始化两个button：
```
UIButton *btn = [[UIButton alloc] initWithFrame:CGRectMake(100,100,100,100)];
[btn setTitle:@"Test" forState:UIControlStateNormal];
btn.backgroundColor = [UIColor lightGrayColor];
self.assignBtn = btn;
self.weakButton = btn;
```
此时打印两个button，没有区别。释放button：
```
btn = nil;
```
释放之后打印self.weakBtn和self.assignBtn
```
NSLog(@"self.weakBtn = %@",self.weakButton);
NSLog(@"self.assignBtn = %@",self.assignBtn);
```
运行，执行到self.assignBtn的时候崩溃了，崩溃信息是
```
 EXC_BAD_ACCESS (code=EXC_I386_GPFLT)
```
weak和assign修饰对象时的差别体现出来了。

weak修饰的对象，当对象释放之后，即引用计数为0时，对象会置为nil
```
2018-12-06 16:17:05.774298+0800 TestClock[15863:192570] self.weakBtn = (null)
```
而向nil发送消息是没有问题的，不会崩溃。

assign修饰的对象，当对象释放之后，即引用计数为0时，对象会变为野指针，不知道指向哪，再向该对象发消息，非常容易崩溃。

因此，当属性类型是对象时，不要使用assign，会带来一些风险。
#### 堆和栈
上面说到，属性用assign修饰，当被释放后，容易变为野指针，容易带来崩溃问题，那么，为何基本数据类型可以用assign来修饰呢？这就涉及到堆和栈的问题。

相对来说，堆的空间大，通常是不连续的结构，使用链表结构。使用堆中的空间，需要开发者自己去释放。OC中的对象，如 UIButton 、UILabel ，[[UIButton alloc] init] 出来的，都是分配在堆空间上。

栈的空间小，约1M左右，是一段连续的结构。栈中的空间，开发者不需要管，系统会帮忙处理。iOS开发 中 int、float等变量分配内存时是在栈上。如果栈空间使用完，会发生栈溢出的错误。

由于堆、栈结构的差异，栈和堆分配空间时的寻址方式也是不一样的。因为栈是连续的控件，所以栈在分配空间时，会直接在未使用的空间中分配一段出来，供程序使用；如果剩下的空间不够大，直接栈溢出；堆是不连续的，堆寻找合适空间时，是顺着链表结点来寻找，找到第一块足够大的空间时，分配空间，返回。根据两者的数据结构，可以推断，堆空间上是存在碎片的。

回到问题，为何assign修饰基本数据类型没有野指针的问题？因为这些基本数据类型是分配在栈上，栈上空间的分配和回收都是系统来处理的，因此开发者无需关注，也就不会产生野指针的问题。
#### 栈是线程安全的嘛
扩展一下，栈是线程安全的嘛？回答问题之前，先看一下进程和线程的关系。
##### 进程和线程的关系
线程是进程的一个实体，是CPU调度和分派的基本单位。一个进程可以拥有多个线程。线程本身是不配拥有系统资源的，只拥有很少的，运行中必不可少的资源（如程序计数器、寄存器、栈）。但是线程可以与同属于一个进程的其他线程，共享进程所拥有的资源。一个进程中所有的线程共享该进程的地址空间，但是每个线程有自己独立的栈，iOS系统中，每个线程栈的大小是1M。而堆则不同。堆是进程所独有的，通常一个进程有一个堆，这个堆为本进程中的所有线程所共享。
##### 栈的线程安全
其实通过上面的介绍，该问题答案已经很明显了：栈是线程安全的。

堆是多个线程所共有的空间，操作系统在对进程进行初始化的时候，会对堆进行分配；
栈是每个线程所独有的，保存线程的运行状态和局部变量。栈在线程开始的时化，每个线程的栈是互相独立的，因此栈是线程安全的。

### copy、strong、mutableCopy
属性修饰符中，还有一个经常被问到的面试题是copy和strong。什么时候用copy，为什么？什么时候用strong,为什么？以及mutableCopy又是什么？这一节介绍一下这些内容。

#### copy和strong
首先看一下copy和strong，copy和strong的区别也是面试中出现频率最高的。之前举得例子中其实已经出现了copy和strong：
```
@property (nonatomic, copy) NSString *sex;

@property (nonatomic, strong) NSMutableArray *books;
```
通常情况下，**不可变对象属性修饰符使用copy，可变对象属性修饰符使用strong**。
##### 可变对象和不可变对象
Objective-C中存在可变对象和不可变对象的概念。像NSArray、NSDictionary、NSString这些都是不可变对象，像NSMutableArray、NSMutableDictionary、NSMutableString这些是可变对象。可变对象和不可变对象的区别是，不可变对象的值一旦确定就不能再修改。下面看个例子来说明。
```
- (void)testNotChange
{
    NSString *str = @"123";
    NSLog(@"str = %p",str);
    str = @"234";
    NSLog(@"after str = %p",str);
}
```
NSString是不可变对象。虽然在程序中修改了str的值，但是此处的修改实际上是系统重新分配了空间，定义了字符串，然后str重新指向了一个新的地址。这也是为何修改之后地址不一致的原因：
```
2018-12-06 22:02:41.350812+0800 TestClock[884:17969] str = 0x106ec1290
2018-12-06 22:02:41.350919+0800 TestClock[884:17969] after str = 0x106ec12d0
```
再来看可变对象的例子：
```
- (void)testChangeAble
{
    NSMutableString *mutStr = [NSMutableString stringWithString:@"abc"];
    NSLog(@"mutStr = %p",mutStr);
    [mutStr appendString:@"def"];
    NSLog(@"after mutStr = %p",mutStr);
}
```
NSMutableString是可变对象。程序中改变了mutStr的值，且修改前后mutStr的地址一致：
```
2018-12-06 22:10:08.457179+0800 TestClock[1000:21900] mutStr = 0x600002100540
2018-12-06 22:10:08.457261+0800 TestClock[1000:21900] after mutStr = 0x600002100540
```
##### 不可变对象用strong
上面说了，可变对象使用strong，不可变对象使用copy。那么，如果不可变对象使用strong来修饰，会有什么问题呢？写代码测试一下：
```
@property (nonatomic, strong) NSString *strongStr;
```
首先明确一点，既然类型是NSString，那么则代表我们不希望testStr被改变，否则直接使用可变对象NSMutableString就可以了。另外需要提醒的一点是，NSMutableString是NSString的子类，对继承了解的应该都知道，子类是可以用来初始化父类的。

介绍完之后，来看一段代码。
```
- (void)testStrongStr
{
    NSString *tempStr = @"123";
    NSMutableString *mutString = [NSMutableString stringWithString:tempStr];
    self.strongStr = mutString;  // 子类初始化父类
    NSLog(@"self str = %p  mutStr = %p",self.strongStr,mutString);   // 两者指向的地址是一样的
    [mutString insertString:@"456" atIndex:0];
    NSLog(@"self str = %@  mutStr = %@",self.strongStr,mutString);  // 两者的值都会改变,不可变对象的值被改变
}
```
注意：**我们定义的不可变对象strongStr，在开发者无感知的情况下被篡改了。**所谓无感知，是因为开发者没有显示的修改strongStr的值，而是再修改其他变量的值时，strongStr被意外的改变。这显然不是我们想得到的，而且也是危险的。项目中出现类似的bug时，通常都很难定位。这就是不可变对象使用strong修饰所带来的风险。
##### 可变对象用copy
上面说了不可变对象使用strong的问题，那么可变对象使用copy有什么问题呢？还是写代码来验证一下：
```
@property (nonatomic, copy) NSMutableString *mutString;
```
这里还是强调一下，既然属性类型是可变类型，说明我们期望再程序中能够改变mutString的值，否则直接使用NSString了。

看一下测试代码：
```
- (void)testStrCopy
{
    NSString *str = @"123";
    self.mutString = [NSMutableString stringWithString:str];
    NSLog(@"str = %p self.mutString = %p",str,self.mutString); // 两者的地址不一样
    [self.mutString appendString:@"456"]; // 会崩溃，因为此时self.mutArray是NSString类型，是不可变对象
}
```
执行程序后，会崩溃，崩溃原因是：
```
[NSTaggedPointerString appendString:]: unrecognized selector sent to instance 0xed877425eeef9883
```
即 self.mutString没有appendString方法。self.mutString是NSMutableString类型，为何没有appendString方法呢？这就是使用copy造成的。看一下
```
self.mutString = [NSMutableString stringWithString:str];
```
这行代码到底发生了什么。这行代码实际上完成了两件事：
```
// 首先声明一个临时变量
NSMutableString *tempString = [NSMutableString stringWithString:str];
// 将该临时变量copy，赋值给self.mutString
self.mutString = [tempString copy];
```
注意，**通过[tempString copy]得到的self.mutString是一个不可变对象**，不可变对象自然没有appendString方法，这也是为何会崩溃的原因。
#### copy和mutableCopy
另外常用来做对比的是copy和mutableCopy。copy和mutableCopy之间的差异主要和深拷贝和浅拷贝有关，先看一下深拷贝、浅拷贝的概念。
##### 深拷贝、浅拷贝
所谓浅拷贝，在Objective-C中可以理解为引用计数加1，并没有申请新的内存区域，只是另外一个指针指向了该区域。深拷贝正好相反，深拷贝会申请新的内存区域，原内存区域的引用计数不变。看图来说明深拷贝和浅拷贝的区别。

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE25d9c9baa17df5d888dfc60b9078d459/15441)

首先A指向一块内存区域，现在设置B = A

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCEafc186bcc0098e08a1b5ffc1fbacbe6e/15443)

现在B和A指向了同一块内存区域，即为浅拷贝。

再来看深考贝

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE6154bd93ef15e2b1e2ac67634a50b798/15447)

首先A指向一块内存区域，现在设置B = A

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE6154bd93ef15e2b1e2ac67634a50b798/15447)

A和B指向的不是同一块内存区域，只是这两块内存区域中的内容是一样的，即为深拷贝。
##### 可变对象的copy、mutableCopy
可变对象的copy和mutableCopy都是深拷贝。以可变对象NSMutableString和NSMutableArray为例，测试代码：
```
- (void)testMutableCopy
{
    NSMutableString *str1 = [NSMutableString stringWithString:@"abc"];
    NSString *str2 = [str1 copy];
    NSMutableString *str3 = [str1 mutableCopy];
    NSLog(@"str1 = %p str2 = %p str3 = %p",str1,str2,str3);
    
    NSMutableArray *array1 = [NSMutableArray arrayWithObjects:@"a",@"b", nil];
    NSArray *array2 = [array1 copy];
    NSMutableArray *array3 = [array1 mutableCopy];
    NSLog(@"array1 = %p array2 = %p array3 = %p",array1,array2,array3);
}
```
输出结果：
```
2018-12-07 13:01:27.525064+0800 TestClock[9357:143436] str1 = 0x60000086d8f0 str2 = 0xc8c1a5736a50d5fe str3 = 0x60000086d9b0
2018-12-07 13:01:27.525198+0800 TestClock[9357:143436] array1 = 0x600000868000 array2 = 0x60000067e5a0 array3 = 0x600000868030
```
可以看到，只要是可变对象，无论是集合对象，还是非集合对象，copy和mutableCopy都是深拷贝。
##### 不可变对象的copy、mutableCopy
不可变对象的copy是浅拷贝，mutableCopy是深拷贝。以NSString和NSArray为例，测试代码如下：
```
- (void)testCopy
{
    NSString *str1 = @"123";
    NSString *str2 = [str1 copy];
    NSMutableString *str3 = [str1 mutableCopy];
    NSLog(@"str1 = %p str2 = %p str3 = %p",str1,str2,str3);
    
    NSArray *array1 = @[@"1",@"2"];
    NSArray *array2 = [array1 copy];
    NSMutableArray *array3 = [array1 mutableCopy];
    NSLog(@"array1 = %p array2 = %p array3 = %p",array1,array2,array3);
}
```
输出结果：
```
2018-12-07 13:06:29.439108+0800 TestClock[9442:147133] str1 = 0x1045612b0 str2 = 0x1045612b0 str3 = 0x6000017e4450
2018-12-07 13:06:29.439236+0800 TestClock[9442:147133] array1 = 0x6000019f5c80 array2 = 0x6000019f5c80 array3 = 0x6000017e1170
```
可以看到，只要是不可变对象，无论是集合对象，还是非集合对象，copy都是浅拷贝，mutableCopy都是深拷贝。
#### 自定义对象如何支持copy方法
项目开发中经常会有自定义对象的需求，那么自定义对象是否可以copy呢？如何支持copy？

自定义对象可以支持copy方法，我们所需要做的是：自定义对象遵守NSCopying协议，且实现copyWithZone方法。NSCopying协议是系统提供的，直接使用即可。

遵守NSCopying协议：
```
@interface Student : NSObject <NSCopying>
{
    NSString *_sex;
}

@property (atomic, copy) NSString *name;

@property (nonatomic, copy) NSString *sex;

@property (nonatomic, assign) int age;

@end
```
实现CopyWithZone方法：
```
- (instancetype)initWithName:(NSString *)name age:(int)age sex:(NSString *)sex
{
    if(self = [super init]){
        self.name = name;
        _sex = sex;
        self.age = age;
    }
    return self;
}

- (instancetype)copyWithZone:(NSZone *)zone
{
    // 注意，copy的是自己，因此使用自己的属性
    Student *stu = [[Student allocWithZone:zone] initWithName:self.name age:self.age sex:_sex];
    return stu;
}
```
测试代码：
```
- (void)testStudent
{
    Student *stu1 = [[Student alloc] initWithName:@"Wang" age:18 sex:@"male"];
    Student *stu2 = [stu1 copy];
    NSLog(@"stu1 = %p stu2 = %p",stu1,stu2);
}
```
输出结果：
```
stu1 = 0x600003a41e60 stu2 = 0x600003a41fc0
```
这里是一个深拷贝，根据copyWithZone方法的实现，应该很容易明白为何是深拷贝。

除了NSCopying协议和copyWithZone方法，对应的还有NSMutableCopying协议和mutableCopyWithZone方法，实现都是类似的，不做过多介绍。
