## 常用的图片类型
### JPEG
jpeg ：支持有损压缩，可以精确到控制压缩比，牺牲图像质量，减小图像所占用的空间。压缩是不可逆的。不支持透明通道。
iOS系统中，jpeg图片的压缩系数默认是0.9。
### PNG
png：png是无损压缩的。png和 jpeg最大的区别是：png支持完整透明通道。
### GIF
gif：相对于jpeg、png，gif支持多帧动画，这是 gif 和 jpeg、png的最大区别。gif 的透明通道通常只占1bit，完全透明或者完全不透明。
### WebP
WebP：WebP是 google 发布的图片格式，支持有损压缩和无损压缩，支持完整的透明通道，支持多帧动画，最重要的是，相对于jpge、Gif来说，质量相近的图片，WebP所占用的内存更小，因此越来越多的app开始使用WebP。
### iOS系统对各类型图片的支持
iOS系统底层使用的是ImageIO.framework 实现的图片编码、解码。目前ImageIO.framework支持的图片格式有：jpeg、png、gif等，不支持WebP。对于支持的图片类型，开发者可以调用ImageIO中的api对图片进行编码和解码。
## ImageIO framework
ImageIO框架提供了读取图片数据、写入图片数据的方法，使用ImageIO框架能够直接读取到图片的数据，以及一些其他的扩展信息。日常开发中，直接用到ImageIO框架的情况较少，通常直接使用UIImage就可以满足需求。在一些特殊情况下，比如显示gif等，则需要使用到ImageIO框架中的api。
### CGImageSourceRef
CGImageSourceRef(图像源)，可以直观的理解成图像的源。通过二进制流可以创建一个CGImageSourceRef，通过磁盘文件路径也可以创建一个CGImageSourceRef。既然是图像源，因此CGImageSourceRef既可以是静态图的图像源，也可以是动态图的图像源。
#### CGImageSourceRef相关的api
##### 新建CGImageSourceRef
```Objective-C
_source = CGImageSourceCreateWithData((__bridge CFDataRef)_data, NULL); // 根据CFDataRef直接创建UIImageSourceRef

// 创建一个增量的UIImageSourceRef，之后使用UpdateData的方式更新数据（适用于边下载边解码的方式）
_source = CGImageSourceCreateIncremental(NULL);
if (_source) CGImageSourceUpdateData(_source, (__bridge CFDataRef)_data, false);
CGImageSourceCreateWithURL  // 支持网络url和磁盘url
```
##### 通过CGImageSourceRef得到图片信息
```
_frameCount = CGImageSourceGetCount(_source); // 得到帧数

CFDictionaryRef properties = CGImageSourceCopyPropertiesAtIndex(_source, i, NULL);   // 获取某一帧的 property

value = CFDictionaryGetValue(properties, kCGImagePropertyPixelWidth); // 获取宽度，单位为像素
value = CFDictionaryGetValue(properties, kCGImagePropertyPixelHeight); // 获取高度，单位为像素
CGImageRef imageRef = CGImageSourceCreateImageAtIndex(_source, index, (CFDictionaryRef)@{(id)kCGImageSourceShouldCache:@(YES)}); // 通过CGImageSourceRef得到CGImageRef
```
### CGImageRef
CGImageRef 是Core Graphics框架中的定义的数据结构，不过在图片解析中经常和ImageIO框架的api一起结合使用，因此这里放到一起介绍。

CGImageRef的定义如下：
```Objective-C
typedef struct CF_BRIDGED_TYPE(id) CGImage *CGImageRef;
```
#### CGImageRef相关的api
##### 创建一个CGImageRef
```
CGImageRef imageRef = CGImageSourceCreateImageAtIndex(_source, index, (CFDictionaryRef)@{(id)kCGImageSourceShouldCache:@(YES)}); // 通过CGImageSourceRef得到CGImageRef
```
##### 根据CGImageRef得到图片信息
```
size_t width = CGImageGetWidth(imageRef);   // 获取图片宽
size_t height = CGImageGetHeight(imageRef); // 获取图片高
CGColorSpaceRef space = CGImageGetColorSpace(imageRef);  // 获取图片颜色空间
CGImageAlphaInfo alphaInfo = CGImageGetAlphaInfo(imageRef); // 获取图片透明度信息
CGBitmapInfo bitmapInfo = CGImageGetBitmapInfo(imageRef);  // 获取图片bitmapinfo
```
### CGImageDestinationRef
CGImageDestinationRef（图像目标），可以理解为一个容器，将图片数据写入到容器中，并设置一些属性，之后生成新的数据。CGImageDestinationRef常用于图片编码中（将jpg、gif等）转为NSData。可以向CGImageDestinationRef中添加一张图片，也可以添加一组图片，在添加过程中，还可以设置image的属性，添加完成后，调用相关的api即可完成数据写入，并得到新的数据。
#### CGImageDestinationRef相关api
##### 新建CGImageDestinationRef
```Objective-C
destination = CGImageDestinationCreateWithURL((CFURLRef)url, YYImageTypeToUTType(_type), count, NULL); // 根据url新建CGImageDestinationRef
destination = CGImageDestinationCreateWithData((CFMutableDataRef)dest, YYImageTypeToUTType(_type), count, NULL); // 根据NSData新建CGImageDestinationRef
```
##### 向CGImageDestinationRef中添加数据
```Objective-C
CGImageDestinationAddImage(destination, ((UIImage *)imageSrc).CGImage, (CFDictionaryRef)frameProperty)  // 向 CGImageDestinationRef中添加数据
CGImageDestinationAddImageFromSource(destination, source, 0, (CFDictionaryRef)frameProperty); // 向CGImageDestinationRef中添加数据 source是CGImageSourceRef对象
```
##### 完成数据写入
```Objective-C
CGImageDestinationFinalize(destination); // 完成数据写入
```
## 图片解压缩
图片的解压缩，简单来说，就是将一张jpg、png的图片转换成一张bitmap的图片，然后显示到手机屏幕上。

关于解压缩的内容，可以参考文章 https://blog.csdn.net/tugele/article/details/78599414
这里不做太多的介绍。
## YYImageCoder
YYImageCoder 中的代码分为三部分。
### YYImageDecoder
YYImageDecoder是一个解码器，用于将从网络上获取到的二进制流解码成一张张的图片。可以解码普通图片如jpg、png、gif，也可以解码webp。解码普通图片使用系统提供的ImageIO框架即可，解码webp需要借助于libWebp库。

YYImageDecoder的过程：
1. 根据NSData 创建 CGImageSourceRef
2. 由CGImageSourceRef可以获得很多图片信息，比如帧数、宽高、gif循环次数、旋转角度等信息。
3. 由CGImageSourceRef可以得到每一帧的CGImageRef
4. 当需要显示某一帧图片时，使用第三步得到的CGImageRef进行解压缩，得到bitmap图，之后就可以显示到屏幕上。

至此，YYImageDecoder的流程结束。
### YYImageEncoder
YYImageEncoder 是一个编码器，用于将图片转化为 NSData对象。实际上，对于jpg、png来说，根本不需要使用YYImageEncoder，系统提供的有api，可以直接转为NSData对象。YYImageEncoder的主要功能是，将多张静态图，转成一个gif图片对应的 NSData。项目中图片添加文字的功能就用到了 YYImageEncoder。

