Android App优化之网络优化
===
 [Protobuffer和json深度对比](http://cxshun.iteye.com/blog/1974498)

[Android 性能优化（八）之网络优化](https://juejin.im/post/58ef22e3b123db0058214c60)

[Android App优化之网络优化](https://www.jianshu.com/p/d4c2c62ffc35)

## 哪些方面取优化网络连接

第一节说到了网络请求对App和用户的影响, 那么我们怎么从哪些方面去优化网络进而减少甚至消灭这些影响呢?

简单来说, 两个方面:

*   **减少Radio活跃时间**

    *   也就是减少网络数据获取的频次.
    *   这就减少了radio的电量消耗, 控制电量使用.
*   **减少获取数据包的大小**

    *   可以减少流量消耗
    *   也可以让每次请求更快, 在网络情况不好的情况下也有良好表现, 提升用户体验.

**那么, 具体应该从哪些方面着手呢?**

### 3.1 接口设计

#### API设计

App与Server之间的API设计要考虑网络请求的频次, 资源的状态等. 以便App可以以较少的请求来完成业务需求和界面的展示.

例如, 注册登录. 正常会有两个API, 注册和登录, 但是设计API时我们应该给注册接口包含一个隐式的登录. 来避免App在注册后还得请求一次登录接口(有可能失败, 从而导致业务流程失败).

再例如, 上文提到的获取repo详情, 实际上请求了4个接口, 请求了repo的信息, forks列表, contributors列表, readme, 这是因为github提供的接口是尽量单一职责的. 然而在我们的实际开发中, 我们的Server除了提供这些单一职责的小接口外, 最好还能组合一个满足客户端业务需求的repo详情接口出来.

#### Gzip压缩

使用Gzip来压缩request和response, 减少传输数据量, 从而减少流量消耗.

#### 考虑使用Protocol Buffer代替JSON

从前我们传输数据使用XML, 后来使用JSON代替了XML, 很大程度上也是为了可读性和减少数据量(当然还有映射成POJO的方便程度).

[Protocol Buffer](https://link.jianshu.com?t=https://github.com/google/protobuf/)是Google推出的一种数据交换格式.

如果我们的接口每次传输的数据量很大的话, 可以考虑下protobuf, 会比JSON数据量小很多.

当然相比来说, JSON也有其优势, 可读性更高.

> 本文以网络流量优化的角度推荐protobuf作为一个选择, 具体还需更具实际情况考虑.

#### 图片的Size

上面Network Monitor中看到的22s到27s之间的有多次请求, 且数据量还很大. 就是在获取图片资源.

图片相对于接口请求来说, 数据量要大得多. 故而也是我们需要优化的一个点.

我们可以在获取图片时告知服务器需要的图片的宽高, 以便服务器给出合适的图片, 避免浪费.

我们现在很多公司的图片资源都是使用第三方的云存储服务的(七牛, 阿里云存储之类的).

以七牛为例, 可以在请求图片的url中添加诸如质量, 格式, width, height等path来获取合适的图片资源:

```
imageView2/<mode>/w/<LongEdge>
                 /h/<ShortEdge>
                 /format/<Format>
                 /interlace/<Interlace>
                 /q/<Quality>
                 /ignore-error/<ignoreError>

```

> 参考[七牛官方文档](https://link.jianshu.com?t=http://developer.qiniu.com/code/v6/api/kodo-api/image/imageview2.html).

### 3.2 网络缓存

适当的缓存, 既可以让我们的应用看起来更快, 也能避免一些不必要的流量消耗.

关于Android App的网络缓存, 请参考[MVP架构实现的Github客户端(4-加入网络缓存)](https://www.jianshu.com/p/faa46bbe8a2e)一文.

### 3.3 打包网络请求

当接口设计不能满足我们的业务需求时. 例如可能一个界面需要请求多个接口, 或是网络良好, 处于Wifi状态下时我们想获取更多的数据等.

这时就可以打包一些网络请求, 例如请求列表的同时, 获取Header点击率较高的的item项的详情数据.

> 可以通过一些统计数据来帮助我们定位用户接下来的操作是高概率的, 提前获取这部分的数据.

### 3.4 监听相关状态

通过监听设备的状态:

*   休眠状态
*   充电状态
*   网络状态

结合[JobScheduler](https://link.jianshu.com?t=https://developer.android.com/reference/android/app/job/JobScheduler.html)来根据实际情况做网络请求. 比方说Splash闪屏广告图片, 我们可以在连接到Wifi时下载缓存到本地; 新闻类的App可以在充电, Wifi状态下做离线缓存.

### 3.5 弱网测试&优化

除了正常的网络优化, 我们还需考虑到弱网情况下, App的表现.

### 3.5.1 弱网测试

有几种方式来模拟弱网进行测试.

#### Android Emulator

创建和启动Android模拟器可以设置网络速度和延迟:

**创建时**:

![](//upload-images.jianshu.io/upload_images/851999-0062476d94a4a87a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

create emulator

**启动时**, 使用emulator命令:

```
$emulator -netdelay gprs -netspeed gsm -avd Nexus_5_API_22

```

具体参数参考[这里](https://link.jianshu.com?t=https://developer.android.com/studio/run/emulator-commandline.html#netdelay)和[这里](https://link.jianshu.com?t=https://developer.android.com/studio/run/emulator-commandline.html#netspeed), 需要翻墙.

#### 使用网络代理工具

以[Charles](https://link.jianshu.com?t=https://www.charlesproxy.com/)为例:
保持手机和PC处于同一个局域网, 在手机端wifi设置高级设置中设置代理方式为手动, 代理ip填写PC端ip地址, 端口号默认8888.

![](//upload-images.jianshu.io/upload_images/851999-e5542fd23c2e2350.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

charles proxy

![](//upload-images.jianshu.io/upload_images/851999-cae8443f32c04b30.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

charles throttling

#### 其他模拟弱网方式

如果你恰好也是iOS的开发者, Apple提供了[Network Link Conditioner](https://link.jianshu.com?t=http://nshipster.cn/network-link-conditioner/), 非常好用.

可以模拟的网络情况与上述类似:

![](//upload-images.jianshu.io/upload_images/851999-d4fdd5270af8537d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/380)

ios_network

如果你使用Linux环境开发, 还可以试下facebook出的[ATC](https://link.jianshu.com?t=http://facebook.github.io/augmented-traffic-control/).

### 3.5.2 弱网优化

利用上述工具模拟弱网, 在弱网情况下体验我们的App. 一般来说, 网络延迟在60ms内, 是OK的, 超过200ms就比较糟糕了. 我们需要做的是在比较糟糕的网络环境下还能给用户较好的体验.

弱网优化, 本质上是在弱网的情况下能让用户流畅的使用我们的App. 我们要做的就是结合上述的优化项:

*   压缩/减少数据传输量
*   利用缓存减少网络传输
*   针对弱网(移动网络), 不自动加载图片
*   界面先反馈, 请求延迟提交
    例如, 用户点赞操作, 可以直接给出界面的点赞成功的反馈, 使用[**JobScheduler**](https://link.jianshu.com?t=https://developer.android.com/reference/android/app/job/JobScheduler.html)在网络情况较好的时候打包请求.
##  网络优化

**网络优化主要从三个方面进行：1\. 速度；2\. 成功率；3\. 流量。**

### Gzip压缩

HTTP协议上的Gzip编码是一种用来改进WEB应用程序性能的技术，用来减少传输数据量大小，减少传输数据量大小有两个明显的好处：

*   **可以减少流量消耗；**
*   **可以减少传输的时间。**

### IP直连与HttpDns；

**DNS解析的失败率占联网失败中很大一种，而且首次域名解析一般需要几百毫秒。针对此，我们可以不用域名，才用IP直连省去 DNS 解析过程，节省这部分时间。**

另外熟悉阿里云的小伙伴肯定知道HttpDns：HttpDNS基于Http协议的域名解析，替代了基于DNS协议向运营商Local DNS发起解析请求的传统方式，可以避免Local DNS造成的域名劫持和跨网访问问题，解决域名解析异常带来的困扰。

### 图片处理

#### 图片下载

*   **使用WebP格式；同样的照片，采用WebP格式可大幅节省流量，相对于JPG格式的图片，流量能节省将近 25% 到 35 %；相对于 PNG 格式的图片，流量可以节省将近80%。最重要的是使用WebP之后图片质量也没有改变。**
*   使用缩略图；**App中需要加载的图片按需加载，列表中的图片根据需要的尺寸加载合适的缩略图即可，只有用户查看大图的时候才去加载原图。**不仅节省流量，同时也能节省内存！之前使用某公司的图片存储服务在原图链接之后拼接宽高参数，根据参数的不同返回相应的图片。

####  图片上传

图片（文件）的上传失败率比较高，不仅仅因为大文件，同时**带宽、时延、稳定性等因素在此场景下的影响也更加明显；**

*   **避免整文件传输，采用分片传输；**
*   **根据网络类型以及传输过程中的变化动态的修改分片大小；**
*   **每个分片失败重传的机会。**

**备注：图片上传是一项看似简单、共性很多但实际上复杂、需要细分的工作。移动互联网的场景和有线的场景是有很多区别的，例如移动网络的质量/带宽经常会发生“跳变”，但有线网络却是“渐变”。**

> 图片上传其它细节请参见《移动App性能评测与优化》一书。

###  协议层的优化

使用最新的协议，Http协议有多个版本：0.9、1.0、1.1、2等。新版本的协议经过再次的优化，例如：

*   **Http1.1版本引入了“持久连接”，多个请求被复用，无需重建TCP连接，而TCP连接在移动互联网的场景下成本很高，节省了时间与资源；**
*   **Http2引入了“多工”、头信息压缩、服务器推送等特性。**

新的版本不仅可以节省资源，同样可以减少流量；我对Http2并没有实际接入经验，此处仅从原理进行分析。

###  请求打包

**合并网络请求，减少请求次数。**对于一些接口类如统计，无需实时上报，将统计信息保存在本地，然后根据策略统一上传。这样头信息仅需上传一次，减少了流量也节省了资源。

###  网络缓存

**对服务端返回数据进行缓存，设定有效时间，有效时间之内不走网络请求，减少流量消耗。对网络的缓存可以参见[HttpResponseCache](https://link.juejin.im?target=https%3A%2F%2Fdeveloper.android.com%2Freference%2Fandroid%2Fnet%2Fhttp%2FHttpResponseCache.html)。**

**备注：我们也可以自定义缓存的实现，一些网络库例如：Volley、Okhttp等都有好的实践供参考。**

###  网络状态

根据网络状态对网络请求进行区别对待，2G与Wifi状态下网络质量肯定是不一样的，那对应的网络策略也应该是不一样的。例如：在Wifi场景下可以进行数据的预取、一些统计的集中上传等；而在2G场景下此类操作以及网络请求的次数策略都应该调低。网络状态可以由[TelephonyManager](https://link.juejin.im?target=https%3A%2F%2Fandroid.googlesource.com%2Fplatform%2Fframeworks%2Fbase%2F%2B%2Fandroid-6.0.1_r77%2Ftelephony%2Fjava%2Fandroid%2Ftelephony%2FTelephonyManager.java).getNetworkType()方法获取到。

![](https://user-gold-cdn.xitu.io/2017/4/14/5ac43a2ef5810034270ad8cefbf56358.jpg?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**备注：还可以使用Facebook的开源库[network-connection-class](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Ffacebook%2Fnetwork-connection-class)来做网络状态的判断。**

###  其它

*   断点续传，文件、图片等的下载，采用断点续传，不浪费用户之前消耗过的流量；

*   重试策略，一次网络请求的失败，需要多次的重试来断定最终的失败，可以参考Volley的重试机制实现。

*   Protocol Buffer
    Protocol Buffer是Google的一种数据交换的格式，它独立于语言，独立于平台。相较于目前常用的Json，数据量更小，意味着传输速度也更快。

    具体的对比可以参见：[《Protobuffer和json深度对比》](https://link.juejin.im?target=http%3A%2F%2Fcxshun.iteye.com%2Fblog%2F1974498)。

*   尽量避免客户端的轮询，而使用服务器推送的方式；

*   数据更新采用增量，而不是全量，仅将变化的数据返回，客户端进行合并，减少流量消耗；

##  其它

*   **对于网络优化，实际上和内存优化一样，是一项投入巨大的事情。提升网络的成功率尤为困难。因此建议优先进行流量优化，减少干扰项；**
*   **弱网不仅仅指代网络不好，移动互联网的网络带宽很容易出现“跳变”，下一秒的传送速度可能降到前一秒的几十分之一；而且即便是信号满格也传不出一个字节；**
*   **对于真正的弱网，可以使用抓包工具进行模拟，也有聪明的小伙伴使用wifi精灵进行限速；**
*   **Facebook的开源项目[augmented-traffic-control](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Ffacebook%2Faugmented-traffic-control)可以模拟不同的网络环境，针对带宽、时延抖动、丢包率、错包率、包重排序率等方面，堪称弱网调试神器；**
