## app启动时间优化
## 相关知识点整理
### .got和.plt
.got，就是Global Offset Table,全局偏移表。

.plt，就是Procedure Linkage Table，进程链接表。

完。

### 寄存器
寄存器是CPU内的硬件，是CPU的组成部分。寄存器的实质是存储器，类似于硬盘、内存都是存储器，寄存器数量非常少，但是速度非常快，是系统操作数据的最快途径。寄存器可以用来暂存指令、数据和地址。

寄存器通常以它们可以保存的bit数量来估量，比如说，一个8位的寄存器或者一个32位的寄存器。X86架构的CPU有8个整数寄存器和8个浮点数寄存器；X86_64架构的CPU有16个整数寄存器和16个浮点数寄存器；arm架构的CPU通常有16个整数寄存器和16个浮点数寄存器。

完。

### 地址空间布局随机化(ASLR)
地址空间布局随机化，英文为 Address space layout randomization,缩写为ASLR。是一种防范内存损坏漏洞被利用的计算机安全技术。ASLR通过随机的放置进程关键数据区域的地址空间来防止攻击者能可靠的跳转到内存的特定位置来利用函数。现代的操作系统一般都加了这一机制。

完。
### 内核态 && 用户态
因为操作系统的资源是有限的，如果访问资源的操作过多，必然会消耗过多的资源，而且如果不进行区分，还有可能造成资源访问的冲突。为了减少访问资源的操作和冲突，Unix/Linux的设计哲学之一是：对不同的操作赋予不同的执行等级，也就是特权的概念。Intel的X86架构的CPU提供了0到3四个特权级，数字越小，权限越高。Linux操作系统中使用了0和3两个特权级，分别对应内核态和用户态。

运行于用户态的进程可以执行的操作和访问的资源会受到极大的限制，而运行与内核态的进程则可以执行更高特权的操作。通常情况下，程序是运行于用户态，但是在执行过程中，部分操作会切换到内核态执行，这就涉及到用户态和内核态之间的切换。比如说，C语言中的malloc()函数，其底层使用的是sbrk()系统函数，当malloc()调用sbrk()时，就涉及到了一次从用户态到内核态的切换。

从用户态切换到内核态的情况有三种：

1. 系统调用，如刚才所说的malloc()函数
2. 异常事件。当CPU在执行用户态程序时，突然发生了某些异常事件，就会出发从用户态切换到内核态执行相关的异常事件，比如缺页异常。
3. 外围设备的中断。当外围设备完成用户的请求操作后，会向CPU发出中断信号，此时，CPU就会暂停执行下一条即将要执行的指令，转而去执行终端信号对应的处理程序。如果先前执行的指令在用户态下，就会发生用户态向内核态的切换。

完。

### launchd
#### 计算机系统启动过程
计算机系统的启动分为两个过程，第一步是底层硬件固件程序的运行以加载操作系统内核；第二步是操作系统接管之后相关进程的启动过程。

大部分PC引导程序使用BIOS(Basic Input Output System),计算机通电后第一件事情就是读取ROM中的BIOS程序。BIOS做的第一件事情是硬件检测，检测各硬件是否正常。如果硬件不正常，则终止启动。硬件检测没问题，BIOS将控制权交给MBR（Master Boot Record,主引导记录），MBR通过分区表找到操作系统加载器代码，并将控制权交给加载器。之后，操作系统的内核被载入。以Linux为例，内核载入成功后，开始运行第一个程序init，用于初始化系统环境。由于init是第一个运行程序，它的PID为1，其它进程都是从它衍生出来的后代。init运行起来后，负责加载运行各种开机启动程序（守护进程）。
#### iOS系统启动过程
iOS系统的启动过程和PC的启动过程不太一样，是苹果自创的一套引导流程。iOS的启动分为三种模式，分别是正常启动，恢复模式启动，固件更新模式启动。这里介绍一下正常开机启动的流程。

当按下手机电源键之后，如果没有其他交互，iOS设备正常启动需要经历一下的过程：
> 引导ROM -> LLB -> iBoot -> 加载内核 -> 启动launchd -> 启动守护程序和代理程序

引导ROM负责初始化设备，并加载底层引导加载器（Low Level Bootloader，LLB）。ROM属于硬件设备的一部分，无法更新。LLB负责定位并加载iBoot,如果查找iBoot失败，LLB将放弃加载并切换到固件更新模式引导。iBoot是引导过程中的加载器，负责加载操作系统内核。内核被载入后，将启动第一个程序launchd,launchd相当于上面提到的Linux中的init程序，是iOS系统启动的第一个进程。之后，由launchd启动守护程序和代理程序，直到桌面应用程序SprintBoard运行，系统启动完成。
#### launchd
launchd由操作系统内核启动，用户没有权限去进行手动启动。在Mac OS中，launchd的实例不止一个，当用户登录后，将会从系统中的launchd实例fork出一个属于用户范畴的launchd实例。由于iOS系统不需要登录，所以只有一个launchd实例，并且它是系统运行期间唯一不能终止的进程，当系统关闭时，它作为最后一个进程退出。

在iOS/Mac OS上，launchd是怎样来启动守护进程的呢？实际上是通过查看特定文件夹中的plist文件，根据这些plist文件来决定启动哪些程序。特定的文件夹目录有：/System/Library/LaunchDaemons、/System/Library/LaunchAgents等。

完。

### iOS开发中的静态库、动态库、framework
日常开发中会用到各种各样的库，系统提供的库，第三方库等。库到底是什么？有哪些类型的库？

库可以理解成一个可执行文件，加上相应的头文件之后，开发者就可以直接使用。项目中，可以将一些变动很小的基础功能进行封装，打包成库，这样一方面可以减少编译时间（因为库是已经编译好的），另一方面，也可以隐藏代码的实现。

iOS开发中，库的类型有三种：静态库、动态库、framework。
#### 静态库
静态库全称静态链接库，如Windows系统下的.lib文件，iOS系统下的.a文件，都是静态库。静态库在编译时，就会被复制到目标程序里，可以理解成在目标程序里复制了一份静态库的代码。显而易见的，静态库会增加安装包的大小。而且，如果不同的应用程序使用了相同的静态库，每个应用程序内部都会有该静态库。

项目中使用到静态库的情景是非常多的。比如说微信分享sdk，微博分享sdk，使用的都是静态库的形式。

iOS系统下，静态库是.a文件。通常情况下，一个静态库只有.a文件是不够的，因为静态库要提供给其他人使用，需要告诉其他人静态库提供的接口，因此还需要头文件.h文件。比如说，微博分享静态库如下：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE0190c6651244df2a765bc8473fb9c936/8418)

微信分享静态库如下：
![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE5c58ba6962c15b96e10d646cdae03d8e/8423)
#### 动态库
动态库全称动态链接库，如windows系统下的.dll文件，iOS、Mac OS系统下的.dylib、.tbd文件，都是动态链接库。相对于静态库来说，在编译时，动态库并不会被复制到目标程序中，只是在目标程序中保留了一个动态库的索引。等到程序运行时，动态库才会被真正的加载到内存中。

和静态库相比，动态库不会增大安装包的体积，而且多个应用程序可以使用同一个动态库。

Mac OS系统下，在/usr/lib目录下，存放了大量供系统与应用程序调用的动态库文件。
![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE7aaa0aea3cfc4edf074cbb0c787fa80f/8450)
#### framework
除了静态库和动态库外，日常开发中还会使用到大量的framework,framework和静态库、动态库相比有什么区别呢？

framework是一种打包方式，将静态库、动态库的可执行文件、头文件，以及相关的资源文件打包到一个包中，方便管理。因此，framework既可以表示静态库，也可以表示动态库。系统提供了很多的framework供开发者日常使用，如UIKit.framework、WebKit.framework、ImageIO.framework等。由上面的介绍，很容易猜到，系统提供的framework都是动态库。项目中当然也可以创建自己的framework,开发者创建的framework通常是静态库。项目中使用到的一些framework：
![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE6b457c8882cd35bc8c5ae10952ee1bcd/8472)

framework通常会包含该库对应的头文件，
![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE2b5ce0d7e1d49dec1bfd793d763809e6/8481)
#### 总结
上面介绍了iOS开发中的库，项目开发中，使用自定义静态库的情况比较多，如.a库，framework静态库等；无论是.dylib文件，还是framework动态库，动态库通常是系统提供。

完。
### 操作系统&&操作系统内核
#### 操作系统
操作系统（operating system）是管理计算机硬件和软件资源的计算机程序。操作系统需要负责输入输出、管理文件系统、操作网络、分配内存、决定系统资源工序的优先次序等工作。

除了上述提到的功能外，有的操作系统还提供了图形界面，方便用户与操作系统交互，如Mac操作系统，也有操作系统仅仅提供了命令行界面，供用户与操作系统交互，如Linux操作系统。
#### 操作系统内核
操作系统内核是操作系统的一部分，内核程序（kernel）主要负责和硬件交互，直接与硬件打交道。一个计算机中，由于硬件标准不同，厂家不同，直接对硬件操作是非常复杂的，内核通常提供一种硬件抽象的方法，来完成这些操作。

计算机硬件、操作系统、系统内核、应用程序之间的结构可以表示为：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE186f7a3916924d5299a14595e017fba8/8529)

完。

