#混合开发（Hybrid）
混合开发是 Native 和 Web 技术一起用，开发者以 Native 代码为主体，在合适的地方部分使用 Web 技术。比如在 Android 中的 Activity 内放置一个 Webview（一个浏览器引擎，只拥有渲染 HTML，CSS 和执行 JavaScript 的核心功能）。这样，部分用户界面就可以在 WebView 中使用 Web 技术实现。

促使我们在移动开发中使用 Web 技术主要动力在于，相比于 Native 技术，Web 技术具有诸多优势：

* 高效率的界面开发：HTML，CSS，JavaScript 的组合被证明在用户界面开发方面具有很高的效率。

* 跨平台：统一的浏览器内核标准，使得 Web 技术具有跨平台特性。iOS 和 Android 可以使用一套代码。

* 热更新：可越过发布渠道自主更新应用。

这些优势都和开发效率有关。Web 技术具有这些优势的原因是，Web 技术是一个开放标准。基于开放的标准已经发展出来庞大生态，而且这个生态从 PC 时代发展至今已积累多年，开发者可以利用生态中产出的各种成果，从而省去很多重复工作。

在大型移动应用的开发中，项目代码庞杂，通常还需要 iOS，Android，移动 Web 和 桌面 Web 全平台支持。相对于同时开发几个版本，使用混合开发显然可以在代码重用、开发成本和效率方面有很大的优势，在权衡性能体验的前提下，使用混合开发是非常现实的选择。

#Rexxar 是什么
Rexxar是一个针对移动端的混合开发框架。支持Android、iOS和移动Web。

####Rexxar主要由三部分组成：

* Rexxar-web：前端代码库。包括一套打包、调试、发布工具，以及公共前端组件，和对Rexxar Container实现的Widget的调用。

* Rexxar-Router：路由表，将每个页面分配一个服务器端链接，以及一个本地URI，通过路由表来访问页面。

* Rexxar-container：增强版WebView，封装了一些Native API支持，包括OAuth授权、图片缓存等。

Rexxar目前已经开源，并且分为3个项目，你可以只使用其中某个项目来开发对应平台的代码：

* Rexxar Web：https://github.com/douban/rexxar-web

* Rexxar iOS：https://github.com/douban/rexxar-ios

* Rexxar Android：https://github.com/douban/rexxar-android

####Rexxar 主要由以下三部分组成：

 * **Rexxar Route**，我们使用 URL 来标识每一个页面。在 App 中通过指明 URL 跳转到此页面。所以，需要一个路由表。通过路由表可以根据 URL 找到一个 Rexxar Web 的对应资源来正确展示相应页面；

* **Rexxar Web**，前端代码库，由 HTML、CSS、JavaScript、Image 等组成，用来提供在移动客户端使用的用户页面；

* **Rexxar Container**，一个前端代码的运行容器。它其实是一个内嵌的浏览器(WebView)，我们为内嵌浏览器提供了一些必要的原生端支持，包括 API 的 OAuth 授权、图片缓存、Native UI 组件的调用等；现在有 Android 和 iOS 两个版本的实现。

在项目实践中，Rexxar Web 和 Rexxar Route 由一个项目实现，并部署于同一个 Web 项目中。

####**Rexxar Route**