YYImageEncoder 主要使用了 CGImageDestinationRef，图片添加到CGImageDestinationRef中，最后可以将数据写到 NSData中。
### UIImage + YYImageCoder
对UIImage的扩展，方便使用。
## YYImage
YYImage是UIImage的子类。系统提供的UIImageView是不支持显示gif的，因此如果想显示gif，需要做一些特殊的处理。
### YYAnimatedImage 协议
为了显示gif，YYWebImage提供了 YYAnimatedImage协议，该协议的定义在 YYAnimatedImageView.h文件中。看一下该协议提供的方法：
```Objective-C
// 总帧数
- (NSUInteger)animatedImageFrameCount;

// 循环次数，0代表无限循环
- (NSUInteger)animatedImageLoopCount;

// 每帧的大小
- (NSUInteger)animatedImageBytesPerFrame;

// 返回指定第几帧图片
- (nullable UIImage *)animatedImageFrameAtIndex:(NSUInteger)index;

// 返回指定第几帧的一个帧间距
- (NSTimeInterval)animatedImageDurationAtIndex:(NSUInteger)index;
```
可以看到，上面的几个方法，已经能够表示一个gif。gif实际上也是由一张张图片组合而成的，有每一帧的图片，以及帧于帧之间的间距，也就能够表示一个gif。
### YYImage内部实现
YYImage提供的初始化方法有四个：
```Objective-C
+ (nullable YYImage *)imageNamed:(NSString *)name; // no cache!
+ (nullable YYImage *)imageWithContentsOfFile:(NSString *)path;
+ (nullable YYImage *)imageWithData:(NSData *)data;
+ (nullable YYImage *)imageWithData:(NSData *)data scale:(CGFloat)scale;
```
实际上，无论是哪一个初始化方法，最终执行的都是 .m文件中的
```Objective-C
- (instancetype)initWithData:(NSData *)data scale:(CGFloat)scale
```
方法。

YYImage类中有实例变量 _decoder，由之前的介绍，结合YYImage.m 中的代码，YYImage中使用NSData对象初始化了 YYImageDecoder，YYImageDecoder对图片数据进行了解码，解码后可以得到图片的宽、高，以及每一帧需要显示的图片。

YYImage.m中对YYAnimatedImage协议的实现：
```Objective-C
- (NSUInteger)animatedImageFrameCount {
    // 使用 _decoder得到帧数
    return _decoder.frameCount;
}

- (NSUInteger)animatedImageLoopCount {
    // 使用 _decoder得到循环次数
    return _decoder.loopCount;
}

- (NSUInteger)animatedImageBytesPerFrame {
    return _bytesPerFrame;
}

- (UIImage *)animatedImageFrameAtIndex:(NSUInteger)index {
    // 入参判断
    if (index >= _decoder.frameCount) return nil;
    // 加锁，保证线程安全
    dispatch_semaphore_wait(_preloadedLock, DISPATCH_TIME_FOREVER);
    UIImage *image = _preloadedFrames[index];
    dispatch_semaphore_signal(_preloadedLock);
    if (image) return image == (id)[NSNull null] ? nil : image;
    // 使用 _decoder得到第index帧应该显示的图片
    return [_decoder frameAtIndex:index decodeForDisplay:YES].image;
}

- (NSTimeInterval)animatedImageDurationAtIndex:(NSUInteger)index {
    // 使用 _decoder得到帧间距，这里有个特殊判断
    NSTimeInterval duration = [_decoder frameDurationAtIndex:index];
    if (duration < 0.011f) return 0.100f;
    return duration;
}
```

注意：YYImage是UIImage的子类，如果是gif，YYImage到底是哪一帧图片呢？YYImage表示的是第0帧的图片：
```Objective-C
// 第一帧图片对象
YYImageFrame *frame = [decoder frameAtIndex:0 decodeForDisplay:YES];
UIImage *image = frame.image;
if (!image) return nil;
self = [self initWithCGImage:image.CGImage scale:decoder.scale orientation:image.imageOrientation];
```
## YYAnimatedImageView
### _YYImageWeakProxy
_YYImageWeakProxy 是NSProxy的子类，NSProxy是Objective-C 提供的一个代理类，可以用于消息转发等。

YYAnimatedImageView.m 中的_YYImageWeakProxy主要是用于解决timer强引用target，导致target不能释放的问题。具体的可以参考下文的NSTimer部分。
### _YYAnimatedImageViewFetchOperation
_YYAnimatedImageViewFetchOperation是一个同步的NSOperation。主要作用是获取从下一帧开始的图片，获取到的图片是经过解压缩的，可以直接显示。解压缩的过程是在 YYImageCoder中。

NSOperation通常是和NSOperationQueue结合使用。YYAnimatedImageView中的NSOperationQueue是：
```Objective-C
NSOperationQueue *_requestQueue; ///< image request queue, serial
```
通过设置该NSOperationQueue的最大并发量为1，达到串行的目的。
```Objective-C
_requestQueue = [[NSOperationQueue alloc] init];
_requestQueue.maxConcurrentOperationCount = 1;
```
### YYAnimatedImageView
#### 循环播放gif的实现
YYAnimatedImageView中有一个定时器 _link。
```Objective-C
CADisplayLink *_link; ///< ticker for change frame
```
CADisplayLink 是一个特殊类型的NSTimer，它的特殊之处在于，timer的间隔和屏幕的刷新频率是一致的。iOS设备的屏幕刷新频率(frame per second)是60，也就是说，CADisplayLink的方法一秒钟会被调用60次。

_link对应的selector是step方法，step方法的主要作用是使YYAnimatedImageView连续的显示一帧帧图片。

step方法中做的操作是：
1. 根据当前显示的帧获取下一帧的索引
2. 判断当前是否到时间播放下一帧
3. 如果可以播下一帧了，则尝试从缓存中获取下一帧图片，如果获取到了，则显示下一帧图片，并更新_curIndex。
4. 如果缓存中没有保存到所有帧的图片，则开启新的operation,添加到队列中执行operation。

step方法的部分代码：
```Objective-C
// 定时器方法
- (void)step:(CADisplayLink *)link {
    UIImage <YYAnimatedImage> *image = _curAnimatedImage;
    UIImage *bufferedImage = nil;
    NSUInteger nextIndex = (_curIndex + 1) % _totalFrameCount;
    BOOL bufferIsFull = NO;
    
    NSTimeInterval delay = 0;
    // 第一步：前期处理
    if (!_bufferMiss) {
        _time += link.duration;
        delay = [image animatedImageDurationAtIndex:_curIndex];
        // 如果还不到播放下一帧的时间，直接返回
        if (_time < delay) return;
        _time -= delay;
    }
    LOCK(
         // 从缓存中取下一帧图片
         bufferedImage = buffer[@(nextIndex)];
         if (bufferedImage) {
             // 修改_curIndex、_curFrame的值
             [self willChangeValueForKey:@"currentAnimatedImageIndex"];
             _curIndex = nextIndex;
             [self didChangeValueForKey:@"currentAnimatedImageIndex"];
             _curFrame = bufferedImage == (id)[NSNull null] ? nil : bufferedImage;
             // 再更新nextIndex的值
             nextIndex = (_curIndex + 1) % _totalFrameCount;
             _bufferMiss = NO;
             if (buffer.count == _totalFrameCount) {
                 // 已经将全部的帧全部给缓存下来了
                 bufferIsFull = YES;
             }
         } else {
             _bufferMiss = YES;
         }
    )//LOCK
    
    if (!_bufferMiss) {
        // 显示新的图片
        [self.layer setNeedsDisplay]; // let system call `displayLayer:` before runloop sleep
    }
    if (!bufferIsFull && _requestQueue.operationCount == 0) { // if some work not finished, wait for next opportunity
        // 如果还没有缓存所有的图片，且队列中任务为空，添加新的任务
        _YYAnimatedImageViewFetchOperation *operation = [_YYAnimatedImageViewFetchOperation new];
        operation.view = self;
        operation.nextIndex = nextIndex;
        operation.curImage = image;
        [_requestQueue addOperation:operation];
    }
}

```
注意：这只是部分代码，实际上还有一些判断条件、异常保护相关的代码。
#### YYAnimatedImageView中关于内存的一些优化
1. 收到内存警告时，会移除除下一帧外其余的帧。
```
- (void)didReceiveMemoryWarning:(NSNotification *)notification {
    [_requestQueue cancelAllOperations];
    [_requestQueue addOperationWithBlock: ^{
        NSNumber *next = @((_curIndex + 1) % _totalFrameCount);
        LOCK(
             // 除下一帧外，其他的都移除
             NSArray * keys = _buffer.allKeys;
             for (NSNumber * key in keys) {
                 if (![key isEqualToNumber:next]) { // keep the next frame for smoothly animation
                     [_buffer removeObjectForKey:key];
                 }
             }
        )//LOCK
    }];
}
```
2. 动态计算当前能缓存的最大帧数
```ObjectiveC
// 动态计算最多能缓存的帧数（最大能使用的内存空间 / 每帧的大小）
- (void)calcMaxBufferCount {
    int64_t bytes = (int64_t)_curAnimatedImage.animatedImageBytesPerFrame;
    if (bytes == 0) bytes = 1024;
    
    int64_t total = _YYDeviceMemoryTotal();
    int64_t free = _YYDeviceMemoryFree();
    int64_t max = MIN(total * 0.2, free * 0.6);
    // BUFFER_SIZE默认是10M
    max = MAX(max, BUFFER_SIZE);
    if (_maxBufferSize) max = max > _maxBufferSize ? _maxBufferSize : max;
    double maxBufferCount = (double)max / (double)bytes;
    if (maxBufferCount < 1) maxBufferCount = 1;
    else if (maxBufferCount > 512) maxBufferCount = 512;
    _maxBufferCount = maxBufferCount;
}
```
## YYWebImageOperation
YYWebImageOperation是NSOperation的子类，主要职责是获取图片，可以从内存获取，也可以从磁盘获取，或者从网络获取。