### macOS && iOS系统结构
#### 系统名称介绍
macOS，2012年前称之为Mac OS X，2012 - 2016年称OS X，2016年起称macOS，是苹果公司推出的图形用户界面操作系统。iOS，原名iPhone OS，后改为iOS。是苹果公司为移动设备所开发的移动操作系统。无论是iOS还是macOS,都是类Unix操作系统。
#### 系统名词解释
在学习iOS、macOS系统结构时，如果没有充分的了解，常常会被一些名词给搞混，这里对一些不常用到的名词做一下解释。
##### Darwin
Darwin（达尔文）是苹果公司于2000年发布的一个开源操作系统，Darwin是macOS 和 iOS的一部分。可以将Darwin理解为操作系统的代号。Darwin由XNU和一些其他的Darwin库组成。
##### XNU
XNU是由苹果公司发布的操作系统内核，即Darwin的内核是XNU，是Darwin操作系统的一部分。除macOS外，XNU还是iOS、tvOS、watchOS操作系统的内核。XNU是X is not Unix的缩写。XNU包含三部分：Mach内核、BSD、I/O Kit。
##### Mach内核
XNU内核以一个被深度定制的Mach3.0内核作为基础。Mach是一个由卡内基梅隆大学开发的计算机操作系统微内核，主要是为了用于操作系统研究，特别是在分布式与并行运算上。XNU中的Mach所负责的功能非常少（核心功能），只能完成操作系统最基本的职责，比如任务调度、消息传递、进程间通信等。
##### BSD
BSD，伯克利软件套件（Berkeley Software Distribution）,也被称为伯克利Unix(Berkeley Unix)，是一个操作系统的名称。XNU中的BSD部分提供了POSIX应用程序接口（BSD系统称之为API）:进程模型、网络协议栈、虚拟文件系统等。
##### I/O Kit
I/O Kit是一个设备驱动框架，为开发者提供了开发设备驱动程序的API。
#### 易混淆名词解释
除上面提到的名词外，还有一些易混淆的名词，这里也做一下解释。
##### 用户体验层
用户体验层又被称为应用层，主要包括用户能够接触到的图形应用，如SprintBoard等。
##### 应用框架层
应用框架层即Cocoa层，就是开发人员能够接触到的Cocoa等框架。
##### 核心框架层
核心框架层包括各种核心架构、OpenGL等。

核心框架层、应用框架层、用户体验层均位于Darwin之上。
#### 系统结构
根据官方文档介绍，整个系统可以分为上面提到的4个层次：Darwin、核心框架层、应用框架层、用户体验层。整个系统的结构可以表示为下图：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE38452de952d72412d18494f90147cdeb/10199)

完。

### Mach-O文件
在Linux操作系统上，以及很多的类Unix操作系统，可执行文件的格式是ELF。mac OS虽然也是类Unix操作系统，然而mac OS以及iOS上，可执行文件的格式是Mach-O。

先理清一个易混淆的点，Mach-O和Mac没有什么关系。Mac是苹果电脑Macintosh的简称，而Mach是一种操作系统微内核，苹果公司的设备上操作系统内核使用的是Mach。在Mach内核中，一种可执行文件格式是Mach-O。所以不要被Mach-O和Mac相似的名字迷惑了，实际上两者的关系不大。

在介绍Mach-O文件格式之前，首先了解一下通用二进制格式。
#### 通用二进制格式
通用二进制格式（Universal Binary），又称为胖二进制(Fat Binary)。通用二进制文件实际上就是将支持不同CPU架构的二进制文件打包成一个文件，系统在加载运行时，会根据通用二进制文件中提供的架构，选择和当前系统匹配的二进制文件。因此，很多人认为，将通用二进制文件称为胖二进制文件更为合适。

mac OS中自带了很多的通用二进制文件，使用file命令可以查看这些通用二进制文件的信息，比如使用file命令查看python的信息：
```shell
file /Users/.../Desktop/python
/Desktop/python: Mach-O universal binary with 2 architectures: [x86_64:Mach-O 64-bit executable x86_64] [i386:Mach-O executable i386]
/Desktop/python (for architecture x86_64):	Mach-O 64-bit executable x86_64
/Desktop/python (for architecture i386):	Mach-O executable i386
```
可以看到，python通用二进制文件包含两种架构的Mach-O文件，分别是x86_64架构和i386架构。

看一下代码中是如何定义通用二进制文件的，在/usr/include/mach-o目录下有通用二进制相关的文件。在fat.h中可以看到通用二进制文件头部结构fat_header的定义（从文件名的角度来看，通用二进制文件称为胖二进制文件也更为合适）：
```Objective-C
#define FAT_MAGIC	0xcafebabe
#define FAT_CIGAM	0xbebafeca	/* NXSwapLong(FAT_MAGIC) */

struct fat_header {
	uint32_t	magic;		/* FAT_MAGIC or FAT_MAGIC_64 */
	uint32_t	nfat_arch;	/* number of structs that follow */
};
```
magic是一个固定的值，值是0xcafebabe或0xbebafeca，表示这是一个通用二进制文件；nfat_arch表示的是该通用二进制文件包含多少个架构文件（也就是Mach-O文件）。

在fat_header之后紧跟着的是多个fat_arch结构体，fat_arch的定义如下：
```Objective-C
struct fat_arch_64 {
	cpu_type_t	cputype;	/* cpu specifier (int) */
	cpu_subtype_t	cpusubtype;	/* machine specifier (int) */
	uint64_t	offset;		/* file offset to this object file */
	uint64_t	size;		/* size of this object file */
	uint32_t	align;		/* alignment as a power of 2 */
	uint32_t	reserved;	/* reserved */
};
```
其中，cputype指定了cpu的类型,cpusubtype指定了cpu的子类型，offset指定了该架构数据相对于文件开头的偏移量，size指定了该架构数据的大小，align指定了数据的内存对齐边界，值必须是2的次方，reserved是保留字段，只有64位架构的有，32位架构的无此字段。

cputype的部分取值有：
```
#define CPU_TYPE_X86		((cpu_type_t) 7)
#define CPU_TYPE_I386		CPU_TYPE_X86		
#define	CPU_TYPE_X86_64		(CPU_TYPE_X86 | CPU_ARCH_ABI64)
#define CPU_TYPE_ARM		((cpu_type_t) 12)
#define CPU_TYPE_ARM64          (CPU_TYPE_ARM | CPU_ARCH_ABI64)
#define CPU_TYPE_POWERPC		((cpu_type_t) 18)
#define CPU_TYPE_POWERPC64		(CPU_TYPE_POWERPC | CPU_ARCH_ABI64
```
cpusubtype的部分取值有：
```
#define CPU_SUBTYPE_X86_ALL		((cpu_subtype_t)3)
#define CPU_SUBTYPE_X86_64_ALL		((cpu_subtype_t)3)
#define CPU_SUBTYPE_X86_ARCH1		((cpu_subtype_t)4)
#define CPU_SUBTYPE_X86_64_H		((cpu_subtype_t)8)
```
使用MachOview软件看一下python的信息：
![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCEd94c8521af3133f3caca1d2bf08e3d73/9208)
可以清楚的看到，Fat Header里面的内容：

首先是fat_header，包含两种架构，后面跟着两个fat_arch结构。从图中可以看到，Fat Header之后就是两个Executable文件，也就是可执行文件，也就是Mach-O文件。
#### Mach-O格式
Mach-O是mac OS系统可执行文件的格式，平时用到的可执行文件，动态库，静态库，Dsym文件，都是Mach-O格式的文件。看一下苹果官方文档中对Mach-O文件的介绍：
![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE152f522e6b926f49e5e516ebe6ab06e2/9228)

可以看到，一个Mach-O文件包含三部分：Header、Load commands、Data。
##### Mach-O Header
Mach-O头部，描述了Mach-O的cpu类型，文件类型以及加载命令大小、条数等信息。还是使用MachOview看一下python可执行文件中的Mach-O文件：
![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE91fe71271ae4e28a455526b5bbd8f787/9245)

通过MachOview可以得到Mach-O header中包含的信息，包含了Magic Number、Cpu type、Cpu subtype、file type等。看一下代码中对于Mach-O header的定义，相关的代码在mach-o/loader。h中：
```
struct mach_header_64 {
	uint32_t	magic;		/* mach magic number identifier */
	cpu_type_t	cputype;	/* cpu specifier */
	cpu_subtype_t	cpusubtype;	/* machine specifier */
	uint32_t	filetype;	/* type of file */
	uint32_t	ncmds;		/* number of load commands */
	uint32_t	sizeofcmds;	/* the size of all the load commands */
	uint32_t	flags;		/* flags */
	uint32_t	reserved;	/* reserved */
};

/* Constant for the magic field of the mach_header_64 (64-bit architectures) */
#define MH_MAGIC_64 0xfeedfacf /* the 64-bit mach magic number */
#define MH_CIGAM_64 0xcffaedfe /* NXSwapInt(MH_MAGIC_64) */
```
magic字段是一个固定的值，为0xfeedfacf或者0xcffaedfe，表示的是这是一个Mach-O格式的文件。

cpu_type_t 和 cpu_subtype_t和上文中提到的一样，这里不再介绍。

filetype表示Mach-O文件的具体类型，它的部分取值如下：
```
#define	MH_OBJECT	0x1		/* relocatable object file */
#define	MH_EXECUTE	0x2		/* demand paged executable file */
#define	MH_PRELOAD	0x5		/* preloaded executable file */
#define	MH_DYLIB	0x6		/* dynamically bound shared library */
#define	MH_DYLINKER	0x7		/* dynamic link editor */
#define	MH_BUNDLE	0x8		/* dynamically bound bundle file */
#define	MH_DSYM		0xa		/* companion file with only debug sections */
```
这里python的Mach-O文件类型是MH_EXECUTE，如果我们查看的是Dsym文件，则文件类型是MH_DSYM。

ncmds 表示的是Mach-O文件中加载命令的数量。

sizeofcmds 表示的是Mach-O文件加载命令的大小。

flags 表示文件标志。

