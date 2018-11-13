### IAP介绍
IAP,是in-App Purchase的缩写，可以理解为在App内购买，这也是为何IAP又被称为内购的原因。

苹果规定，凡是在App内提供的服务需要付费时，必须使用IAP，比如说游戏的金币，道具等；而在App外提供的服务需要付费时，可以使用其他的支付方式，比如支付宝SDK、微信SDK等。说的更通俗一点，如果付费购买的商品是虚拟商品，比如游戏中的道具，并不是现实中存在的，那么必须使用IAP；如果付费购买的商品是真实产品，比如在淘宝中买了件衣服，是实实在在存在的，那么没有必要使用IAP。因此，在使用IAP之前，首先要确认是否一定要使用IAP，如果不使用IAP也可以，那么尽量不要用IAP，因为IAP流程、使用复杂度相比支付宝SDK、微信SDK来说，要复杂很多。
### 创建商品
使用IAP之前，首先需要创建商品，创建商品是在iTnuesConnect内。IAP内有四种商品类型，分别是消耗型商品、非消耗型商品、非续期订阅、自动续期订阅。下面介绍下每种商品的特征。
#### 消耗型商品
消耗型商品，可以理解为可以使用，且使用之后就没了的商品。比如游戏中的钻石，使用钻石可以购买app内的其他虚拟物品，但是使用之后钻石也就没了。这种商品就是消耗型商品。
#### 非消耗型商品
非消耗型商品和消耗型商品的区别主要是：一次购买，终身可用。比如说，购买了一门课程，这门课程并不会随着用户学完了而消失，该课程会一直存在。这种商品就是非消耗型商品。
#### 非续期订阅型商品
非续期订阅的商品和消耗型商品有相似之处，区别在于，非续期订阅的商品是有有效期的，且有效期是开发者自己的服务器来控制的。比如说视频app的会员，会员有月会员、季度会员、年会员等，会员是有到期时间的，过了到期时间之后，就享受不到该服务。如视频会员这种商品，通常都是非续期订阅型商品。
#### 自动续期订阅型商品
自动续期订阅型商品同样有有效期，和非续期订阅型商品的不同之处在于，自动续费订阅型商品在有效期到期的前一天，会尝试自动续期，自动续期之后，开发者服务器上的有效期也应该对应延长。比如视频网站的连续包月会员，就是自动续期订阅型商品。
#### iTunesConnect内创建商品
首先登录iTnuesConnect,选择我的App，然后在对应的App下，点击功能，选中App内购买项目，如图：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE0f145a84385fe8fd4e3f3d47e1edc8e7/13205)

之后点击右侧的加号按钮，也就是新建一个IAP商品，之后会让我们选择商品类型，这里选择非续期订阅型商品，如下图：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCEff746943e2acf6dd138549dc20bd4a70/13208)

之后就是填写一些商品信息，如商品名称、商品价格等。

参考名称：参考名称不会显示在用户面前，而是显示在一些报告中，比如可以命名为1_month_vip,代表一个月的vip。

产品ID：产品ID必须是唯一的，命名方式类似于包名，如com.test.iOS.1_month_vip。

价格：选择对应的商品价格即可。

本地化版本-显示名称：注意，这里的名称是显示给用户看的，比如vip月卡。

本地化版本-描述：描述可以显示给用户看，也可以不显示给用户看。

审核信息-屏幕快照：新建商品阶段可以不用管这里，待开发完成后，在商品列表页面截张图，然后放到这里即可。

至此，新建IAP商品完毕。需要注意的是，商品创建完毕后是需要苹果审核的，只有苹果审核通过之后，才可以展示给用户。
### 申请测试账号
开发过程中，如果需要测试IAP，那么需要申请测试账号。申请测试账号也是在iTunesConnect上，申请测试账号的过程如下：
1. 首先登录iTunesConnect
2. 点击用户和访问图标，如下图：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE94886c35334c68417967f83debfc5384/13247)
3. 选择页面左下角的沙箱技术-测试员图标，如下图

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE52d5f087fde09080e9c46ba050f973ed/13251)
4. 新建测试员
5. 使用测试员账号登录进行测试

使用测试账号购买商品时，是不需要花钱的。下面会讲到开发环境和生产环境，如果使用的是测试账号登录、付费，那么服务器在验证时会向生产环境验证；否则会向开发环境验证。