既然YYWebImageOperation是 NSOperation的子类，那么在阅读YYWebImageOperation的代码时，需要想到operation的执行是从那里开始的，子类重写了父类的哪些方法？

### start方法
YYWebImageOperation重写了start方法，看一下start方法中做的操作。
```Objective-C
- (void)start {
    if ([self isCancelled]) {
        // 先判断是否任务已经被取消了，如果已经被取消，则调用 _cancelOperation，且置 finished为YES
        // 在指定的线程上执行_cancelOperation方法
        [self performSelector:@selector(_cancelOperation) onThread:[[self class] _networkThread] withObject:nil waitUntilDone:NO modes:@[NSDefaultRunLoopMode]];
        self.finished = YES;
    } else if ([self isReady] && ![self isFinished] && ![self isExecuting]) {
        // 判断条件  isReady 且  还未结束  且 没有正在执行
        // 真正开始执行任务
        self.executing = YES;
        [self performSelector:@selector(_startOperation) onThread:[[self class] _networkThread] withObject:nil waitUntilDone:NO modes:@[NSDefaultRunLoopMode]];
    }
}
```
如果任务已经取消了，执行的是cancelOperation方法，否则执行startOperation方法。startOperation方法中就是获取图片的操作。需要注意的是这两个方法都是在特定的线程上执行的。

先看一下startOperation里面的内容，也就是获取图片的过程。
### 获得image的过程
startOperation的代码大致如下：
```Objective-C
// runs on network thread
- (void)_startOperation {
    // get image from cache
    if (_cache &&
        !(_options & YYWebImageOptionUseNSURLCache) &&
        !(_options & YYWebImageOptionRefreshImageCache)) {
        // 1.从内存缓存中取图片
        UIImage *image = [_cache getImageForKey:_cacheKey withType:YYImageCacheTypeMemory];
        if (image) {
            if (_completion) _completion(image, _request.URL, YYWebImageFromMemoryCache, YYWebImageStageFinished, nil);
            return;
        }
        if (!(_options & YYWebImageOptionIgnoreDiskCache)) {
            __weak typeof(self) _self = self;
            dispatch_async([self.class _imageQueue], ^{
                // 异步执行
                __strong typeof(_self) self = _self;
                // 从磁盘缓存取图片
                UIImage *image = [self.cache getImageForKey:self.cacheKey withType:YYImageCacheTypeDisk];
                if (image) {
                    // 从磁盘读到之后，保存到内存
                    [self.cache setImage:image imageData:nil forKey:self.cacheKey withType:YYImageCacheTypeMemory];
                    // 在指定的线程上执行
                    [self performSelector:@selector(_didReceiveImageFromDiskCache:) onThread:[self.class _networkThread] withObject:image waitUntilDone:NO];
                } else {
                    // 开启网络请求,注意 onThread
                    [self performSelector:@selector(_startRequest:) onThread:[self.class _networkThread] withObject:nil waitUntilDone:NO];
                }
            });
            return;
        }
    }
    [self performSelector:@selector(_startRequest:) onThread:[self.class _networkThread] withObject:nil waitUntilDone:NO];
}
```
获取image的过程如下：
1. 条件允许的话，首先从内存缓存中获取image，如果获取到了，直接返回，否则到下面流程；
2. 条件允许的话，从磁盘缓存中获取image：
    1. 如果从磁盘中获取到了image，执行_didReceiveImageFromDiskCache 方法，并且将image保存到内存缓存中；返回；
    2. 如果从磁盘没有获取到image，则执行从网络上获取图片的方法，返回；
3. 执行从网络上获取图片的方法

在start方法和startOperation方法中，都有[self.class _networkThread] 这个方法，看一下这个方法到底是什么。
### 网络请求所在的线程
```Objective-C
/// Global image request network thread, used by NSURLConnection delegate.
// 从网络请求图片、取消请求任务，全都是在这一个线程上执行的
+ (NSThread *)_networkThread {
    static NSThread *thread = nil;
    static dispatch_once_t onceToken;
    // 单例，只执行一次
    dispatch_once(&onceToken, ^{
        // 新建线程
        thread = [[NSThread alloc] initWithTarget:self selector:@selector(_networkThreadMain:) object:nil];
        [thread start];
    });
    return thread;
}

+ (void)_networkThreadMain:(id)object {
    [[NSThread currentThread] setName:@"com.ibireme.webimage.request"];
    // 获取当前线程的runloop
    NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
    [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
    // 死循环，以确定线程不退出
    [runLoop run];
}
```
_networkThread方法是新建了一个线程，线程开始执行，调用了_networkThreadMain方法，该方法的操作主要是：
1. 设置线程的名称
2. 使用[NSRunLoop currentRunLoop]方法获取当前线程的runloop
3. 向runloop中添加source，确保runloop不会退出
4. 调用[runloop run]方法，是线程不退出。

我们知道，新建一个线程后，该线程上的操作执行完毕后，线程会退出。而YYWebImageOperation中开启的这个线程，通过这样的处理，可以保证线程不会退出。这样做的理由下面会介绍。
### 使用NSURLConnection获取网络上的图片
#### _startRequest方法
YYWebImageOperation 发送网络请求使用的是NSURLConnection。从网络请求图片的方法是_startRequest方法，看一下_startRequest方法中做了哪些操作。
```Objective-C
// 在网络请求的线程上执行
// runs on network thread
- (void)_startRequest:(id)object {
    if ([self isCancelled]) return;
    // 从网络上获取图片
    if (![self isCancelled]) {
        // 使用 NSURLConnection来请求图片
        //Returns an initialized URL connection and begins to load the data for the URL request.
        //This is equivalent to calling initWithRequest:delegate:startImmediately: and passing YES for startImmediately.
        // 初始化一个 NSURLConnection，并且立即开始执行，默认异步执行
        _connection = [[NSURLConnection alloc] initWithRequest:_request delegate:[_YYWebImageWeakProxy proxyWithTarget:self]];
    }
}
```
初始化NSURLConnection使用了
```Objective-C
- (nullable instancetype)initWithRequest:(NSURLRequest *)request delegate:(nullable id)delegate
```
方法，官方文档中对于该方法delegate参数的介绍中有这样一句话：
> Delegate methods are called on the same thread that called this method.

delegate方法被调用的线程就是初始化方法所在的线程。由上文可知，_startRequest方法所在的线程是[self _networkThread],因此URLConnection代理方法所在的线程也是[self _netWorkThread]。

另外一点需要注意的是，_connection的delegate是 [_YYWebImageWeakProxy proxyWithTarget:self]。_YYWebImageWeakProxy继承于 NSProxy类，可以理解成一个代理类。关于为何这里使用这种方式的原因，看一下作者的解释：
> NSURLConnection 文档里并没有说明这个 delegate 在其内部的属性是 strong、weak 或是 assign。这里把 delegate 封装到 WeakProxy 里，能保证在各种情况下都不会出问题。