Rexxar Route 比较简单，只需要表达一个路由表即可。我们使用了一个 json 文件来表达路由表。
![](http://ou21vt4uz.bkt.clouddn.com/rexxar_routes.png)

发布的每个版本的 App 安装包都会包含最新版本的 routes.json 文件。在 App 启动时，都会尝试下载最新版本的 routes.json。在遇到无法解析的 URL 时，也会去下载新版 routes.json,或者根据 URL 查询本机缓存的路由表 routes.json不能找到对应的资源记录时，也会请求 Rexxar Route 服务，获得最新的全量路由表 routes.json，更新本地缓存。

####**Rexxar Web**

Rexxar Web 是 Rexxar 前端实现。Rexxar Container 的实现和 Rexxar Web 的实现是分离的。Rexxar Container 对 Rexxar Web 使用何种技术实现并不关心。所以，可以选择自己的前端技术和 Rexxar Container 进行组合。豆瓣选择的是 React 作为前端开发框架。

####**Rexxar Container**

Rexxar Container 是提供了一个运行前端代码的容器。它也是一个内嵌的浏览器（WebView）。不是只简单的load一个 URL 地址，还对内嵌的浏览器做了很多开发，为其包装了很多附加功能。

#### **Rexxar Container 的技术实现**

Rexxar Container 主要的工作是截获 Rexxar Web 的数据请求和原生功能请求。Rexxar Container 截获请求之后，做相应的反应。这种 Native 和 Web 的交互被抽象成三种接口:

   * Decorator：修改数据请求。例如，数据请求加上 OAuth 认证信息。
    
   * Widget: 调用某些 Native UI 组件。例如，调起一个 Toast。
    
   * ContainerAPI：给 Web 一个 Native 的计算结果。例如，给出当前位置信息。

这三种接口都是由 Rexxar Web 发起某种形式的 URL 调用的。Rexxar Web 的业务代码在 App 的 Rexxar Container 内工作方式，和在普通浏览器里差别不大。我们只是在 Web 技术的基础上做了一些拓展，保留了大部分 Web 原有的编写和运行方式。代码都是标准 Web 式的，没有为原生移动开发做太多定制。因此，移植到 Web 平台，在各种浏览器中，代码无需做太多修改就可以正确运行。以 URL 作为协议，也为 Web 和 Native 划定了清晰的边界和数据传递方式。

Rexxar在iOS 和 Android 各开发了一个 Rexxar Container。iOS 和 Android 平台截获请求的方式由于平台差异，并不完全相同。但本质上都是在 Web 和 Native 之间实现了一个 Proxy。Web 发出的请求会被 Proxy 预先处理。要么是修改后再发出去，要么是由 Rexxar Container 自己处理。

具体的实现可以参看两个平台的项目代码。

#### **Rexxar Container 需要实现的功能**

* Rexxar Route 路由表的更新，已经在客户端的保存；

* 为 Rexxar Web 前端代码发出的 API 请求提供包装。带上必要的 OAuth 参数；

* 缓存 Rexxar Web 前端代码所需要的静态文件，包括 HTML、CSS、JavaScript、Image(图片素材)等；
    
* 缓存 Rexxar Web 中所需要加载的资源文件，例如图片等；
    
* 通过协议为 Rexxar Web 提供一些原生支持的功能：包括 Native UI 组件调用，获取 Native 的计算结果。

####**Rexxar Container 和 Rexxar Web 之间的交互**

混合开发实践中，都会涉及到 Native 和 Web 如何通信的问题。这是因为我们把一件事情交给两种技术完成，那么它们之间便会存在有一些通信和协调。在 Rexxar 中是通过从 Rexxar Web 发出 HTTP 请求的方式，由 Rexxar Container 截获的方式进行通信。Native 和 Web 之间协议是由 URL 定义的。Rexxar Web 访问某个特定的 URL, Rexxar Container 截获这些 URL 请求，调用 Native 代码完成相应的功能。

例如，Rexxar 中 UI 相关的功能的协议如下：

* 请求 douban://rexxar.douban.com/widget/nav_title，可以定义 Navigation Bar Title。
    
* 请求 douban://rexxar.douban.com/widget/nav_menu，可以定义 Navigation Bar Button。

* 请求 douban://rexxar.douban.com/widget/toast，可以出现一个消息通知 toast。

Rexxar Web 具体前端实现是在 DOM 中加入一个 iframe 来加载此 URL，以来完成对 Rexxar Container 的通知。

用 Rexxar 完成的页面，不仅仅在 App 内使用，还会在移动 Web 页面上使用。还有其他的移动站点，例如分享到外部（如微信，微博，QQ等）的页面希望复用 Rexxar 在 App 内的工作成果。这样 Native 和 Web 之间的通信是可定义的，可控的则需要我们将 Native 和 Web 的通信以协议的形式规范起来。一些协议在移动 Web 是被可以自动被忽略（比如，nav_title, nav_menu），或者用移动 Web 支持的形式再实现一次（比如，toast）。这样，Rexxar 中的前端业务代码无需太多改动，即可迁移到移动 Web 和桌面 Web 端。

# rexxar在移动端如何加载、解析、渲染web模板

####**Rexxar 的工作流程图**

![](http://ou21vt4uz.bkt.clouddn.com/Rexxar%20%E9%A1%B5%E9%9D%A2%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B.png)

####**Rexxar 页面执行过程**

![](http://lincode.github.io/images/blog/Rexxar.png)

####**Rexxar 的工作流**

客户端接到一个页面请求，要打开一个 URL：douban://douban.com/rexxar_demo。

* 1.根据 URL 查询本机缓存的路由表 routes.json，看是否能够找到对应的资源记录(一般是一个 HTML 文件)。如果找到不到，请求 Rexxar Route 服务，获得最新的全量路由表 routes.json，更新本地缓存，找到对应的资源记录；

* 2.根据路由表指示的 HTML 文件的路径，看本地是否找到对应的文件。如果找不到，请求 Rexxar Web 资源服务器，更新本地缓存；在 Rexxar Container 里展示该 HTML 文件；如有需要，会在 Container 中请求图片资源，图片资源也有缓存，Rexxar Container 会先检查本地缓存。如不存在，会请求 CDN 的图片或者图片服务器；

* 3.Rexxar Web 前端代码在 Container 里继续执行，发出 API 请求。Rexxar Container 代理这些请求，为 API 请求添加 OAuth 验证，或增加某些参数；

* 4.Rexxar Web 前端代码继续执行，根据 API 返回的结果，展示响应的页面，可能会请求 CDN 的图片或者图片服务器等；

* 5.Rexxar Web 前端代码继续执行，如果需要修改 NavigationBar 等原生界面，可能通过定义好的协议请求 URL： douban://rexxar.douban.com；

* 6.Rexxar Container 拦截请求，按定义好的协议作出反应。例如，修改 NavigationBar 上的按钮。如果需要，会向 Rexxar Web 回调约定好的 Javascript 函数。

#rexxar的使用流程

### 安装

#### gradle

```groovy
   compile 'com.douban.rexxar:core:0.3.5.3'
```


### 配置

#### 1. 初始化

在Application的`onCreate`中调用

```Java
  Rexxar.initialize(Context context);
```

#### 2. 设置路由表文件 api：

```Java
  RouteManager.getInstance().setRouteApi("https://raw.githubusercontent.com/douban/rexxar-web/master/example/dist/routes.json");
```

Rexxar 使用 uri 来标识页面，提供一个正确的 uri 就可以打开对应的页面，路由表提供了每个 uri 对应的 html 资源的下载地址。

Demo 中的路由表如下：

```json

{
  "items": [
    {
      "deploy_time": "Sun, 09 Oct 2016 05:54:22 GMT",
      "remote_file": "https://raw.githubusercontent.com/douban/rexxar-web/master/example/dist/rexxar/demo-252452ae58.html",
      "uri": "douban://douban.com/rexxar_demo[/]?.*"
    }
  ],
  "partial_items": [
    {
      "deploy_time": "Sun, 09 Oct 2016 05:54:22 GMT",
      "remote_file": "https://raw.githubusercontent.com/douban/rexxar-web/master/example/dist/rexxar/demo-252452ae58.html",
      "uri": "douban://partial.douban.com/rexxar_demo/_.*"
    }
  ],
  "deploy_time": "Sun, 09 Oct 2016 05:54:22 GMT"
}


```

#### 3. 设置需要代理或缓存的请求host

```Java
  ResourceProxy.getInstance().addProxyHosts(List<>() hosts);
```

Rexxar是通过`WebViewClient`的`shouldInterceptRequest`方法来拦拦截请求，请求线上数据并返回给'webview'。为了减少不必要的流程破坏，只有明确需要拦截的hosts（支持正则）的请求才会被拦截代理，并根据mime-type决定哪些内容需要缓存。

#### 4. 预置资源文件

使用 Rexxar 一般会预置一份路由表，以及资源文件在应用包中。这样就可以减少用户的下载，加快第一次打开页面的速度。在没有网络的情况下，如果没有数据请求的话，页面也可访问。这都有利于用户体验。
预置文件路径是`assets/rexxar`, 暂不支持修改。



### 使用 RexxarWebView

直接使用 `RexxarWebView` 为的混合开发客户端容器。或者也可以在 `RexxarWebView` 基础上实现你自己的客户端容器。

为了初始化 RexxarWebView，你需要只一个 url。在路由表文件 api 提供的路由表中可以找到这个 url。这个 url 标识了该页面所需使用的资源文件的位置。Rexxar Container 会通过 url 在路由表中寻找对应的 javascript，css，html 资源文件。

```Java
  // 根据uri打开指定的web页面
  mWebView.loadUri("douban://douban.com/rexxar_demo");
```

## 定制你自己的 Rexxar Container

我们暴露了三类接口。供开发者更方便地扩展属于自己的特定功能实现。

### 定制 RexxarWidget

Rexxar Container 提供了一些原生 UI 组件，供 Rexxar Web 使用。RexxarWidget 是一个 Java 协议（Protocol）。该协议是对这类原生 UI 组件的抽象。如果，你需要实现某些原生 UI 组件，例如，弹出一个 Toast，或者添加原生效果的下拉刷新，你就可以实现一个符合 RexxarWidget 协议的类，并实现以下方法：`getPath:`, `handle:`。

在 Demo 中可以找到一个例子：`TitleWidget` ，通过它可以设置导航栏的标题文字。

```Java

    public class TitleWidget implements RexxarWidget {

    static final String KEY_TITLE = "title";

    @Override
    public String getPath() {
        return "/widget/nav_title";
    }

    @Override
    public boolean handle(WebView view, String url) {
        if (TextUtils.isEmpty(url)) {
            return false;
        }
        Uri uri = Uri.parse(url);
        if (TextUtils.equals(uri.getPath(), getPath())) {
            String title = uri.getQueryParameter(KEY_TITLE);
            if (null != view && view.getContext() instanceof Activity) {
                ((Activity)view.getContext()).setTitle(Uri.decode(title));
            }
            return true;
        }
        return false;
    }
}
```

### 定制 RexxarContainerAPI

我们常常需要在 Rexxar Container 和 Rexxar Web 之间做数据交互。比如 Rexxar Container 可以为 Rexxar Web 提供一些计算结果。如果你需要提供一些由原生代码计算的数据给 Rexxar Web 使用，你就可以选择实现 RexxarContainerAPI 协议（Protocol），并实现以下三个方法：`getPath:`, `call:`。

在 Demo 中可以找到一个例子：`LocationAPI`。这个例子中，`LocationAPI` 返回了设备所在城市信息。当然，这个 ContainerAPI 仅仅是一个示例，它提供的是一个假数据，数据永远不会变化。你当然可以遵守 `RexxarContainerAPI` 协议，实现一个类似的但是数据是真实的功能。

```Java

    static class LocationAPI implements RexxarContainerAPI {

        @Override
        public String getPath() {
            return "/loc";
        }

        @Override
        public Response call(Request request) {
            Response.Builder responseBuilder = newResponseBuilder(request);
            try {
                JSONObject jsonObject = new JSONObject();
                jsonObject.put("lat", "0.0");
                jsonObject.put("lng", "0.0");
                responseBuilder.body(ResponseBody.create(MediaType.parse(Constants.MIME_TYPE_JSON), jsonObject.toString()));
            } catch (Exception e) {
                e.printStackTrace();
            }
            return responseBuilder.build();
        }
    }
```


### 定制 Rexxar Decorator

如果你需要修改运行在 Rexxar Container 中的 Rexxar Web 所发出的请求。例如，在 http 头中添加登录信息，你可以自定义OkHttpClient，`Rexxar.setOkHttpClient(OkHttpClient okHttpClient)`

在 Demo 中可以找到一个例子：`AuthInterceptor`。这个例子为 Rexxar Web 发出的请求添加了登录信息。

```Java

    public class AuthInterceptor implements Interceptor{

        @Override
        public Response intercept(Chain chain) throws IOException {
            Request request = chain.request();

            String url = request.url().toString();
            if (TextUtils.isEmpty(url)) {
                return null;
            }

            Request.Builder builder = request.newBuilder();
            builder.header("Authorization", "123456789");
            return chain.proceed(builder.build());
        }
    }

    // Rexxar初始化时设置
    Rexxar.setOkHttpClient(new OkHttpClient().newBuilder()
            .retryOnConnectionFailure(true)
            .addNetworkInterceptor(new AuthInterceptor())
            .build());
```

## 高级使用

### native调用js方法

```

    // 方法名
    RexxarWebView.callFunction(String functionName)
    
    // 方法名和json数据
    RexxarWebView.callFunction(String functionName, String jsonString)
    
```
  
## Partial RexxarWebView

如果，你发现一个页面无法全部使用 Rexxar 实现。你可以在一个原生页面内内嵌一个 `RexxarWebView`，部分功能使用原生实现，另一部分功能使用 Rexxar 实现。


Demo 中的给出了一个示例。

```
 mRexxarWebView.loadPartialUri("douban://douban.com/rexxar_demo");
```

#rexxar使用过程中的限制及注意事项

* 1.性能方面

Web 的性能没法和 Native 相比。这种状况可能会长期存在。因为，前端代码运行于内嵌浏览器之上，和直接调用原生系统相比，理论上总会存在性能上的差距，可以用规避的方式面对性能问题：即性能问题会明显影响到用户体验时，我们就不使用 Rexxar 来做，而是使用传统 Native 写两份代码，一份 iOS，一份 Android。这样就限缩了 Rexxar 的使用范围。

* 2.内存问题

Rexxar在客户端的实现其实就是一个定制了更多功能的WebView。由于Rexxar使用的是系统的WebView。所以对App的体积没有影响。但是Rexxar同时使用很多个WebView带来的内存问题，这是需要注意的。

* 3.错误报告

Rexxar的Crash有两种：

* 一种是JavaScript的错误，也就是应用逻辑的问题。这类错误他们在WebView中做了捕获，然后通过App的日志系统发回服务器。
    
* 一种是WebView的Crash，这种错误WebView自己无法捕获，现在是通过fabric，Umeng，bugly这种原生的Crash收集系统收集。

在应用中使用 Rexxar 之后，在收集到的 Crash Report 中，JavaScript 的相关错误，和浏览器相关的错误开始增加。而对这类错误，由于移动应用的使用环境更为复杂，错误报告经过了 JavaScript 引擎，原生系统两层之后，给出的错误信息并不够明确。豆瓣在这方面的经验也并不多，现在还没有很好的办法降低这类错误。这对提高 App 的稳定性带来了问题。

#为什么不用PhonGap/Cordova

在混合开发中早已有了很成熟的方案，就是PhoneGap和它的后继者Cordoba. 为什么豆瓣还要造自己的轮子呢？

郭麟说，如果Hybrid方案定义为前端和原生技术的混合使用，那他们认为PhoneGap/Cordova严格来说不算是Hybrid方案，因为它的目标是全面使用前端技术开发移动应用，而不是前端和原生技术混合使用。但是，包括Cordova，还可以加上React Native，以及Rexxar的目标是一致的：使用前端技术来开发移动应用，提高工程效率。

豆瓣实际上使用PhoneGap开发过一款移动App，并在AppStore上架了，这个应用叫豆瓣音乐人，因此，其实豆瓣对PhoneGap/Cordova已经有一定了解和使用经验。为何在开发豆瓣App时又造了一个叫Rexxar的“轮子”呢？这是因为，他们对PhoneGap/Cordova这个项目的理念并不完全赞同，Rexxar的出发点和PhoneGap/Cordova并不一样。

PhoneGap/Cordova这个项目极具野心。它希望完全使用前端技术完成移动开发。所以，可以看到它尽力让前端技术完成尽量多的开发工作，只在前端无法直接调用的原生系统功能方面提供了前端可用的接口。主流的PhoneGap/Cordova项目将业务逻辑都实现在一个WebView中。目标是，让开发者只使用前端技术就可以完成一个移动应用的所有开发工作。这种做法需要有一个前提：前端技术可以解决移动开发的所有需求。他们认为PhoneGap/Cordova这个理念在现阶段有些过于理想化了，或者说过于激进了。

Rexxar则相对实际，或者说保守一些。郭麟表示，他们仍然认为，**现阶段，甚至在相当遥远的未来，移动开发中前端技术都不太可能完全代替原生技术。**但他们同时承认，**移动开发中总是存在部分功能是适合使用前端技术完成的。**在他们的认识中，前端技术和原生技术应该是共存的。移动开发中，前端技术不会完全代替原生技术；而有了前端技术的加入，移动开发的效率会提高。基于这种认识，豆瓣开发了Rexxar。

可以看到，**Rexxar立足于在一个原生项目使用前端技术，而不是整个项目都使用前端技术实现。**他们甚至提供一个页面部分使用Rexxar完成，部分使用原生技术实现的方案。豆瓣希望借助前端技术优秀的排版能力、开发速度、通用性，来弥补原生开发在这方面的不足。在微信作为主要内容分享渠道的今天，这样做还带来了一个额外的好处，Rexxar页面可以平滑的使用在微信中。

总结而言，如果Rexxar和PhoneGap/Cordova比较的话，大目标是一致的：使用前端技术开发移动应用。实现技术栈差不多：使用WebView，提供调用原生功能的接口。但是，出发点不一样。PhoneGap/Cordova致力于完全使用前端技术进行移动开发；Rexxar致力于在移动项目中部分使用前端技术。

#鼓励移动开发者学习前端技术

目前，我们移动团队大约有十多位客户端工程师，其中 iOS 和 Android 各一半。可以委派一位优秀的前端工程师专门支持App中的混合开发，他负责Rexxar Web的开发，提供基础设施。同时如果有一些较复杂的业务要用Rexxar实现，他也会参与和指导业务开发。

使用Rexxar这类混合开发技术，使得团队开发的技术栈向前端技术偏斜了。所以，较理想的配置是团队中加入较优秀的前端工程师，由他来处理基础设施的开发，和疑难问题的解决。同时，整个团队需要理解混合开发所带来的优势，认可这个开发方式的转变，并且愿意学习和调整自己的技术栈。

在项目中，在合适的场景中，可以优先使用Rexxar。在团队中，应该鼓励非前端工程师学习和使用前端技术。由于以前专门组织了关于前端技术内部培训，让有意愿的非前端工程师具有了可以使用前端技术进行日常开发的基本能力。期望在App的日常开发中，大部分Rexxar页面都可以由客户端工程师完成，前端工程师会帮忙做Code Review和解决疑难问题。😄

#总结与展望

通过在移动开发中使用Rexxar，在一定程度上提高了开发效率。以前一个页面需要 iOS 和 Android 两位工程师各开发一遍，现在只需要一位工程师写一次前端代码，甚至还可以应用到移动 Web 站上去。前端技术在开发界面方面也有效率上的优势，热部署能力，使他们规避了发布移动应用的审核过程，也让bug修复过程更便利。

豆瓣将Rexxar这个项目开源，一方面，是因为提高移动开发的工程效率是一个普遍问题，而他们的实践结果也证明Rexxar确实帮助改善了工程效率。所以，他们认为Rexxar应该能给大家提供一些借鉴的方向。另一方面，是为了提高项目本身的质量，没有方案是完美的，Rexxar也还存在不少问题。开源这个项目，促使他们提高了整个项目的代码质量。同时，也更容易听到大家的意见和建议。

虽然Rexxar仍然存在一些问题和使用上的限制。但是在有限的使用中，豆瓣App团队仍然收获不少。在未来他们会持续推动Rexxar在豆瓣移动开发中的使用。对于Rexxar未来的发展，他们主要关注两个方面：

* 一方面是基础设施，比如，如何在产品中，更好地监控Rexxar页面出现的问题，如何调试和解决Rexxar页面出现的bug。如果希望在大型项目中使用Rexxar，这些基础设施是应该配备的；

* 另一方面是性能，Rexxar仍然跑在浏览器引擎中。浏览器引擎这个中间层提高了工程效率，但也因为性能问题局限了其使用范围。所以，他们会花一些精力提高Rexxar的运行效率。比如，Rexxar的iOS版一直在关注从UIWebView迁移到WKWebView的可能性。