reserved 是保留字段，64位cpu架构特有。

##### Mach-O Load commands
Mach-O Header之后就是Load commands,也就是加载命令。加载命令的作用时，在Mach-O文件被加载到内存时，加载命令告诉内核加载器或者动态链接器如何调用。还是先试用MachOview看一下Mach-O文件中的加载命令：
![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE63ede155ce6a9c8a22cc984ea2d2c20f/9292)

可以看到，Mach-O文件中有多条加载命令，加载命令的前两个字段分别是Command和Command Size。load command的数据结构定义如下：
```
struct load_command {
	uint32_t cmd;		/* type of load command */
	uint32_t cmdsize;	/* total size of command in bytes */
};
```
cmdsize字段表示当前加载命令的大小。

cmd字段代表当前加载命令的类型，加载命令的类型不同，结构体就不同。对于不同类型的加载命令，他们都会在load_command结构体后面加上一个或者多个字段来表示自己特定的结构体信息。加载命令的类型比较多，其部分取值如下：
```
#define	LC_SEGMENT	0x1	/* segment of this file to be mapped */
#define	LC_THREAD	0x4	/* thread */
#define	LC_UNIXTHREAD	0x5	/* unix thread (includes a stack) */
#define LC_PREPAGE      0xa     /* prepage command (internal use) */
#define	LC_DYSYMTAB	0xb	/* dynamic link-edit symbol table info */
#define	LC_LOAD_DYLIB	0xc	/* load a dynamically linked shared library */
#define LC_CODE_SIGNATURE 0x1d   /* local of code signature */
……
```
LC_SEGMENT是一个段加载命令；LC_LOAD_DYLIB表示这事一个需要动态加载的链接库；LC_CODE_SIGNATURE和签名、加密有关。

上图中看到加载命令是LC_SEGMENT_64，表示的是将64位的段映射到进程的地址空间。可以看一下段加载命令的数据结构：
```
struct segment_command_64 { /* for 64-bit architectures */
	uint32_t	cmd;		/* LC_SEGMENT_64 */
	uint32_t	cmdsize;	/* includes sizeof section_64 structs */
	char		segname[16];	/* segment name */
	uint64_t	vmaddr;		/* memory address of this segment */
	uint64_t	vmsize;		/* memory size of this segment */
	uint64_t	fileoff;	/* file offset of this segment */
	uint64_t	filesize;	/* amount to map from the file */
	vm_prot_t	maxprot;	/* maximum VM protection */
	vm_prot_t	initprot;	/* initial VM protection */
	uint32_t	nsects;		/* number of sections in segment */
	uint32_t	flags;		/* flags */
};
```
cmd 和 cmdsize上面已经介绍过了，不再重复。

segname字段表示的是该segment的名称。

vmaddr字段表示段的虚拟内存地址。

vmsize字段表示段所占的虚拟内存的大小。

fileoff字段表示段数据在文件中的偏移。

filesize字段表示段数据的实际大小。

maxprot字段表示页面所需要的最高内存保护。

initprot字段表示页面初始的内存保护。

nsects字段表示该segment包含了多少个section(节区)，一个段可以包含0个或者多个section。

flags字段是段的标志信息。
###### section的结构
来简单看一下section的数据结构：
```
struct section_64 { /* for 64-bit architectures */
	char		sectname[16];	/* name of this section */
	char		segname[16];	/* segment this section goes in */
	uint64_t	addr;		/* memory address of this section */
	uint64_t	size;		/* size in bytes of this section */
	uint32_t	offset;		/* file offset of this section */
	uint32_t	align;		/* section alignment (power of 2) */
	uint32_t	reloff;		/* file offset of relocation entries */
	uint32_t	nreloc;		/* number of relocation entries */
	uint32_t	flags;		/* flags (section type and attributes)*/
	uint32_t	reserved1;	/* reserved (for offset or index) */
	uint32_t	reserved2;	/* reserved (for count or sizeof) */
	uint32_t	reserved3;	/* reserved */
};
```
sectname字段表示该section的name。

segname字段表示该section所属的segment的segmentName。

addr字段表示该section的内存起始地址。

size字段表示该section的大小。

offset字段表示该section相对文件的偏移量。

align字段表示字节区的内存对齐边界。

reloff表示重定位信息的文件偏移。

nreloc表示重定位条目的数目。

flags是section的一些标志属性。

使用MachOview看一下Section的信息：
![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE583f0d652279df3dee1abc6bb2151dbe/9361)
这是Mach-O文件中某个Section的信息，和section的数据结构是一一对应的。
##### Mach-O Data
Mach-O中Load Commands之后的就是Data数据。每个段的数据都保存在这里，这里存放了具体的数据与代码。由于Mach-O Data中的内容更多的与具体的数据有关，而与格式无关，因此就不做太多的介绍了。
##### 总结
关于Mach-O文件格式就全部介绍完了，很多的底层知识都和Mach-O有关，包括app启动过程，符号表解析，bitcode等。了解Mach-O格式，在涉及到一些底层原理时，能帮助我们更好的理解。

完。

### XNU加载Mach-O和dyld
我们知道，操作系统是电脑、手机上最基本的软件，任何其他的软件都必须在操作系统的支持下才能够运行。同理，软件的启动也必须在操作系统的支持下才能够运行。对于iOS系统来说，操作系统内核是XNU（X is not Unix），那么在一个app的启动过程中，XNU发挥了什么作用呢？本篇文章，我们来探究一下这个问题。
#### XNU启动launchd
XNU的代码是开源的，可以从苹果开源代码平台上下载XNU的代码，通过分析XNU的源码，可以大致了解XNU是如何加载Mach-O文件以及dyld的。

XNU内核启动后，启动的第一个进程是launchd，launchd启动之后会启动其他的守护进程。XNU启动launchd的过程是 load_init_program() -> load_init_program_at_path()。可以看一下这两个函数的源码。

load_init_program()部分代码：
```
void load_init_program(proc_t p)
{
	……

	error = ENOENT;
    // 核心代码在这里，加载初始化程序，对 init_programs数组遍历
	for (i = 0; i < sizeof(init_programs)/sizeof(init_programs[0]); i++) {
        // 调用load_init_program_at_path方法
		error = load_init_program_at_path(p, (user_addr_t)scratch_addr, init_programs[i]);
		if (!error)
			return;
	}

	panic("Process 1 exec of %s failed, errno %d", ((i == 0) ? "<null>" : init_programs[i-1]), error);
}
```
init_programs是一个数组，可以看一下该数组的定义：
```
// 内核的debug模式下可以加载供调试的launchd,非debug模式下，只加载launchd
// launchd负责进程管理
static const char * init_programs[] = {
#if DEBUG
	"/usr/local/sbin/launchd.debug",
#endif
#if DEVELOPMENT || DEBUG
	"/usr/local/sbin/launchd.development",
#endif
	"/sbin/launchd",
};
```
可以看出，load_init_program的作用就是加载launchd，加载launchd使用的方法是load_init_program_at_path函数。load_init_program_at_path的部分代码如下：
```
static int load_init_program_at_path(proc_t p, user_addr_t scratch_addr, const char* path)
{
    ……
	/*
	 * Set up argument block for fake call to execve.
	 */
	init_exec_args.fname = argv0;
	init_exec_args.argp = scratch_addr;
	init_exec_args.envp = USER_ADDR_NULL;

	/*
	 * So that init task is set with uid,gid 0 token
	 */
	set_security_token(p);
    // 会调用execve方法
	return execve(p, &init_exec_args, retval);
}
```
load_init_program_at_path调用了execve()函数，实际上，execve是加载Mach-O文件流程的入口函数。因为launchd进程比较特殊，所以多了两个方法。因此，接下来我们就从execve()函数开始分析。
#### XNU加载Mach-O
##### execve()函数
上面说到了，execve()函数是加载Mach-O文件的入口，看一下execve()函数做了哪些事情：
```
/*
 uap是对可执行文件的封装，uap->fname可以得到执行文件的文件名
 uap->argp 可以得到执行文件的参数列表
 uap->envp 可以得到执行文件的环境变量列表
 */
int execve(proc_t p, struct execve_args *uap, int32_t *retval)
{
	struct __mac_execve_args muap;

	muap.fname = uap->fname;
	muap.argp = uap->argp;
	muap.envp = uap->envp;
	muap.mac_p = USER_ADDR_NULL;
    // 调用了__mac_execve方法
	err = __mac_execve(p, &muap, retval);

	return(err);
}
```
可以看到，execve()函数的作用主要是进行了一些赋值，然后调用了__mac_execve()函数，看来核心操作在__max_execve()函数中。
##### __mac_execve()函数
__mac_execve()函数会使用fork_create_child()函数启动新进程，之后使用新的进程，生成新的task。__mac_execve()函数的主要功能就是干这个，之后就调用了exec_activate_image()函数。