这样写的目的能够保证各种情况下都不会有问题。
#### NSURLConnection代理方法
NSURLConnection的代理方法比较多，下面只介绍涉及到数据处理的几个方法。
```Objective-C
// 接收到服务器响应
- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response
```
看一下该方法的实现：
```Objective-C
- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response {
    NSError *error = nil;
    if ([response isKindOfClass:[NSHTTPURLResponse class]]) {
        NSHTTPURLResponse *httpResponse = (id) response;
        NSInteger statusCode = httpResponse.statusCode;
        if (statusCode >= 400 || statusCode == 304) {
            // 如果服务器返回码大于400，或者为304
            error = [NSError errorWithDomain:NSURLErrorDomain code:statusCode userInfo:nil];
        }
    }
    if (error) {
        // 如果错误，取消连接
        [_connection cancel];
        [self connection:_connection didFailWithError:error];
    } else {
        if (response.expectedContentLength) {
            // 数据总长度
            _expectedSize = (NSInteger)response.expectedContentLength;
        }
        // 初始化长度为 expectedSize 的数据流
        _data = [NSMutableData dataWithCapacity:_expectedSize > 0 ? _expectedSize : 0];
    }
}
```
如果服务器返回码大于400或者为304，则请求按照失败处理；否则的话，获得到数据大小，并初始化对应大小的空间。

```Objective-C
// 接收到服务器返回的数据流方法
- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data
```
看一下该方法的实现：
```Objective-C
// 收到服务器返回的数据流
- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data {
    BOOL canceled = [self isCancelled];
    if (canceled) return;
    // 添加到数据流中
    if (data) [_data appendData:data];
}
```
实际上就是将接收到的数据流拼接起来。

```Objective-C
// 当连接成功结束时
- (void)connectionDidFinishLoading:(NSURLConnection *)connection
```
看一下该方法的实现：
```Objective-C
- (void)connectionDidFinishLoading:(NSURLConnection *)connection {
    _connection = nil;
    if (![self isCancelled]) {
        // 请求完毕之后，需要对 NSData 进行解码
        dispatch_async([self.class _imageQueue], ^{
            BOOL shouldDecode = (self.options & YYWebImageOptionIgnoreImageDecoding) == 0;
            BOOL allowAnimation = (self.options & YYWebImageOptionIgnoreAnimatedImage) == 0;
            UIImage *image;
            BOOL hasAnimation = NO;
            if (allowAnimation) {
                image = [[YYImage alloc] initWithData:self.data scale:[UIScreen mainScreen].scale];
                if (shouldDecode) image = [image yy_imageByDecoded];
                if ([((YYImage *)image) animatedImageFrameCount] > 1) {
                    hasAnimation = YES;
                }
            } else {
                YYImageDecoder *decoder = [YYImageDecoder decoderWithData:self.data scale:[UIScreen mainScreen].scale];
                image = [decoder frameAtIndex:0 decodeForDisplay:shouldDecode].image;
            }
            [self performSelector:@selector(_didReceiveImageFromWeb:) onThread:[self.class _networkThread] withObject:image waitUntilDone:NO];
        });
    }
}
```
请求结束之后，得到的是一个数据流NSData对象，需要对NSData对象进行解码，变成我们所需要的UIImage对象。上面代码做的就是这些操作，解码使用的是YYImageCoder中的方法。
## YYWebImageManager
YYWebImageManager 是用来创建和管理YYWebImageOperation的的类。代码量比较少，其内部实现也比较简单，不做过多的介绍。简单看下YYWebImageManager中的一些方法：
```Objective-C
// 内部实现是单例，在该方法中初始化了cache和queue,并调用 - (instancetype)initWithCache:(YYImageCache *)cache queue:(NSOperationQueue *)queue 方法
+ (instancetype)sharedManager;
```

```Objective-C
// 使用cache、queue初始化一个实例对象，并设置默认超时时间，默认request的header
- (instancetype)initWithCache:(YYImageCache *)cache queue:(NSOperationQueue *)queue
```

```Objective-C
// 初始化一个YYWebImageOperation对象，如果_queue不为nil的话，将operation添加到队列中；否则直接执行operation
- (YYWebImageOperation *)requestImageWithURL:(NSURL *)url
                                     options:(YYWebImageOptions)options
                                    progress:(YYWebImageProgressBlock)progress
                                   transform:(YYWebImageTransformBlock)transform
                                  completion:(YYWebImageCompletionBlock)completion
```
## YYCache
YYWebImage中使用的缓存是YYCache，YYCache是YYWebImage作者自己设计的一个缓存，相对于NSCache来说，YYCache提供了多种缓存控制标准，以及保证了较高的缓存命中率。

YYCache内部由YYMemoryCache和YYDiskCache构成，分别是内存缓存和磁盘缓存。内存缓存提供容量小但访问效率高的缓存，磁盘缓存提供容量大但访问效率低的缓存，在使用时，可以根据不同的情况使用不同的缓存。
### YYMemoryCache
首先来看一下YYMemoryCache提供的部分方法：
```Objective-C
// 移除键值为key的对象
- (void)removeObjectForKey:(id)key;

// 移除所有对象
- (void)removeAllObjects;


// 根据缓存总数量限制来删除缓存
- (void)trimToCount:(NSUInteger)count;

// 根据缓存所占空间大小来删除缓存
- (void)trimToCost:(NSUInteger)cost;

// 根据时间来删除缓存
- (void)trimToAge:(NSTimeInterval)age;
```
看一下YYMemoryCache的内部实现。

YYMemoryCache使用了双向链表+字典来保存缓存对象，这样在访问元素、更新元素时，都能够达到O(1)的时间复杂度。
#### YYMemoryCache中的双向链表
既然是链表，需要有链表结点。看一下YYMemoryCache中链表结点的定义：
```Objective-C
@interface _YYLinkedMapNode : NSObject {
    __unsafe_unretained _YYLinkedMapNode *_prev; // 前向指针
    __unsafe_unretained _YYLinkedMapNode *_next; // 后向指针
    id _key;
    id _value;
    NSUInteger _cost;
    NSTimeInterval _time;
}
```
有了链表结点，还需要有数据结构表示双向链表，看一下双向链表的定义：
```Objective-C
@interface _YYLinkedMap : NSObject {
    CFMutableDictionaryRef _dic; // 使用字典来提高访问效率
    NSUInteger _totalCost;
    NSUInteger _totalCount;
    _YYLinkedMapNode *_head; // 链表头结点
    _YYLinkedMapNode *_tail; // 链表尾部结点
}
```
_YYLinkedMap中提供了链表相关的操作方法，比如在链表头部插入结点，移除一个结点，把一个结点移动到头部等。这些方法的实现都比较简单，不做太多的介绍。
#### YYMemoryCache中移除内存元素的实现
YYMemoryCache中有双向链表变量：
```Objective-C
@implementation YYMemoryCache {
    _YYLinkedMap *_lru; // 双向链表
}
```
访问缓存对象、更新缓存对象、删除缓存对象主要是通过 _lru双向链表来完成的。

YYMemoryCache中移除元素遵循的原则是LRU，访问每个元素时，都会更新元素结点的访问时间。双向链表_lru中，头结点的访问时间最晚，尾结点的访问时间最早，删除元素时，从尾结点开始删除，直到达到了标准。

看一下设置元素的代码：
```Objective-C
- (void)setObject:(id)object forKey:(id)key withCost:(NSUInteger)cost {
    _YYLinkedMapNode *node = CFDictionaryGetValue(_lru->_dic, (__bridge const void *)(key));
    NSTimeInterval now = CACurrentMediaTime();
    if (node) {
        // 如果原来该key对应的元素存在，则更新该元素信息
        _lru->_totalCost -= node->_cost;
        _lru->_totalCost += cost;
        node->_cost = cost;
        node->_time = now;
        node->_value = object;
        // 将该元素放到链表头结点（最新访问）
        [_lru bringNodeToHead:node];
    } else {
        // 如果不存在，则新建结点
        node = [_YYLinkedMapNode new];
        node->_cost = cost;
        node->_time = now;
        node->_key = key;
        node->_value = object;
        // 将结点插入到链表头结点
        [_lru insertNodeAtHead:node];
    }
}
```
当缓存元素总大小超过costLimit时，删除部分缓存元素的实现：
```Objective-C
- (void)_trimToCost:(NSUInteger)costLimit {
    BOOL finish = NO;
    if (costLimit == 0) {
        // 移除所有元素
        [_lru removeAll];
        finish = YES;
    } else if (_lru->_totalCost <= costLimit) {
        finish = YES;
    }
    if (finish) return;
    
    while (!finish) {
        if (_lru->_totalCost > costLimit) {
            // 从链表尾部开始移除,链表尾部元素是访问时间最早的元素
            [_lru removeTailNode];
        } else {
            finish = YES;
        }
    }
}
```
可以看到，在删除元素时，是从链表尾部开始删除的，因为链表尾部是访问时间最早的，遵循LRU原则。