上面提到，很多的IAP商品是有期限的，比如1周，1月，1年，那么在测试的过程中如何测试期限呢？真的等这么久肯定是不现实的，这一点上，苹果做的是比较人性化的。测试账号的时间和真实的时间有一个对应关系，就是为了方便测试。对应关系如下：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE830c9db4a29050b2453c2910dcc240cb/13270)

在测试自动续期订阅型商品时，该对应关系会非常有用。
### 内购流程
苹果的内购流程步骤比较多，内购流程中涉及到三个角色：客户端，App Store服务器，开发者自己的服务器。苹果提供的内购框架是StoreKit，涉及到内购的，需要在项目中引入该框架。在后续中，会看到很多和内购相关的类是以SK开头，也是由于这个原因。下面介绍下苹果的内购流程。
1. 客户端向开发者服务器发请求，请求要展示的商品列表
2. 开发者服务器返回对应客户端需要展示的商品id数组，这里的id就是在iTunesConnect中创建商品时的id

    注意：在部分客户端实现中，前两步是可以省略的，具体实现是客户端将所需要展示的商品ids记录的客户端本地，直接进行第三步，节省这次的服务器请求时间。但是这种方式也有明显的缺陷，即如果想修改展示的套餐，只能通过重新发版来解决，不能通过服务端控制。具体使用哪种方式，可以根据自己的需求来选择。
3. 客户端根据获得到的需要展示的产品ids，向appStore服务器请求详细的产品信息，包括产品价格，产品描述等。从appStore服务器获取产品信息需要使用StoreKit中的API。实现代码如下：
```
// 创建SKProductsRequest，并且其delegate为self，这里的delegate是SKProductsRequestDelegate
- (void)requestProductsFromAppStore:(NSString *)productID {
    NSSet * set = [NSSet setWithObject:productID];
    SKProductsRequest * request = [[SKProductsRequest alloc] initWithProductIdentifiers:set];
    request.delegate = self;
    [request start];
}

#pragma mark - SKProductsRequestDelegate方法
- (void)productsRequest:(SKProductsRequest *)request didReceiveResponse:(SKProductsResponse *)response {
    // 接收到appStore返回的产品的详细信息，这里的product是SKProduct类型
    self.products = response.products;
}

```
4. 客户端接收到appStore服务器返回的详细信息后，负责展示和渲染UI。至此，第一阶段结束。
5. 当用户点击购买按钮时，客户端负责向appStore服务器发起购买请求
6. appStore负责处理购买行为，包括用户付款，输入appleID、密码等行为。用户付款成功后，客户端做相应的处理。至此，第二阶段结束。购买行为需要用到StoreKit中的API，核心代码如下：
```
// (1)首先成为[SKPaymentQueue defaultQueue]的观察者，这里self类需要遵守SKPaymentTransactionObserver协议
if ([SKPaymentQueue defaultQueue]) {
    [[SKPaymentQueue defaultQueue] addTransactionObserver:self];
}

// (2)创建payment，并添加到支付队列中
- (void)buyProduct:(SKProduct *)productIdentifier onCompletion:(IAPbuyProductCompleteResponseBlock)completion {
    // 根据productId创建一个商品的payment，并将其添加到支付队列中
    SKPayment *payment = [SKPayment paymentWithProduct:productIdentifier];
    if ([SKPaymentQueue defaultQueue]) {
        [[SKPaymentQueue defaultQueue] addPayment:payment];
    }
}

// (3)实现SKPaymentTransactionObserver协议的- (void)paymentQueue:(SKPaymentQueue *)queue updatedTransactions:(NSArray *)transactions方法
// 该方法监听交易状态改变信息，根据交易状态，分别做对应的处理
- (void)paymentQueue:(SKPaymentQueue *)queue updatedTransactions:(NSArray *)transactions
{
    for (SKPaymentTransaction *transaction in transactions)
    {
        switch (transaction.transactionState)
        {
            case SKPaymentTransactionStatePurchased:
                [self completeTransaction:transaction];
                break;
            case SKPaymentTransactionStateFailed:
                [self failedTransaction:transaction];
                break;
            case SKPaymentTransactionStateRestored:
                [self restoreTransaction:transaction];
            default:
                break;
        }
    }
}

// (4)无论是交易成功，还是交易失败，最终都要从交易队列中移除交易
[[SKPaymentQueue defaultQueue] finishTransaction: transaction];

```
7. 注意，这里的交易完成并不是最终的结束，客户端还需要做进一步的处理。交易完成后，appStore服务器会返回给客户端一个收据（receipt）。客户端需要将该收据发送给开发者自己的服务器，注意收据必须要使用base64编码，核心代码如下：
```
- (void)checkReceiptWithTransaction:(SKPaymentTransaction *)transaction
{
    // 将AppStore返回的收据（receipt）发送到服务器进行校验
    NSURL *receiptUrl = [[NSBundle mainBundle] appStoreReceiptURL];
    if ( [[NSFileManager defaultManager] fileExistsAtPath:[receiptUrl path]] ) {
        // 使用base64编码
        NSData *receiptData = [NSData dataWithContentsOfURL:receiptUrl];
        NSString *receiptBase64 = [receiptData base64EncodedStringWithOptions:0];
        
        // 发送网络请求,到开发者自己的服务器进行验证
        ...
        // 验证成功后，客户端将该交易从交易队列中移除
        [[SKPaymentQueue defaultQueue] finishTransaction:transaction];
    }
}
```
8. 开发者自己的服务器收到客户端发来的收据时，需要向appStore服务器校验该收据是否合法，注意该请求必须是post请求。校验地址根据开发环境和生产环境分别对应不同的url。