__mac_execve()函数的部分代码：
```
int __mac_execve(proc_t p, struct __mac_execve_args *uap, int32_t *retval)
{
    // 新的task定义
	task_t new_task = NULL;
	boolean_t should_release_proc_ref = FALSE;
	boolean_t exec_done = FALSE;
	boolean_t in_vfexec = FALSE;
	void *inherit = NULL;

	context.vc_thread = current_thread();
	context.vc_ucred = kauth_cred_proc_ref(p);	/* XXX must NOT be kauth_cred_get() */
	
	/* Initialize the common data in the image_params structure */
    // 使用uap初始化imgp结构体中的一些通用数据
	imgp->ip_user_fname = uap->fname;
	imgp->ip_user_argv = uap->argp;
	imgp->ip_user_envv = uap->envp;
	imgp->ip_vattr = vap;
	imgp->ip_origvattr = origvap;
	imgp->ip_vfs_context = &context;
	imgp->ip_flags = (is_64 ? IMGPF_WAS_64BIT : IMGPF_NONE) | ((p->p_flag & P_DISABLE_ASLR) ? IMGPF_DISABLE_ASLR : IMGPF_NONE);
	imgp->ip_seg = (is_64 ? UIO_USERSPACE64 : UIO_USERSPACE32);
	imgp->ip_mac_return = 0;
	imgp->ip_cs_error = OS_REASON_NULL;

	uthread = get_bsdthread_info(current_thread());
	if (uthread->uu_flag & UT_VFORK) {
		imgp->ip_flags |= IMGPF_VFORK_EXEC;
		in_vfexec = TRUE;
	} else {
        // 创建进程和新的task
		imgp->ip_new_thread = fork_create_child(current_task(),
					NULL, p, FALSE, p->p_flag & P_LP64, TRUE);
		/* task and thread ref returned by fork_create_child */
		if (imgp->ip_new_thread == NULL) {
			error = ENOMEM;
			goto exit_with_error;
		}

		new_task = get_threadtask(imgp->ip_new_thread);
		context.vc_thread = imgp->ip_new_thread;
	}
    // 调用了exec_activate_image方法
	error = exec_activate_image(imgp);

	return(error);
}

```
##### exec_activate_image
exec_activate_image函数会按照可执行文件的格式，而执行不同的函数。目前有三种格式，单指令集可执行文件，多指令集可执行文件，shell 脚本。如果是多指令集的，最终还是会执行到单指令集所对应的函数。因为分析的是如何加载Mach-O文件，所以暂不考虑shell 脚本。来看一下exec_activate_image的内部实现：
```
// 根据二进制文件的不同格式，执行不同的内存映射函数
static int exec_activate_image(struct image_params *imgp)
{
encapsulated_binary:
	error = -1;
    // 核心在这里，循环调用execsw相应的格式映射的加载函数进行加载
	for(i = 0; error == -1 && execsw[i].ex_imgact != NULL; i++) {

		error = (*execsw[i].ex_imgact)(imgp);

		switch (error) {
		/* case -1: not claimed: continue */
		case -2:		/* Encapsulated binary, imgp->ip_XXX set for next iteration */
			goto encapsulated_binary;

		case -3:		/* Interpreter */
			imgp->ip_vp = NULL;	/* already put */
			imgp->ip_ndp = NULL; /* already nameidone */
			goto again;

		default:
			break;
		}
	}

	return (error);
}
```
execsw是一个数组，看一下这个数组的定义：
```
struct execsw {
	int (*ex_imgact)(struct image_params *);
	const char *ex_name;
} execsw[] = {
    // 单指令集的Mach-O
	{ exec_mach_imgact,		"Mach-o Binary" },
    // 多指令集的Mac-O exec_fat_imgact会先进行指令集分解，然后调用exec_mach_imgact
	{ exec_fat_imgact,		"Fat Binary" },
    // shell脚本
	{ exec_shell_imgact,		"Interpreter Script" },
	{ NULL, NULL}
};
```
可以看到，单指令集的Mach-O文件最终调用的函数是exec_mach_imgact()。exec_mach_imgact()函数中的一个重要功能是将Mach-O文件映射到内存。将Mach-O文件映射到内存的函数是load_machfile()，因此，在介绍exec_mach_imgact()函数之前，先介绍一下load_machfile()函数。
##### load_machfile
load_machfile()函数会为mach-o文件分配虚拟内存，并且计算mach-o文件和dyld随机偏移量的值。之后会调用解析mach-o文件的函数parse_machfile()。看一下load_machfile的部分代码：
```
load_return_t load_machfile(
	struct image_params	*imgp,
	struct mach_header	*header,
	thread_t 		thread,
	vm_map_t 		*mapp,
	load_result_t		*result
)
{
	struct vnode		*vp = imgp->ip_vp;
	off_t			file_offset = imgp->ip_arch_offset;
	off_t			macho_size = imgp->ip_arch_size;
	off_t			file_size = imgp->ip_vattr->va_data_size;
	pmap_t			pmap = 0;	/* protected by create_map */
	vm_map_t		map;
	load_result_t		myresult;
	load_return_t		lret;
	boolean_t enforce_hard_pagezero = TRUE;
	int in_exec = (imgp->ip_flags & IMGPF_EXEC);
	task_t task = current_task();
	proc_t p = current_proc();
	mach_vm_offset_t	aslr_offset = 0;
	mach_vm_offset_t	dyld_aslr_offset = 0;

	if (macho_size > file_size) {
		return(LOAD_BADMACHO);
	}

	result->is64bit = ((imgp->ip_flags & IMGPF_IS_64BIT) == IMGPF_IS_64BIT);
    // 为当前task分配内存
	pmap = pmap_create(get_task_ledger(ledger_task),
			   (vm_map_size_t) 0,
			   result->is64bit);
    // 创建虚拟内存映射空间
	map = vm_map_create(pmap,
			0,
			vm_compute_max_offset(result->is64bit),
			TRUE);
	
	/*
	 * Compute a random offset for ASLR, and an independent random offset for dyld.
	 */
	if (!(imgp->ip_flags & IMGPF_DISABLE_ASLR)) {
		uint64_t max_slide_pages;

		max_slide_pages = vm_map_get_max_aslr_slide_pages(map);
        
        // binary（mach-o文件）随机的ASLR
		aslr_offset = random();
		aslr_offset %= max_slide_pages;
		aslr_offset <<= vm_map_page_shift(map);
        
        // dyld 随机的ASLR
		dyld_aslr_offset = random();
		dyld_aslr_offset %= max_slide_pages;
		dyld_aslr_offset <<= vm_map_page_shift(map);
	}
	
    // 使用parse_machfile方法解析mach-o
	lret = parse_machfile(vp, map, thread, header, file_offset, macho_size,
	                      0, (int64_t)aslr_offset, (int64_t)dyld_aslr_offset, result,
			      NULL, imgp);

    // pagezero处理，64 bit架构，默认4GB
	if (enforce_hard_pagezero &&
	    (vm_map_has_hard_pagezero(map, 0x1000) == FALSE)) {
		{
			vm_map_deallocate(map);	/* will lose pmap reference too */
			return (LOAD_BADMACHO);
		}
	}

	vm_commit_pagezero_status(map);
	*mapp = map;
	return(LOAD_SUCCESS);
}
```
PAGEZERO是可执行程序的第一个段程序，用于捕获空指针异常，总是位于虚拟内存最开始的位置，大小和CPU的架构有关。在64位的CPU架构下，PAGEZERO的大小是4G。

可以看到，mach-o文件的随机偏移值和dyld的随机偏移值，其实就是获取了一个随机数，然后根据随机数进行了一些计算。在load_machfile()函数中已经为mach-o文件分配了虚拟内存，接下来看一下parse_machfile()函数做了哪些操作。
##### parse_machfile
parse_machfile()函数做的工作主要有3个：（1）Mach-O文件的解析，以及对每个segment进行内存分配；（2）dyld的加载（3）dyld的解析以及虚拟内存分配。

看一下parse_machfile()函数的部分代码：
```
// 1.Mach-o的解析，相关segment虚拟内存分配
// 2.dyld的加载
// 3.dyld的解析以及虚拟内存分配
static load_return_t parse_machfile(
	struct vnode 		*vp,       
	vm_map_t		map,
	thread_t		thread,
	struct mach_header	*header,
	off_t			file_offset,
	off_t			macho_size,
	int			depth,
	int64_t			aslr_offset,
	int64_t			dyld_aslr_offset,
	load_result_t		*result,
	load_result_t		*binresult,
	struct image_params	*imgp
)
{
	uint32_t		ncmds;
	struct load_command	*lcp;
	struct dylinker_command	*dlp = 0;
	load_return_t		ret = LOAD_SUCCESS;

    // depth第一次调用时传入值为0,因此depth正常情况下值为0或者1
	if (depth > 1) {
		return(LOAD_FAILURE);
	}
    // depth负责parse_machfile 遍历次数（2次），第一次是解析mach-o,第二次'load_dylinker'会调用
    // 此函数来进行dyld的解析
	depth++;

    // 会检测CPU type
	if (((cpu_type_t)(header->cputype & ~CPU_ARCH_MASK) != (cpu_type() & ~CPU_ARCH_MASK)) ||
	    !grade_binary(header->cputype, 
	    	header->cpusubtype & ~CPU_SUBTYPE_MASK))
		return(LOAD_BADARCH);
	
	switch (header->filetype) {
	case MH_EXECUTE:
		if (depth != 1) {
			return (LOAD_FAILURE);
		}
		break;
    // 如果fileType是dyld并且是第二次循环调用，那么is_dyld标记为TRUE
	case MH_DYLINKER:
		if (depth != 2) {
			return (LOAD_FAILURE);
		}
		is_dyld = TRUE;
		break;
	default:
		return (LOAD_FAILURE);
	}

    // 如果是dyld的解析，设置slide为传入的aslr_offset
	if ((header->flags & MH_PIE) || is_dyld) {
		slide = aslr_offset;
	}
	for (pass = 0; pass <= 3; pass++) {
        // 遍历load_command
		offset = mach_header_sz;
		ncmds = header->ncmds;
		while (ncmds--) {
            // 针对segment进行内存映射
			switch(lcp->cmd) {
			case LC_SEGMENT: {
				struct segment_command *scp = (struct segment_command *) lcp;
                // segment解析和内存映射
				ret = load_segment(lcp,header->filetype,control,file_offset,macho_size,vp,map,slide,result);
				break;
			}
			case LC_SEGMENT_64: {
				struct segment_command_64 *scp64 = (struct segment_command_64 *) lcp;
				ret = load_segment(lcp,header->filetype,control,file_offset,macho_size,vp,map,slide,result);
				break;
			}
			case LC_UNIXTHREAD:
				ret = load_unixthread((struct thread_command *) lcp,thread,slide,result);
				break;
			case LC_MAIN:
				ret = load_main((struct entry_point_command *) lcp,thread,slide,result);
				break;
			case LC_LOAD_DYLINKER:
                // depth = 1，解析dyld
				if ((depth == 1) && (dlp == 0)) {
					dlp = (struct dylinker_command *)lcp;
					dlarchbits = (header->cputype & CPU_ARCH_MASK);
				} else {
					ret = LOAD_FAILURE;
				}
				break;
			case LC_UUID:
				break;
			case LC_CODE_SIGNATURE:
				ret = load_code_signature((struct linkedit_data_command *) lcp,vp,file_offset,macho_size,header->cputype,result,imgp);
				break;
			default:
				ret = LOAD_SUCCESS;
				break;
			}
		}
	}
	if (ret == LOAD_SUCCESS) { 
		if ((ret == LOAD_SUCCESS) && (dlp != 0)) {
            // 第一次解析mach-o dlp会有赋值，进行dyld的加载
			ret = load_dylinker(dlp, dlarchbits, map, thread, depth,
					    dyld_aslr_offset, result, imgp);
		}
	}
	return(ret);
}

```
parse_machfile()函数中调用了load_dylinker函数，看一下load_dylinker()函数中做了哪些操作。
##### load_dylinker
load_dylinker()函数主要负责加载dyld,以及调用parse_machfile()函数对dyld解析。