YYMemoryCache还提供了以元素总数、元素访问时间为标准来删除元素的方式，实现逻辑和上面看到的基本类似，就不做太多的介绍了。

### YYDiskCache
YYDiskCache是YYCache中的磁盘缓存类，支持文件缓存和sqllite缓存。

YYDiskCache提供的接口和YYMemoryCache类似，简单看一下：
```Objective-C
/** The name of the cache. Default is nil. */
@property (nullable, copy) NSString *name;

/** The path of the cache (read-only). */
@property (readonly) NSString *path;

@property (readonly) NSUInteger inlineThreshold;
// 总个数限制
@property NSUInteger countLimit;
// 所占总空间限制
@property NSUInteger costLimit;
// 访问时间限制
@property NSTimeInterval ageLimit;
// 磁盘所剩余空间限制
@property NSUInteger freeDiskSpaceLimit;
```
相对于内存缓存来说，YYDiskCache多了freeDiskSpaceLimit，通过限制磁盘剩余的空间来控制缓存对象；另外呢多了 inlineThreshold，该值决定了对象采用文件存储还是数据库存储，下面会介绍到。

看一下YYDiskCache的内部实现。
#### NSMapTable
和YYMemoryCache不同，YYDiskCache内部中使用一个全局的NSMapTable来管理不同的diskCache。NSMapTable类似于NSDictionary，不过NSMapTable中的key、value可以是任意指针，而不仅仅是对象。当初始化缓存时，会先从NSMapTable中尝试获得，如果能够获得，则直接返回，节省效率。来看一下NSMapTable相关的代码：
```Objective-C
static NSMapTable *_globalInstances;

static void _YYDiskCacheInitGlobal() {
    static dispatch_once_t onceToken;
    // 单例，应用程序使用期间，只被初始化一次
    dispatch_once(&onceToken, ^{
        _globalInstances = [[NSMapTable alloc] initWithKeyOptions:NSPointerFunctionsStrongMemory valueOptions:NSPointerFunctionsWeakMemory capacity:0];
    });
}

static YYDiskCache *_YYDiskCacheGetGlobal(NSString *path) {
    _YYDiskCacheInitGlobal();
    id cache = [_globalInstances objectForKey:path];
    return cache;
}

static void _YYDiskCacheSetGlobal(YYDiskCache *cache) {
    _YYDiskCacheInitGlobal();
    [_globalInstances setObject:cache forKey:cache.path];
}
```
初始化YYDiskCache时的过程：
```Objective-C
- (instancetype)initWithPath:(NSString *)path inlineThreshold:(NSUInteger)threshold {
    self = [super init];
    // 先尝试从NSMapTable中获取
    YYDiskCache *globalCache = _YYDiskCacheGetGlobal(path);
    if (globalCache) return globalCache;
    
    // 没获取到，做初始化相关的工作
    // 更新NSMapTable
    _YYDiskCacheSetGlobal(self);
    return self;
}
```
#### YYKVStorageItem
和YYMemoryCache类似，YYDiskCache也不会直接操作缓存对象，而是通过 YYKVStorage来操作缓存对象的。YYKVstorage中还定义了YYKVStorageItem来表示缓存对象。

YYKVStorageItem的定义：
```Objective-C
@interface YYKVStorageItem : NSObject
@property (nonatomic, strong) NSString *key;
@property (nonatomic, strong) NSData *value;
@property (nullable, nonatomic, strong) NSString *filename;
@property (nonatomic) int size;
@property (nonatomic) int modTime;
@property (nonatomic) int accessTime;
@end
```
YYKVStorageItem可以理解成YYMemoryCache中的_YYLinkMapNode。
#### YYKVStorageType
YYKVStorageType是一个枚举值，决定了缓存对象的存储方式。YYKVStorateType的取值类型：
```Objective-C
typedef NS_ENUM(NSUInteger, YYKVStorageType) {
    // 保存为文件
    YYKVStorageTypeFile = 0,
    // 保存到sqlite
    YYKVStorageTypeSQLite = 1,
    // 保存到文件或者sqlite
    YYKVStorageTypeMixed = 2,
};
```
使用YYDiskCache时，可以选择文件存储，也可以选择sqlite存储，如果type是YYKVStorageTypeMixed，则根据缓存对象的大小来决定使用文件存储还是sqlite存储。缓存对象的大小和上面提到的inlineThreshold值比较，如果大于inlineThreshold，则选择文件保存；否则选择sqlite保存。
#### YYKVStorage的内部实现
看一下YYKVStorage的内部实现，YYKVStorage内部有很多关于数据库和文件的操作，主要是看一下其提供的接口的实现。
##### 添加元素到缓存中
```Objective-C
- (BOOL)saveItemWithKey:(NSString *)key value:(NSData *)value filename:(NSString *)filename extendedData:(NSData *)extendedData {
    if (filename.length) {
        // 写文件
        if (![self _fileWriteWithName:filename data:value]) {
            return NO;
        }
        // 保存到数据库（如果文件路径不为空，既写文件，又写数据库）
        if (![self _dbSaveWithKey:key value:value fileName:filename extendedData:extendedData]) {
            // 保存失败，删除文件
            [self _fileDeleteWithName:filename];
            return NO;
        }
        return YES;
    } else {
        // 保存到数据库
        return [self _dbSaveWithKey:key value:value fileName:nil extendedData:extendedData];
    }
}
```
保存数据到数据库的代码：
```Objective-C
- (BOOL)_dbSaveWithKey:(NSString *)key value:(NSData *)value fileName:(NSString *)fileName extendedData:(NSData *)extendedData {
    NSString *sql = @"insert or replace into manifest (key, filename, size, inline_data, modification_time, last_access_time, extended_data) values (?1, ?2, ?3, ?4, ?5, ?6, ?7);";
    sqlite3_stmt *stmt = [self _dbPrepareStmt:sql];
    int timestamp = (int)time(NULL);
    sqlite3_bind_text(stmt, 1, key.UTF8String, -1, NULL);
    sqlite3_bind_text(stmt, 2, fileName.UTF8String, -1, NULL);
    sqlite3_bind_int(stmt, 3, (int)value.length);
    if (fileName.length == 0) {
        // 如果在本地文件没有保存，则将原始数据保存到数据库中
        sqlite3_bind_blob(stmt, 4, value.bytes, (int)value.length, 0);
    } else {
        // 如果本地文件已经保存了，则数据库中没必要再次保存原始数据(注意这里的区别)
        sqlite3_bind_blob(stmt, 4, NULL, 0, 0);
    }
    sqlite3_bind_int(stmt, 5, timestamp);
    sqlite3_bind_int(stmt, 6, timestamp);
    sqlite3_bind_blob(stmt, 7, extendedData.bytes, (int)extendedData.length, 0);
    int result = sqlite3_step(stmt);
    return YES;
}
```
注意：如果缓存对象在本地已经保存过了，那么在数据库中不需要再次保存其原始数据，节省资源。
##### 从缓存中移除元素
可以根据多种限制条件从缓存中移除元素，这里举一个简单的例子：
```Objective-C
- (BOOL)removeItemsToFitSize:(int)maxSize {
    // 获取总的size
    int total = [self _dbGetTotalItemSize];
    NSArray *items = nil;
    BOOL suc = NO;
    do {
        int perCount = 16;
        // 根据访问时间排序，访问时间越早，排在越前面，每次取16个(LRU算法)
        items = [self _dbGetItemSizeInfoOrderByTimeAscWithLimit:perCount];
        for (YYKVStorageItem *item in items) {
            if (total > maxSize) {
                if (item.filename) {
                    // 先删除本地文件
                    [self _fileDeleteWithName:item.filename];
                }
                // 在删除数据库中的记录
                suc = [self _dbDeleteItemWithKey:item.key];
                total -= item.size;
            } else {
                break;
            }
            if (!suc) break;
        }
    } while (total > maxSize && items.count > 0 && suc);
    return suc;
}
```
使用LRU算法从缓存中移除元素。
##### 更新元素的访问时间
```Objective-C
- (YYKVStorageItem *)getItemForKey:(NSString *)key {
    YYKVStorageItem *item = [self _dbGetItemWithKey:key excludeInlineData:NO];
    if (item) {
        // 更新访问时间
        [self _dbUpdateAccessTimeWithKey:key];
    }
    return item;
}
```
## 小知识点整理
### __bridge
Core Foundation框架是一组C语言接口，为iOS程序提供基本的数据管理。
Core Foundation框架和Foundation框架是紧密相关的。可以理解成，Core Foundation框架和Foundation框架为相同的功能提供了不同的接口，Foundation框架提供的是Objective-C接口。两个框架中的数据类型也是对应的。例如：NSData 对应于 CFDataRef，id 可以和 void*对应。