开发环境的校验地址是：https://buy.itunes.apple.com/verifyReceipt

生产环境的校验地址是：https://sandbox.itunes.apple.com/verifyReceipt

校验时如果有问题，苹果会返回对应的错误码，错误码如下：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE39155a4e6338a4d4de6c917f229c0197/13370)

9. 开发者自己服务器校验完成后，根据校验结果，成功或者失败，做对应的处理。举例来说，如果用户购买的是视频VIP月卡，那么校验成功后，服务端需要对用户VIP时间做对应的处理。之后，通知客户端。
10. 客户端收到开发者自己服务器通知后，做对应的处理，如刷新个人信息。至此，内购流程全部结束。

网上有一张图片对整个流程描述的比较清晰，这里看一下：

![image](https://note.youdao.com/yws/public/resource/bba39d75a3d87a96f65a409a0b99df90/xmlnote/WEBRESOURCE70396c77d5f648a2b903734ab1e943c9/13388)

#### 内购时的两种边界情况
上面提到的流程是网络畅通、app稳定运行情况下的流程，倘若用户网络不太好，或者是app突然崩溃，就会出现一些边界情况，这里介绍两种边界情况。
##### appStore服务器返回给客户端的收据信息出问题
当appStore服务器处理完用户的购买请求，将receiptData返回给客户端时，客户端出现了断网情况，或者是客户端发生了崩溃，这样客户端就收不到receipt，但是用户已经付过钱了。这种情况怎么处理呢？所幸，苹果已经考虑到了这种情况，我们只需要在app启动时，成为[SKPaymentQueue defaultQueue]的observer即可。比如说让AppDelegate类遵守SKPaymentTransactionObserver协议。这样，app启动时，苹果会负责改变交易的状态，这样，客户端实现交易状态改变的监听方法，如果该方法被调用，则检测是否有完成的交易，如果有完成的交易，则发送到开发者自己的服务器进行校验，然后进行后续的操作即可。
##### 客户端向开发者自己服务器发送数据失败
客户端在向开发者自己的服务器发送交易数据时，如果客户端断网，或者服务器发生了错误，服务器没有收到客户端的交易数据，发生这种情况时，客户端是不能将交易从交易队列中移除的。合理的处理方式是，遇到这种情况时，客户端应该定时向服务器发请求，直到服务器正常收到客户端的交易数据，然后按照后续的流程即可。还有一种边界情况是，在客户端向服务器发交易数据的瞬间，客户端崩溃了，这样就会造成交易数据的丢失。因此，在收到appStore返回的交易数据后，客户端应该将首先其保存到本地，然后再向服务器发送请求，直到该交易最终完成，才将存在本地的交易删除。
### IAPHelper
通过上面的介绍可以了解到，苹果的IAP流程确实是比较繁琐的。幸运的是，网络上已经有一些框架封装了IAP的核心操作。我们项目中使用的是IAP，使用比较简单，不做太多的介绍。有一点需要注意的是，IAPHelper甚至在客户端实现了校验receipt的逻辑，可以根据需求选择使用。