load_dylinker()的部分代码：
```
// load_dylinker函数主要负责dyld的加载，解析等工作
static load_return_t load_dylinker(
	struct dylinker_command	*lcp,
	integer_t		archbits,
	vm_map_t		map,
	thread_t	thread,
	int			depth,
	int64_t			slide,
	load_result_t		*result,
	struct image_params	*imgp
)
{
	struct vnode		*vp = NULLVP;	/* set by get_macho_vnode() */
	struct mach_header	*header;
	load_result_t		*myresult;
	kern_return_t		ret;
	struct macho_data	*macho_data;
	struct {
		struct mach_header	__header;
		load_result_t		__myresult;
		struct macho_data	__macho_data;
	} *dyld_data;

#if !(DEVELOPMENT || DEBUG)
    // 非内核debug模式下，会校验name是否和DEFAULT_DYLD_PATH相同，如果不同，直接报错
	if (0 != strcmp(name, DEFAULT_DYLD_PATH)) {
		return (LOAD_BADMACHO);
	}
#endif
    // 读取dyld
	ret = get_macho_vnode(name, archbits, header,
	    &file_offset, &macho_size, macho_data, &vp);
	if (ret)
		goto novp_out;

	*myresult = load_result_null;
	myresult->is64bit = result->is64bit;

    // 解析dyld
	ret = parse_machfile(vp, map, thread, header, file_offset,
	                     macho_size, depth, slide, 0, myresult, result, imgp);
novp_out:
	FREE(dyld_data, M_TEMP);
	return (ret);
}

```
dyld同样是Mach-O类型的文件，因此使用parse_machfile()函数针对dyld进行解析，以及加载其中的segment。代码中使用到了 DEFAULT_DYLD_PATH，看一下DEFAULT_DYLD_PATH的定义：
```
// dyld默认加载地址
#define DEFAULT_DYLD_PATH "/usr/lib/dyld"
```
从这里，也可以看到dyld在系统中的路径。
##### exec_mach_imgact
Mach-O文件以及dyld被映射到虚拟内存后，回过头来，我们再来看一下exec_mach_imgact()函数做的操作。

exec_mach_imgact()函数的部分代码：
```
static int exec_mach_imgact(struct image_params *imgp)
{
	struct mach_header *mach_header = (struct mach_header *)imgp->ip_vdata;
	proc_t			p = vfs_context_proc(imgp->ip_vfs_context);
	int			error = 0;
	thread_t		thread;
	load_return_t		lret;
	load_result_t		load_result;
	
    // 判断是否是Mach-O文件
	if ((mach_header->magic == MH_CIGAM) ||
	    (mach_header->magic == MH_CIGAM_64)) {
		error = EBADARCH;
		goto bad;
	}

    // 判断是否是可执行文件
	if (mach_header->filetype != MH_EXECUTE) {
		error = -1;
		goto bad;
	}

    // 判断cputype和cpusubtype
	if (imgp->ip_origcputype != 0) {
		/* Fat header previously had an idea about this thin file */
		if (imgp->ip_origcputype != mach_header->cputype ||
			imgp->ip_origcpusubtype != mach_header->cpusubtype) {
			error = EBADARCH;
			goto bad;
		}
	} else {
		imgp->ip_origcputype = mach_header->cputype;
		imgp->ip_origcpusubtype = mach_header->cpusubtype;
	}

	task = current_task();
	thread = current_thread();
	uthread = get_bsdthread_info(thread);

	/*
	 * Actually load the image file we previously decided to load.
	 */
    // 加载Mach-O文件，如果返回LOAD_SUCCESS,binary已经映射成可执行内存
	lret = load_machfile(imgp, mach_header, thread, &map, &load_result);
	// 设置内存映射的操作权限
    vm_map_set_user_wire_limit(map, p->p_rlimit[RLIMIT_MEMLOCK].rlim_cur);

	lret = activate_exec_state(task, p, thread, &load_result);
	return(error);
}

```
exec_mach_imgact()函数中做的操作是使用load_machfile()将Mach-O文件映射到内存中，以及设置了一些内存映射的操作权限，之后调用了activate_exec_state()函数。看一下activate_exec_state()函数中做了哪些操作。
##### activate_exec_state
activate_exec_state()函数中主要是调用了thread_setentrypoint()函数。

activate_exec_state()函数的部分代码：
```
static int activate_exec_state(task_t task, proc_t p, thread_t thread, load_result_t *result)
{
	thread_setentrypoint(thread, result->entry_point);
	return KERN_SUCCESS;
}
```
##### thread_setentrypoint
thread_setentrypoint函数实际上设置入口地址，设置的是__dyld_start函数的入口地址。从这一步开始，__dyld_start开始执行。__dyld_start是dyld起始函数，dyld是运行在用户态的，也就是从这里开始，内核态切换到了用户态。

thread_setentrypoint()函数的部分代码：
```
void
thread_setentrypoint(thread_t thread, mach_vm_address_t entry)
{
	pal_register_cache_state(thread, DIRTY);
	if (thread_is_64bit(thread)) {
		x86_saved_state64_t	*iss64;
		iss64 = USER_REGS64(thread);
		iss64->isf.rip = (uint64_t)entry;
	} else {
		x86_saved_state32_t	*iss32;
		iss32 = USER_REGS32(thread);
		iss32->eip = CAST_DOWN_EXPLICIT(unsigned int, entry);
	}
}
```
实际上就是把entry_point的地址直接写入到了寄存器里面。
#### 总结
到这里，就完成了XNU如何将一个Mach-O文件以及dyld加载到内存中的流程分析。其实不看源码，大体流程我们也可以猜到，操作系统想要启动一个app，无非是给这个app分配进程，以及相应的进程空间，之后是给app分配内存，将app映射到内存中。通过源码，能看到每一步是如何实现的。这里只是分析到了XNU将Mach-O文件加载到内存中，实际上后续用户态的dyld还要做一些工作，才能真正的将一个app启动。关于后续dyld做的工作，之后的文章再介绍。

完。

### dyld在app启动过程中的作用
在前面的文章中介绍过，app启动过程中，首先是操作系统内核进行一些处理，比如新建进程，分配内存等。在iOS/Mac OS系统中，操作系统内核是XNU。在XNU完成相关的工作后，会将控制权交给dyld。dyld，即动态链接器，用于加载动态库。dyld是运行在用户态的，从XNU到dyld，完成了一次内核态到用户态的切换。那么，后续dyld做了哪些事情呢？幸运的是，dyld是开源的，我们通过分析dyld的源码，来看一下dyld在app启动过程中做了哪些工作。
#### dyld入口
在之前的文章中介绍过，dyld入口函数是__dyld_start,我们看一下__dyld_start里面做了那些操作。dyld中的部分源码是汇编语言，__dyld_start源码就是汇编。__dyld_start部分代码如下：
```
__dyld_start:
	// 这里调用了dyldbootstrap::start()函数，此函数会完成动态库加载过程，并返回主程序main函数入口
	bl	__ZN13dyldbootstrap5startEPK12macho_headeriPPKclS2_Pm
	mov	x16,x0                  // save entry point address in x16
	ldr     x1, [sp]
	cmp	x1, #0

	// LC_MAIN case, set up stack for call to main()
Lnew:	mov	lr, x1		    // simulate return address into _start in libdyld.dylib
	ldr     x0, [x28, #8] 	    // main param1 = argc
	add     x1, x28, #16	    // main param2 = argv
	add	x2, x1, x0, lsl #3  
	add	x2, x2, #8	    // main param3 = &env[0]
	mov	x3, x2
```
__dyld_start内部调用了dyldbootstrap::start()函数，看一下dyldbootstrap::start()内部的实现：
```
uintptr_t start(const struct macho_header* appsMachHeader, int argc, const char* argv[], 
				intptr_t slide, const struct macho_header* dyldsMachHeader,
				uintptr_t* startGlue)
{
	// if kernel had to slide dyld, we need to fix up load sensitive locations
	// we have to do this before using any global variables
	if ( slide != 0 ) {
		rebaseDyld(dyldsMachHeader, slide);
	}
	// 调用dyld中的_main()函数，_main()函数返回主程序的main函数入口，也就是我们App的main函数地址
	return dyld::_main(appsMachHeader, appsSlide, argc, argv, envp, apple, startGlue);
}

```
查找App main函数地址的操作主要是在_main函数中，_main函数中做了较多的操作，看一下_main()函数是如何实现的。
#### _main()函数
_main()函数中代码比较多，做的事情也比较多。主要完成了上下文的建立，主程序初始化成ImageLoader对象，加载共享的系统动态库，加载依赖的动态库，链接动态库，初始化主程序，返回主程序main()函数地址。接下来分别看一下每个功能的具体实现。
##### instantiateFromLoadedImage
instantiateFromLoadedImage()函数主要是将主程序Mach-O文件转变成了一个ImageLoader对象，用于后续的链接过程。ImageLoader是一个抽象类，和其相关的类有ImageLoaderMachO，ImageLoaderMachO是ImageLoader的子类，ImageLoaderMachO又有两个子类，分别是ImageLoaderMachOCompressed和ImageLoaderMachOClassic。这几个类之间的关系如下：
![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE27644e328208a3d7fc7bc45db1cfc4a0/10658)