在程序开发中，有时会用到Core Foundation框架中的对象（简称CF），并且可能会需要将OC对象和CF对象相互转换，或者将CF对象和OC对象相互转换，此时可以使用__bridge（桥接）来转换。

需要注意的是，__bridge是ARC环境下的技术，在MRC环境下不能使用，而且不会改变对象的引用计数。__bridge的使用举例：
``` Objective-C
YYImageDetectType((__bridge CFDataRef)data);
```

完。
### NSTimer 使用注意事项
NSTimer 是系统提供的定时器，系统提供的api也比较简单，使用很方便，项目开发中会经常用到。然而，在使用NSTimer时，如果不注意，非常容易引起内存泄露的问题。本文总结了下NSTimer 引起内存泄露问题的原因，以及解决方案。
#### NSTimer的使用
通常情况下，NSTimer 是作为controller或者view的一个属性来使用:
```Objective-C
/**
 gif播放的定时器
 */
@property (nonatomic, strong) NSTimer *gifPlayTimer;
```
timer初始化：
```Objective-C
NSTimer * timer = [NSTimer scheduledTimerWithTimeInterval:0.01 target:self selector:@selector(refreshPlayTime) userInfo:nil repeats:YES];
    self.gifPlayTimer = timer;
    [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```
该timer的作用是每隔0.01秒会执行一次self 的 refreshPlayTime 方法。

这样使用是没有问题的，每0.01秒确实会执行一次 refreshPlayTime方法。

然而当该控制器退出之后，会发现timer仍旧在执行，每隔0.01秒还是会调用 refreshPlayTime方法，而且，控制器退出了，但是该控制器的 dealloc 方法并没有被调用，也就是该控制器没有被释放，有内存泄露的问题。

既然timer没有停止，那么手动调用timer的 invalidate方法试一下。

通常情况下，我们希望在控制器释放的时候结束timer，也就是在 dealloc 方法中将timer给停掉。代码如下：
```Objective-C
- (void)dealloc
{
    [self.gifPlayTimer invalidate];
}
```
然而，并没有什么用，在退出控制器之后，timer仍旧生效。原因上面其实也说了，因为控制器的 dealloc方法根本没有被调用。为什么控制器不会被释放？以及如何解决？
#### NSTimer对target的强引用
首先看一下NSTimer初始化方法的官方文档介绍：
```Objective-C
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo;
```
注意target参数的描述：
> The object to which to send the message specified by aSelector when the timer fires. The timer maintains a strong reference to target until it (the timer) is invalidated.

注意：文档中写的很清楚，timer对target会有一个强引用，直到timer is invalidated。也就是说，在timer调用 invalidate方法之前，timer对target一直都有一个强引用。这也是为什么控制器的dealloc 方法不会被调用的原因。

由于timer对target强引用的特性，如果要避免控制器不释放的问题，需要在特定的时机调用timer 的 invalidate方法，也就是提前结束timer。在通常情况下，这种方式是可以解决问题的，虽然需要警惕页面退出之前有没有结束timer，但毕竟解决了问题不是。但是，日常项目中通常是多人协作，如果该timer是一个view的属性，而这个view又需要让别人使用，那timer什么时候结束呢？让调用者来管理timer的结束显然是不合理的。更好的方式还是应该在dealloc 方法中结束timer，这样调用者根本无须关注timer。

那么如何解决呢？
#### timer修饰符改为weak
上述代码中，self强引用了timer，timer又强引用了self，导致timer不能释放，self也一直不能释放，那么如果timer的修饰符是weak，能解决这个问题嘛？
```Objective-C
/**
 gif播放的定时器
 */
@property (nonatomic, weak) NSTimer *gifPlayTimer;
```
经过验证，**使用weak修饰timer并不能解决问题**。Why?

看一下
```Objective-C
- (void)addTimer:(NSTimer *)timer forMode:(NSRunLoopMode)mode;
```
方法的文档介绍：
> The receiver retains aTimer. To remove a timer from all run loop modes on which it is installed, send an invalidate message to the timer.

也就是说，runLoop会对timer有强引用，因此，timer修饰符是weak，timer还是不能释放，timer的target也就不能释放。
#### target用weak来修饰
既然timer强引用了target，导致target一直不能释放，如果target用weak来修饰，能解决这个问题嘛？
```Objective-C
__weak typeof(self) weakSelf = self;
NSTimer * timer = [NSTimer scheduledTimerWithTimeInterval:0.01 target:weakSelf selector:@selector(refreshPlayTime) userInfo:nil repeats:YES];
self.gifPlayTimer = timer;
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```
经过验证，**并没有解决问题**。Why?

实际上，上面的写法和直接使用self的并没有太大的区别，唯一的区别是这种写法timer的target有可能是nil，不过这种可能性太低了。
#### 使用中间target的方式
这种方法的思路是：新建一个中间对象Object，该中间对象对timer真正的target有一个弱引用，写代码时，timer的target 是Object。timer触发的方法仍旧是真正target中的方法。

部分代码如下：
```Objective-C
@interface _YYImageWeakProxy : NSProxy
// 对target有一个弱引用
@property (nonatomic, weak, readonly) id target;
- (instancetype)initWithTarget:(id)target;
+ (instancetype)proxyWithTarget:(id)target;
@end
```
timer的初始化方法：
```Objective-C
NSTimer * timer = [NSTimer scheduledTimerWithTimeInterval:0.01 target:[_YYImageWeakProxy proxyWithTarget:self] selector:@selector(refreshPlayTime) userInfo:nil repeats:YES];
self.gifPlayTimer = timer;
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```
使用这种方式验证一下，**是可以解决问题的，target的dealloc方法会被调用**。

上述的引用关系如下：
```
graph TD
A[self] -->B[timer]
B -->|strong|C[_YYImageWeakProxy]
C -->|weak|D[self]
```
这种方式避免了timer直接引用target，因此self的dealloc方法会调用，在dealloc方法中移除timer即可。