在app启动过程中，主程序和其相关的动态库，最后都被转化成了一个ImageLoader对象。看一下instantiateFromLoadedImage中做的操作。
```
static ImageLoaderMachO* instantiateFromLoadedImage(const macho_header* mh, uintptr_t slide, const char* path)
{
	// 检测mach-o header的cputype与cpusubtype是否与当前系统兼容
	if ( isCompatibleMachO((const uint8_t*)mh, path) ) {
		ImageLoader* image = ImageLoaderMachO::instantiateMainExecutable(mh, slide, path, gLinkContext);
		addImage(image);
		return (ImageLoaderMachO*)image;
	}
}
```
isCompatibleMachO主要是检测mach-o文件的cputype和cpusubtype是否与当前系统兼容,之后调用了instantiateMainExecutable()函数，看一下instantiateMainExecutable()函数的实现：
```
// 初始化ImageLoader
ImageLoader* ImageLoaderMachO::instantiateMainExecutable(const macho_header* mh, uintptr_t slide, const char* path, const LinkContext& context)
{
	bool compressed;
	unsigned int segCount;
	unsigned int libCount;
    // sniffLoadCommands主要获取加载命令中compressed的值（压缩还是传统）以及segment的数量、libCount(需要加载的动态库的数量)
	sniffLoadCommands(mh, path, false, &compressed, &segCount, &libCount, context, &codeSigCmd, &encryptCmd);
	if ( compressed ) 
		return ImageLoaderMachOCompressed::instantiateMainExecutable(mh, slide, path, segCount, libCount, context);
	else
#if SUPPORT_CLASSIC_MACHO
		return ImageLoaderMachOClassic::instantiateMainExecutable(mh, slide, path, segCount, libCount, context);
#else
		throw "missing LC_DYLD_INFO load command";
#endif
}
```
instantiateMainExecutable()函数根据Mach-O文件是否被压缩过，分别调用了ImageLoaderMachOCompressed::instantiateMainExecutable()和ImageLoaderMachOClassic::instantiateMainExecutable()。现在的Mach-O文件都是被压缩过的，因此我们只看一下ImageLoaderMachOCompressed::instantiateMainExecutable的实现。
```
// 根据macho_header，返回一个ImageLoaderMachOCompressed对象
ImageLoaderMachOCompressed* ImageLoaderMachOCompressed::instantiateMainExecutable(const macho_header* mh, uintptr_t slide, const char* path, 
																		unsigned int segCount, unsigned int libCount, const LinkContext& context)
{
	ImageLoaderMachOCompressed* image = ImageLoaderMachOCompressed::instantiateStart(mh, path, segCount, libCount);
	image->setSlide(slide);
	image->disableCoverageCheck();
	image->instantiateFinish(context);
	image->setMapped(context);

	return image;
}
```
通过这样一系列的操作，最终，一个Mach-O文件被转变为了一个ImageLoaderMachOCompressed对象。
##### mapSharedCache
mapSharedCache()负责将系统中的共享动态库加载进内存空间，比如UIKit就是动态共享库，这也是不同的App之间能够实现动态库共享的机制。不同App间访问的共享库最终都映射到了同一块物理内存，从而实现了共享动态库。

在Mac OS系统中，动态库共享缓存以文件的形式存放在/var/db/dyld目录下，更新共享缓存的程序是update_dyld_shared_cache,该程序位于 /usr/bin 目录下。update_dyld_shared_cache通常只在系统的安装器安装软件与系统更新时调用。接下来看一下mapSharedCache()的内部实现逻辑。

mapSharedCache()中的代码比较多，我们只看部分代码：
```
// 将本地共享的动态库加载到内存空间，这也是不同app实现动态库共享的机制
// 常见的如UIKit、Foundation都是共享库
static void mapSharedCache()
{
	// _shared_region_***函数，最终调用的都是内核方法
	if ( _shared_region_check_np(&cacheBaseAddress) == 0 ) {
		// 共享库已经被映射到内存中
		sSharedCache = (dyld_cache_header*)cacheBaseAddress;
		if ( strcmp(sSharedCache->magic, magic) != 0 ) {
			// 已经映射到内存中的共享库不能被识别
			sSharedCache = NULL;
			if ( gLinkContext.verboseMapping ) {
				return;
			}
		}
	}
	else {
		// 共享库没有加载到内存中，进行加载
		// 获取共享库文件的句柄，然后进行读取解析
		int fd = openSharedCacheFile();
		if ( fd != -1 ) {
			if ( goodCache ) {
                // 做一个随机的地址偏移
                cacheSlide = pickCacheSlide(mappingCount, mappings);
                //使用_shared_region_map_and_slide_np方法将共享文件映射到内存，_shared_region_map_and_slide_np
                // 内部实际上是做了一个系统调用
	            if (_shared_region_map_and_slide_np(fd, mappingCount, mappings, cacheSlide, slideInfo, slideInfoSize) == 0) {
		            // successfully mapped cache into shared region
		            sSharedCache = (dyld_cache_header*)mappings[0].sfm_address;
		            sSharedCacheSlide = cacheSlide;
	            }
            }		
		}
	}
}
```
mapSharedCache()中调用了内核中的一些方法，最终实际上是做了系统调用。mapSharedCache()的主要逻辑就是：先判断共享动态库是否已经映射到内存中了，如果已经存在，则直接返回；否则打开缓存文件，并将共享动态库映射到内存中。
##### loadInsertedDylib
共享动态库映射到内存后，dyld会把app 环境变量DYLD_INSERT_LIBRARIES中的动态库调用loadInsertedDylib()函数进行加载。可以在xcode中设置环境变量，打印出app启动过程中的DYLD_INSERT_LIBRARIES环境变量，这里看一下我们开发的app的DYLD_INSERT_LIBRARIES环境变量：
```
DYLD_INSERT_LIBRARIES=/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/CoreSimulator/Profiles/Runtimes/iOS.simruntime/Contents/Resources/RuntimeRoot/usr/lib/libBacktraceRecording.dylib:/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/CoreSimulator/Profiles/Runtimes/iOS.simruntime/Contents/Resources/RuntimeRoot/usr/lib/libMainThreadChecker.dylib:/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/CoreSimulator/Profiles/Runtimes/iOS.simruntime/Contents/Resources/RuntimeRoot/Developer/Library/PrivateFrameworks/DTDDISupport.framework/libViewDebuggerSupport.dylib
```
官方文档对于DYLD_INSERT_LIBRARIES的解释：
> This is a colon separated list of dynamic libraries to load before the ones
specified in the program.  This lets you test new modules of existing dynamic
shared libraries that are used in flat-namespace images by loading a temporary
dynamic shared library with just the new modules.

看一下loadInsertedDylib中的实现逻辑：
```
static void loadInsertedDylib(const char* path)
{
	// loadInsertedDylib方法中主要调用了load方法
	ImageLoader* image = NULL;
	try {
		LoadContext context;
		context.useSearchPaths		= false;
		context.useFallbackPaths	= false;
		context.useLdLibraryPath	= false;
		image = load(path, context, cacheIndex);
	}
}

```
loadInsertedDylib()函数中主要是调用了load()函数，看一下load()函数的实现：
```
// load函数是一系列查找动态库的入口
ImageLoader* load(const char* path, const LoadContext& context, unsigned& cacheIndex)
{
	// 根据路径进行一系列的路径搜索、cache查找等
	ImageLoader* image = loadPhase0(path, orgPath, context, cacheIndex, NULL);
	if ( image != NULL ) {
		CRSetCrashLogMessage2(NULL);
		return image;
	}
	// 查找失败，再次查找
	image = loadPhase0(path, orgPath, context, cacheIndex, &exceptions);
	if ( (image == NULL) && cacheablePath(path) && !context.dontLoad ) {
		if ( (myerr == ENOENT) || (myerr == 0) )
		{
			// 从缓存里面找
			if ( findInSharedCacheImage(resolvedPath, false, NULL, &mhInCache, &pathInCache, &slideInCache) ) {
				struct stat stat_buf;
				try {
					image = ImageLoaderMachO::instantiateFromCache(mhInCache, pathInCache, slideInCache, stat_buf, gLinkContext);
					image = checkandAddImage(image, context);
				}
			}
		}
	}
}
```
load()函数是查找动态库的入口，在load()函数中，会调用loadPhase0,loadPhase1,loadPhase2,loadPhase3,loadPhase4,loadPhase5,loadPhase6,对动态库进行查找。最终在loadPhase6中，对mach-o文件进行解析，并最终转成一个ImageLoader对象。看一下loadPhase6中的实现逻辑：
```
// 进行文件读取和mach-o文件解析，最后调用ImageLoaderMachO::instantiateFromFile生成ImageLoader对象
static ImageLoader* loadPhase6(int fd, const struct stat& stat_buf, const char* path, const LoadContext& context)
{
	uint64_t fileOffset = 0;
	uint64_t fileLength = stat_buf.st_size;
	// 最小的mach-o文件大小是4K
	if ( fileLength < 4096 ) {
		if ( pread(fd, firstPages, fileLength, 0) != (ssize_t)fileLength )
			throwf("pread of short file failed: %d", errno);
		shortPage = true;
	} 
	else {
		if ( pread(fd, firstPages, 4096, 0) != 4096 )
			throwf("pread of first 4K failed: %d", errno);
	}
	// 是否兼容，主要是判断cpuType和cpusubType
	if ( isCompatibleMachO(firstPages, path) ) {
		// 只有MH_BUNDLE、MH_DYLIB、MH_EXECUTE 可以被动态的加载
		const mach_header* mh = (mach_header*)firstPages;
		switch ( mh->filetype ) {
			case MH_EXECUTE:
			case MH_DYLIB:
			case MH_BUNDLE:
				break;
			default:
				throw "mach-o, but wrong filetype";
		}
		// 使用instantiateFromFile生成一个ImageLoaderMachO对象
		ImageLoader* image = ImageLoaderMachO::instantiateFromFile(path, fd, firstPages, headerAndLoadCommandsSize, fileOffset, fileLength, stat_buf, gLinkContext);
		return checkandAddImage(image, context);
	}
}

```
loadPhase6中使用了ImageLoaderMachO::instantiateFromFile()函数来生成ImageLoader对象，ImageLoaderMachO::instantiateFromFile()的实现和上面提到的instantiateMainExecutable实现逻辑类似，也是先判断mach-o文件是否被压缩过，然后根据是否被压缩，生成不同的ImageLoader对象，这里不做过多的介绍。
##### link
在将主程序以及其环境变量中的相关动态库都转成ImageLoader对象之后，dyld会将这些ImageLoader链接起来，链接使用的是ImageLoader自身的link()函数。看一下具体的代码实现：
```
void ImageLoader::link(const LinkContext& context, bool forceLazysBound, bool preflightOnly, bool neverUnload, const RPathChain& loaderRPaths, const char* imagePath)
{
	// 递归加载所有依赖库
	this->recursiveLoadLibraries(context, preflightOnly, loaderRPaths, imagePath);

    // 递归修正自己和依赖库的基地址，因为ASLR的原因，需要根据随机slide修正基地址
 	this->recursiveRebase(context);
	
	// recursiveBind对于noLazy的符号进行绑定，lazy的符号会在运行时动态绑定
 	this->recursiveBind(context, forceLazysBound, neverUnload);
}
```
link()函数中主要做了以下的工作：
1. recursiveLoadLibraries递归加载所有的依赖库
2. recursiveRebase递归修正自己和依赖库的基址
3. recursiveBind递归进行符号绑定

在递归加载所有的依赖库的过程中，加载的方法是调用loadLibrary()函数，实际最终调用的还是load()方法。经过link()之后，主程序以及相关依赖库的地址得到了修正，达到了进程可用的目的。
##### initializeMainExecutable
link()函数执行完毕后，会调用initializeMainExecutable()函数，可以将该函数理解为一个初始化函数。实际上，一个app启动的过程中，除了dyld做一些工作外，还有一个重要的角色，就是runtime，而且runtime和dyld是紧密联系的。runtime里面注册了一些dyld的回调通知，这些通知是在runtime初始化的时候注册的。其中有一个通知是，当有新的镜像加载时，会执行runtime中的load-images()函数。接下来看一些runtime中的源码，分析一下load-images()函数做了哪些操作。
```
void load_images(const char *path __unused, const struct mach_header *mh)
{
    if (!hasLoadMethods((const headerType *)mh)) return;

    recursive_mutex_locker_t lock(loadMethodLock);

    // Discover load methods
    {
        rwlock_writer_t lock2(runtimeLock);
        prepare_load_methods((const headerType *)mh);
    }

    // Call +load methods (without runtimeLock - re-entrant)
    call_load_methods();
}
```
load_images()中首先调用了prapare_load_methods()函数，接着调用了call_load_methods()函数。看一下parpare_load_methods()的实现：
```
void prepare_load_methods(const headerType *mhdr)
{
    size_t count, i;
    classref_t *classlist = 
        _getObjc2NonlazyClassList(mhdr, &count);
    for (i = 0; i < count; i++) {
        schedule_class_load(remapClass(classlist[i]));
    }

    category_t **categorylist = _getObjc2NonlazyCategoryList(mhdr, &count);
    for (i = 0; i < count; i++) {
        category_t *cat = categorylist[i];
        Class cls = remapClass(cat->cls);
        if (!cls) continue;  // category for ignored weak-linked class
        realizeClass(cls);
        assert(cls->ISA()->isRealized());
        add_category_to_loadable_list(cat);
    }
}
```
_getObjc2NonlazyClassList获取到了所有类的列表，而remapClass是取得该类对应的指针，然后调用了schedule_class_load()函数，看一下schedule_class_load的实现：
```
static void schedule_class_load(Class cls)
{
    if (!cls) return;
    assert(cls->isRealized());  // _read_images should realize
    if (cls->data()->flags & RW_LOADED) return;
    // Ensure superclass-first ordering
    schedule_class_load(cls->superclass);
    add_class_to_loadable_list(cls);
    cls->setInfo(RW_LOADED); 
}
```
分析这段代码，可以知道，在将子类添加到加载列表之前，其父类一定会优先加载到列表中。这也是为何父类的+load方法在子类的+load方法之前调用的根本原因。

然后我们在看一下call_load_methods()函数的实现：
```
void call_load_methods(void)
{
    static bool loading = NO;
    bool more_categories;
    loadMethodLock.assertLocked();
    if (loading) return;
    loading = YES;

    void *pool = objc_autoreleasePoolPush();

    do {
        while (loadable_classes_used > 0) {
            call_class_loads();
        }
        more_categories = call_category_loads();
    } while (loadable_classes_used > 0  ||  more_categories);
    objc_autoreleasePoolPop(pool);
    loading = NO;
}
```
call_load_methods中主要调用了call_class_loads()函数，看一下call_class_loads的实现：
```
static void call_class_loads(void)
{
    int i;
    struct loadable_class *classes = loadable_classes;
    int used = loadable_classes_used;
    loadable_classes = nil;
    loadable_classes_allocated = 0;
    loadable_classes_used = 0;
    
    // Call all +loads for the detached list.
    for (i = 0; i < used; i++) {
        Class cls = classes[i].cls;
        load_method_t load_method = (load_method_t)classes[i].method;
        if (!cls) continue; 
        if (PrintLoading) {
            _objc_inform("LOAD: +[%s load]\n", cls->nameForLogging());
        }
        (*load_method)(cls, SEL_load);
    }
    
    if (classes) free(classes);
}
```
其主要逻辑就是从待加载的类列表loadable_classes中寻找对应的类，然后找到@selector(load)的实现并执行。
##### getThreadPC
getThreadPC是ImageLoaderMachO中的方法，主要功能是获取app main函数的地址，看一下其实现逻辑：
```
void* ImageLoaderMachO::getThreadPC()
{
	const uint32_t cmd_count = ((macho_header*)fMachOData)->ncmds;
	const struct load_command* const cmds = (struct load_command*)&fMachOData[sizeof(macho_header)];
	const struct load_command* cmd = cmds;
	for (uint32_t i = 0; i < cmd_count; ++i) {
		// 遍历loadCommand,加载loadCommand中的'LC_MAIN'所指向的偏移地址
		if ( cmd->cmd == LC_MAIN ) {
			entry_point_command* mainCmd = (entry_point_command*)cmd;
			// 偏移量 + header所占的字节数，就是main的入口
			void* entry = (void*)(mainCmd->entryoff + (char*)fMachOData);
			if ( this->containsAddress(entry) )
				return entry;
			else
				throw "LC_MAIN entryoff is out of range";
		}
		cmd = (const struct load_command*)(((char*)cmd)+cmd->cmdsize);
	}
	return NULL;
}
```
该函数的主要逻辑就是遍历loadCommand，找到'LC_MAIN'指令，得到该指令所指向的偏移地址，经过处理后，就得到了main函数的地址，将此地址返回给__dyld_start。__dyld_start中将main函数地址保存在寄存器后，跳转到对应的地址，开始执行main函数，至此，一个app的启动流程正式完成。
#### 总结
在上面，已经将_main函数中的每个流程中的关键函数都介绍完了，最后，我们看一下_main函数的实现：
```
uintptr_t
_main(const macho_header* mainExecutableMH, uintptr_t mainExecutableSlide, 
		int argc, const char* argv[], const char* envp[], const char* apple[], 
		uintptr_t* startGlue)
{
	uintptr_t result = 0;
	sMainExecutableMachHeader = mainExecutableMH;
    // 处理环境变量，用于打印
	if ( sEnv.DYLD_PRINT_OPTS )
		printOptions(argv);
	if ( sEnv.DYLD_PRINT_ENV ) 
		printEnvironmentVariables(envp);
	try {
		// 将主程序转变为一个ImageLoader对象
		sMainExecutable = instantiateFromLoadedImage(mainExecutableMH, mainExecutableSlide, sExecPath);
		if ( gLinkContext.sharedRegionMode != ImageLoader::kDontUseSharedRegion ) {
			// 将共享库加载到内存中
			mapSharedCache();
		}
		// 加载环境变量DYLD_INSERT_LIBRARIES中的动态库，使用loadInsertedDylib进行加载
		if	( sEnv.DYLD_INSERT_LIBRARIES != NULL ) {
			for (const char* const* lib = sEnv.DYLD_INSERT_LIBRARIES; *lib != NULL; ++lib) 
				loadInsertedDylib(*lib);
		}
		// 链接
		link(sMainExecutable, sEnv.DYLD_BIND_AT_LAUNCH, true, ImageLoader::RPathChain(NULL, NULL), -1);
        // 初始化
		initializeMainExecutable(); 
		// 寻找main函数入口
		result = (uintptr_t)sMainExecutable->getThreadPC();
	}
	return result;
}

```
本篇文章介绍了从dyld处理主程序Mach-O开始，一直到寻找到主程序Mach-O main函数地址的整个流程。需要注意的是，这也仅仅是一个大概流程的介绍，实际上，除了文章中所写的这些，源码中还有非常多的细节处理，以及一些没有介绍到的知识点。无论是XNU，还是dyld，阅读其源码都是一个巨大的工程，需要在日后不断的学习、回顾。