使用这种方式timer的方法如何执行呢？其实通过上面的代码也可以看出，借助了NSProxy类。NSProxy类是和NSObject平级的类，该类的作用可以简单的理解为一个代理，将消息转发给另一个对象。
看一下_YYImageWeakProxy中的代码：
```Objective-C
@implementation _YYImageWeakProxy

- (id)forwardingTargetForSelector:(SEL)selector {
    return _target;
}
- (void)forwardInvocation:(NSInvocation *)invocation {
    void *null = NULL;
    [invocation setReturnValue:&null];
}
- (NSMethodSignature *)methodSignatureForSelector:(SEL)selector {
    return [NSObject instanceMethodSignatureForSelector:@selector(init)];
}
- (BOOL)respondsToSelector:(SEL)aSelector {
    return [_target respondsToSelector:aSelector];
}

- (Class)class {
    return [_target class];
}
- (BOOL)isKindOfClass:(Class)aClass {
    return [_target isKindOfClass:aClass];
}
- (BOOL)isMemberOfClass:(Class)aClass {
    return [_target isMemberOfClass:aClass];
}
- (BOOL)conformsToProtocol:(Protocol *)aProtocol {
    return [_target conformsToProtocol:aProtocol];
}
- (BOOL)isProxy {
    return YES;
}

@end
```
主要完成了消息转发的功能，将其接收到的消息，转发给target，这样timer就能正确触发对应的方法。
#### NSTimer+YYAdd
YYKit框架中提供了NSTimer的一个分类，NSTimer+YYAdd，使用该分类中的方法，**能够解决问题，target的dealloc方法会被调用**。看一下使用方法：
```Objective-C
NSTimer * timer = [NSTimer timerWithTimeInterval:3.0f block:^(NSTimer * _Nonnull timer) {
            [weakSelf hideControlViewWithAnimation];
        } repeats:YES];
_hiddenTimer = timer;
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```
```Objective-C
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)seconds block:(void (^)(NSTimer *timer))block repeats:(BOOL)repeats;
```
该方法是NSTimer+YYAdd 提供的。那么该Category是如何解决timer强引用target的问题呢？看一下其内部实现：
```Objective-C
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)seconds block:(void (^)(NSTimer *timer))block repeats:(BOOL)repeats {
    return [NSTimer timerWithTimeInterval:seconds target:self selector:@selector(_yy_ExecBlock:) userInfo:[block copy] repeats:repeats];
}

+ (void)_yy_ExecBlock:(NSTimer *)timer {
    if ([timer userInfo]) {
        void (^block)(NSTimer *timer) = (void (^)(NSTimer *timer))[timer userInfo];
        block(timer);
    }
}
```
注意：timer的target变成了self，也就是timer，而_yy_ExecBlock方法实际上就是执行timer的回调。

也就是说，NSTimer+YYAdd中的解决方式也是使用中间类的方式，只不过这里的中间类正好是NSTimer对象，写起来更简单一些。具体引用关系如下：
```
graph TD
A[self] -->B[timer]
B -->C[timer 中间类]
C -->|weak|D[self]
```
#### invalidate方法注意事项
看一下invalidate方法的介绍：
> This method is the only way to remove a timer from an NSRunLoop object. The NSRunLoop object removes its strong reference to the timer, either just before the invalidate method returns or at some later point.

> You must send this message from the thread on which the timer was installed. If you send this message from another thread, the input source associated with the timer may not be removed from its run loop, which could prevent the thread from exiting properly.

两点：

（1）invalidate方法是唯一能从runloop中移除timer的方式，调用invalidate方法后，runloop会移除对timer的强引用。

（2）timer的添加和timer的移除（invalidate）需要在同一个线程中，否则timer可能不能正确的移除，线程不能正确退出。

完。
### NSOperation
NSOperation 是Objective-C 提供的一种面向对象的多线程编程方式。在项目开发中，多线程通常使用两种方式，GCD 和 NSOperation。GCD的使用相对来说更多一些，因此本文介绍下NSOperation，以及GCD和NSOperation的使用场景区别。
#### NSOperation的使用
先看下官方文档对NSOperation的介绍：
> An abstract class that represents the code and data associated with a single task.

NSOperation本身是一个抽象类，是不能直接使用的，需要子类化之后才能够使用。系统提供的有两个子类：NSBlockOperation 和 NSInvocationOperation。不过，通常在使用NSOperation时，都是自定义子类，而不是直接使用系统提供的两个子类。

NSOperation和NSOperationQueue结合使用，能够实现多线程编程。关于NSOperationQueue,下面会介绍。
#### NSOperation 中的方法
既然自定义类继承于 NSOperation，那么子类需要重写父类的哪些方法呢？在回到这个问题之前，先看一下NSOperation 提供的方法。
```Objective-C
- (void)start;
- (void)main;
- (void)cancel;
```
NSOperation.h中主要是这三个方法。由于这三个方法没有注释，因此还是看一下这三个方法的官方文档介绍，了解一下这三个方法的作用。
##### start方法
> Begins the execution of the operation.
The default implementation of this method updates the execution state of the operation and calls the receiver’s main method. This method also performs several checks to ensure that the operation can actually run. 

根据文档介绍可以了解到start方法的作用以及内部的默认实现。start方法的作用是开始执行这个operation。

start方法的默认实现中，在operation执行的过程中，会改变operation的状态，而且，start方法会调用 main方法。start方法同时也会做一些检测，以确保该operation真的可以执行。

上面提到start方法中会调用main方法，那么main方法中又做了什么操作？
##### main方法
> Performs the receiver’s non-concurrent task.
The default implementation of this method does nothing. You should override this method to perform the desired task. In your implementation, do not invoke super. 

> If you are implementing a concurrent operation, you are not required to override this method but may do so if you plan to call it from your custom start method.

main方法的作用：执行非并行（串行）的任务。

main方法的默认实现是空的。在子类中应该重写main方法，在main方法中做该operation 期望的操作。而且**在main方法中不要调用super 的main方法**。NSOperation如果是一个串行的，那么需要重写main方法，如果实现的是一个并行的任务，则不需要重写main方法。

PS：文档真的非常有用，不仅仅会介绍方法的作用，而且会告诉我们方法的使用场景。
##### cancel方法
> Advises the operation object that it should stop executing its task.
This method does not force your operation code to stop. Instead, it updates the object’s internal flags to reflect the change in state.

cancel方法的作用是修改operation的状态。
#### NSOperation 的几种状态
上面已经多次提到了NSOperation的状态，那么NSOperation都有哪些状态呢？

看一下NSOperation.h中的定义：
```Objective-C
@property (readonly, getter=isCancelled) BOOL cancelled;

@property (readonly, getter=isExecuting) BOOL executing;
@property (readonly, getter=isFinished) BOOL finished;
@property (readonly, getter=isReady) BOOL ready;
```
isCancelled、isExecuting、isFinished、isReady，NSOperation的四个状态：是否被取消、是否正在执行、是否执行完毕、是否准备好执行。看一下NSOperation的四个状态。
##### isCancelled
> A Boolean value indicating whether the operation has been cancelled
The default value of this property is NO. Calling the cancel method of this object sets the value of this property to YES. Once canceled, an operation must move to the finished state.

> You should always check the value of this property before doing any work towards accomplishing the operation’s task, which typically means checking it at the beginning of your custom main method.

作用：标记operation是否被取消了。默认值是NO。调用 cancel 方法后，该值会变为YES。一旦一个operation被取消了，他的finish状态必须变成YES。

在开始执行一个operation之前，应该检查一下isCancelled是否为YES，也就是说，在自定义NSOperation的子类的main方法中，应该首先检查isCancelled，如果operation被取消了，那么就没有执行的必要了。
##### isReady
> A Boolean value indicating whether the operation can be performed now.
The readiness of operations is determined by their dependencies on other operations and potentially by custom conditions that you define. 

标记operation是否准备好执行的bool值。值取决于该operation的依赖关系，以及开发者自定义的条件。
> If you want to use custom conditions to define the readiness of your operation object, reimplement this property and return a value that accurately reflects the readiness of the receiver. If you do so, your custom implementation must get the default property value from super and incorporate that readiness value into the new value of the property. In your custom implementation, you must generate KVO notifications for the isReady key path whenever the ready state of your operation object changes. 

如果开发者想要使用自定义的条件来决定operation isReady的值，那么需要重新实现该属性，如果这样做了，自定义的实现中，必须将[super isReady]和自定义的条件组合起来，两者都为YES，才可以返回YES。另外需要注意的是，如果自定义实现了该方法，当ready值改变时，**需要手动触发KVO通知**。
##### isExecuting
> A Boolean value indicating whether the operation is currently executing.