完。

### iOS app main方法之后做的操作
在iOS开发中，我们都知道，程序的入口是main()函数，位于main.m中。通常情况下，main()函数中的代码是不需要修改的。那么，main()函数中做了哪些操作呢？
#### main函数的实现
在Xcode中新建一个工程之后，Xcode会自动的帮我们生成main.m以及main()函数，main()函数通常是这样的：
```
int main(int argc, char * argv[]) {
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```
main()函数中调用了UIApplicationMain()方法，在确认UIApplicationMain()方法做了什么之前，先来了解一些基本的概念。
##### UIApplication
在开发中，偶尔会用到UIApplication对象，比如[UIApplication sharedApplication]，从使用上来看，就可以猜到这是一个单例对象。实际上，UIApplication对象代表的就是一个app，一个app在运行期间只能有一个UIApplication对象。如果在程序中新建UIApplication对象，会报错。看一下UIApplication类中的一些属性和方法：
```
@interface UIApplication : UIResponder
+ (UIApplication *)sharedApplication;
@property(nullable, nonatomic, assign) id<UIApplicationDelegate> delegate;
@end
```
##### UIApplicationDelegate
UIApplicationDelegate是一个协议，定义在UIApplication.h中，可以看一下UIApplicationDelegate中方法，非常的熟悉：
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(nullable NSDictionary *)launchOptions NS_AVAILABLE_IOS(3_0);
- (void)applicationDidBecomeActive:(UIApplication *)application;
- (void)applicationWillResignActive:(UIApplication *)application;
...
```
这些方法正是AppDelegate.h中提供的方法。移动开发中，app是非常容易受到打扰的，比如说来电、通知，app随时有可能进入后台，甚至被杀死。UIApplicationDelegate协议中的方法正是用来处理这些事件的，app进入前台，进入后台，app被杀死等。

刚才也提到了，Xcode自动生成的AppDelegate文件中实现了UIApplicationDelegate协议中的方法，那么很自然的猜测，AppDelegate类遵守了UIApplicationDelegate协议，事实正是这样：
```
@interface AppDelegate : UIResponder <UIApplicationDelegate>

@property (strong, nonatomic) UIWindow *window;

@end
```
##### UIApplicationMain
UIApplicationMain()接收了4个参数，首先看一下UIApplicationMain()方法的定义：
```
int UIApplicationMain(int argc, char * _Nonnull * _Null_unspecified argv, NSString * _Nullable principalClassName, NSString * _Nullable delegateClassName);
```
argc、argv是系统参数，和其他语言类似。principalClassName是应用程序名称，默认传nil即可，传nil代表的是使用UIApplication类，delegateClassName是代理类名，该类必须要遵守UIApplicationDelegate协议。

现在根据main()函数，来分析下UIApplicationMain()做的事情：
1. 根据传入的principalClassName,创建一个UIApplication对象
2. 根据传入的delegateClassName,创建一个遵守UIApplicationDelegate协议的对象，默认情况下是AppDelegate对象
3. UIApplication对象中有一个属性是 delegate,第2步生成的类赋值给UIApplication对象的delegate属性。
4. 之后开启一个runloop，也就是主runloop，处理事件。

到这里，UIApplication对象已经建立，并且能够处理事件。然而，我们的app最终是要显示到屏幕上，展示给用户看的，一个app如何显示到屏幕上呢？
#### app的显示
app能够显示到屏幕上，不得不提到一个特殊的view，UIWindow，正是由于UIWindow的存在，app才能展示在用户面前。
##### UIWindow
UIWindow是一个特殊的UIView, app启动之后，创建的第一个视图控件就是UIWindow。需要注意的是，UIWindow本身并不做显示，UIWindow更像是一个容器，添加到UIWindow上的view会被显示到屏幕上。
##### 根控制器的创建
我们知道，一个UIWindow必须要有rootViewController，app启动之后会创建一个UIWindow,那么该UIWindow的rootViewController是如何创建的呢？

实际上，app启动后，会加载info.plist文件，如果在info.plist文件中指定了main.storyboard，那么就去加载main.storyboard,根据main.storyboard初始化控制器，并将该控制器赋值给UIWindow的rootViewController属性。如果没有指定main.storyboard,需要手动创建UIViewController，并赋值给UIWindow的rootViewController。手动创建的操作通常是在
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
 {  
    return YES;  
} 
```
方法中。

UIWindow会自动将其rootViewController的view添加，这样其rootViewController就可以显示到屏幕上。
#### 总结
到这里，我们了解到了main()作为程序的入口，做了哪些操作，以及一个app是如何显示到屏幕上的。这篇文章和之前的两篇文章结合起来看，就是一个app启动的完整流程。

完。

### 虚拟内存
物理地址（Physical Address PA）

CPU访问内存最简单、最直接的方法就是使用物理地址，这种寻址方式被称为物理寻址。现代处理器使用的是一种称为虚拟寻址（Virtual Address）的寻址方式。使用虚拟寻址，CPU需要将虚拟地址翻译成物理地址，这样才能访问到真实的物理内存。

MMU（Memory Managment Unit），内存管理单元，用于将虚拟地址翻译成物理地址。MMU需要借助存放在内存中的页表动动态的翻译虚拟地址，页表由操作系统管理（每个进程在物理内存中都有一个对应的页表）。MMU是硬件，位于CPU中，也就是说，MMU是CPU的一部分。

页表就是一个存放在物理内存中的数据结构，它记录了虚拟页与物理页之间的映射关系。页表是一个元素为页表条目（Page Table Entry,PTE）的集合，每个虚拟页在页表中一个固定偏移量的位置上都有一个PTE。

在进行动态内存分配时，例如高级语言中的new关键字，操作系统会在硬盘中创建或申请一段虚拟内存空间，并更新到页表（分配一个PTE，使该PTE指向硬盘上这个新创建的虚拟页）。

由于CPU每次进行地址翻译的时候都需要经过PTE，所以如果想控制内存系统的访问，可以在PTE上添加一些额外许可位（例如读写权限、内核权限等），这样只要指令违反了这些许可条件，CPU就会出发一个一般保护故障，将控制权传递给内核中的异常处理程序。一般这种异常被称为段错误（Segmentation Fault）。

硬盘和内存之间的传输单位是虚拟页（Virtual Page,VP），与之对应的是物理页（Physical Page,PP）,通常情况下，物理页的大小和虚拟页的大小是相同的。

页面调度（paging）:页从硬盘换入内存和从内存换出到硬盘。当缺页异常发生时，才将页面换入到内存的策略称为按需页面调度(demand paging)，所有现代操作系统基本都使用的是按需页面调度策略。

空间局部性原则：一个被访问过的内存地址及其周边的内存地址都会有很大的几率被再次访问。

时间局部性原则：一个被访问过的内存地址在之后会有很大的几率被再次访问。

Linux为每个进程都维护了一个单独的虚拟地址空间。虚拟地址空间分为内核空间和用户空间，用户空间包括代码、数据、堆、栈、共享库，内核空间包括内核中的代码和数据结构。

内核虚拟内存和进程虚拟内存。

Linux将虚拟内存组织成一些段的集合，段的概念允许虚拟地址之间有间隙。一个段就是已经存在着的已分配的虚拟内存的连续片（chunk）。一个段包含多个页。

程序数据加载到内存中使用的是内存映射。Linux通过将一个虚拟内存区域与一个硬盘上的文件关联起来，以初始化这个虚拟内存区域的内容，这个过程称为内存映射（memory mapping）。

动态内存分配器（dynamic memory allocator）。动态内存分配器维护着一个进程的虚拟内存区域，也就是我们所熟悉的堆（heap）。

内存自动回收有两种方法，一种是引用计数的方式，一种是可达性分析的方式。如iOS开发中ARC使用的就是引用计数的方式；Java使用的就是可达性分析的方式。

可达性分析的原理：垃圾收集器将堆内存视为一张有向图，然后选出一组根节点。然后计算从跟节点集合出发的可达路径，只要从根节点出发不可达的节点，都视为垃圾内存。

一个对象被映射到虚拟内存的一个区域，要么是作为共享对象，要么是作为私有对象的。

空闲链表是用来查找空闲块的。

分离存储策略：为了减少分配请求对空闲块匹配的时间，分配器通常采用分离存储策略，即维护多个空闲链表，其中每个链表的块有大致相等的大小。

Linux系统中，每个进程的内存空间为4G，包含3G的用户进程空间和1G的内核进程空间。