标记operation是否正在执行的bool值。
> When implementing a concurrent operation object, you must override the implementation of this property so that you can return the execution state of your operation. In your custom implementation, you must generate KVO notifications for the isExecuting key path whenever the execution state of your operation object changes.

> You do not need to reimplement this property for nonconcurrent operations.

当子类是一个并发operation时，必须重新实现该方法。在自定义实现中，当isExecuting的值改变时，需要手动触发KVO通知。

如果operation是同步的，则没有必要重写isExecuting方法。
##### isFinished
> A Boolean value indicating whether the operation has finished executing its task.

标记operation是否执行完毕的bool值。
> When implementing a concurrent operation object, you must override the implementation of this property so that you can return the finished state of your operation. In your custom implementation, you must generate KVO notifications for the isFinished key path whenever the finished state of your operation object changes. 

> You do not need to reimplement this property for nonconcurrent operations.

当子类operation是并行时，必须重新实现isFinished方法。在自定义实现中，如果isFinished的值改变了，必须触发KVO通知。

如果NSOperation是同步的，则没必要重写isFinished方法。
#### 同步NSOperation的实现
其实呢，根据上面的介绍，已经可以看出同步NSOperation和异步NSOperation的实现是有较大的差别的，而且同步NSOperation的实现要简单一些。

既然是同步的，start方法结束时，operation也就执行完毕了，而start方法在执行过程中会调用main方法，因此通常情况下，同步NSOperation只需要重写main方法就可以。main方法中实现operation真正要执行的操作。

举个例子：
```Objective-C
// 1.main方法中不需要调用 [super main]
// 2.在main开始之前，先检查operation是否被取消了
- (void)main {
    if ([self isCancelled]) return;
    // do somthing
}
```
#### 异步NSOperation的实现
对于异步NSOperation来说，start方法执行完毕后，operation可能还没有执行完毕。因此，默认的start方法是不能满足异步NSOperation的需求的。

对于一个异步NSOperation来说，通常需要重写的方法有：

start方法：需要在start方法中将operation放在其他线程执行，且需要改变operation的状态。

isExecuting 和 isFinished。这两个方法上面已经介绍过了，而且官方文档中明确说明了什么时候需要重写，什么时候不需要重写。

举个例子：
```Objective-C
- (void)start {
    if ([self isCancelled]) {
        // 先判断是否任务已经被取消了，如果已经被取消，则调用 _cancelOperation，且置 finished为YES
        [self performSelector:@selector(_cancelOperation) onThread:[[self class] _networkThread] withObject:nil waitUntilDone:NO modes:@[NSDefaultRunLoopMode]];
        self.finished = YES;
    } else if ([self isReady] && ![self isFinished] && ![self isExecuting]) {
        // 判断条件  isReady 且  还未结束  且 没有正在执行
        // 改变executing的状态
        self.executing = YES;
        // 在其他线程上执行任务
        [self performSelector:@selector(_startOperation) onThread:[[self class] _networkThread] withObject:nil waitUntilDone:NO modes:@[NSDefaultRunLoopMode]];
    }
}
```
```Objective-C
- (void)setFinished:(BOOL)finished {
    if (_finished != finished) {
        // 触发kvo
        [self willChangeValueForKey:@"isFinished"];
        _finished = finished;
        [self didChangeValueForKey:@"isFinished"];
    }
}

- (BOOL)isFinished {
    BOOL finished = _finished;
    return finished;
}
```
isExecuting的代码和isFinished的代码类似。
#### NSOperation的依赖关系
上面在介绍start方法时提到了NSOperation的依赖。可以在不同的operation之间添加依赖关系。假设operationA依赖 operationB，那么只有operationB执行完毕后，operationA的isReady 才为YES，也就是说，依赖关系能够决定operation的执行顺序。

需要注意的是，依赖是operation与operation之间的关系，即使operation处在不同的NSOperationQueue中，依赖关系也是生效的。
#### NSOperation的优先级
NSOperation的定义中有一个属性是：queuePriority。queuePriority是一个枚举类型，其可取的值有：
```Objective-C
typedef NS_ENUM(NSInteger, NSOperationQueuePriority) {
	NSOperationQueuePriorityVeryLow = -8L,
	NSOperationQueuePriorityLow = -4L,
	NSOperationQueuePriorityNormal = 0,
	NSOperationQueuePriorityHigh = 4,
	NSOperationQueuePriorityVeryHigh = 8
};
```
该值表示了一个operation的优先级。在一个队列中，优先级高的operation要先于优先级低的operation执行。
#### NSOperationQueue
NSOperationQueue 用于管理operation。一个NSOperation开始执行有两种方式：一种是直接调用start方法，另外一种就是添加到NSOperationQueue中。

NSOperationQueue中的一些方法和属性：
```Objective-C
@property NSInteger maxConcurrentOperationCount;
- (void)cancelAllOperations;
@property (getter=isSuspended) BOOL suspended;
```
##### maxConcurrentOperationCount
NSOperationQueue中最大的并发量。当maxConcurrentOperationCount的值为1时，operation是一个一个执行，实际上就是一个串行的队列。
##### cancelAllOperations
取消队列中所有的operation。根据上面的介绍，实际上是队列中所有的operation执行cancel方法，也就是将isCancelled置为YES。
##### suspended
如果suspended为YES，那么NSOperationQueue不会执行新的operation。
#### NSOperation vs GCD
在实际的项目开发中，使用GCD的情况可能会多一些，但是在一些特殊的场景中，使用NSOperation可能会更方便，比如说网络请求。那么NSOperation相对于GCD来说，有什么优势呢?

1. NSOperation和NSOperationQueue结合使用能够设置最大的并发量，GCD是不可以的。
2. NSOperation支持取消操作，而GCD不支持。
3. 同一个NSOperationQueue中的NSOperation可以设置不同的优先级，而GCD只能设置队列的优先级。
4. 使用NSOperation可以得到该operation的状态，且状态改变时会触发KVO通知，这一点GCD是不具备的。
5. operation与operation之间支持设置依赖关系，以保证operation执行的先后顺序。GCD不支持依赖关系。

完。
### LRU算法
LRU全称 least recently used,近期最早使用算法，常应用于清除缓存中的数据。

#### 背景
我们知道，内存的成本相对于磁盘的成本是高很多的，因此内存的空间也相对来说小一些。但是，内存的访问速度是远远大于硬盘的。在程序开发中，为了程序效率，会经常读内存中的数据，但是操作系统提供的内存是有限的，当内存达到上限时，应该删除内存中的一部分数据。那么，应该删除哪些数据呢？随便删除数据可能会造成缓存命中率降低，下次读数据的时候效率降低。

LUR提供了一种算法，根据对象的访问时间来删除对象。
#### LRU算法的依据
在详细介绍LRU算法前，先说一下LRU算法的依据：在操作系统中，如果一个元素近期被访问了，那么未来访问他的可能性非常大。

#### LRU算法的过程
举一个LUR算法的例子，就可以明白LRU算法的整个过程。

假设一块内存只能存放5个元素，磁盘上有10个元素A、B、C、D、E、F、G、H、I、J。内存在访问元素时，记录下每个元素的访问时间，如：

1：访问A，  3：00

2：访问B，  3：02

3：访问C，  3：05

4：访问B，  更新B的访问时间 4：00

5：访问F，  4：02，此时内存中共有四个元素

6：访问G，  4：05

假设此时访问了新的元素J，如果需要将新元素J存放到内存中，那么就需要从内存中删除一个元素，LRU算法遵循的原则就是删除访问最早的元素，在该例子中也就是A，因为A的访问时间是最早的。之后再次触发删除缓存中元素的时候，再根据访问时间删除即可。
#### 其他的一些缓存删除算法
需要注意的是，LRU并不是唯一的缓存删除算法，也不是最好的缓存删除算法，有多种缓存删除算法。其他一些常见的缓存删除算法：
1. FIFO (first in first out)
2. LIFO (last in first out)
3. MRU (most recently used) 和LRU对应
4. RR（Random ReplaceMent）
5. ......

在设计一个缓存时，需要结合缓存的使用场景，找到最合适的缓存删除算法，以期得到最高的缓存命中率。

完。
