# Android 原生第三方登录（微信，微博，QQ）
## 微信登录
### 集成步骤
* 在开放平台注册创建应用，申请登录权限
* 下载SDK，拷贝相关文件到项目工程目录
* 全局初始化微信组件
* 请求授权登录，获取code
* 通过code获取授权口令access_token
* 在第5步判断access_token是否存在和过期
* 如果access_token过期无效，就用refresh_token来刷新
* 使用access_token获取用户信息
* 注意事项

**1. 在开放平台注册创建应用，申请登录权限**
   
在[微信开放平台上](https://open.weixin.qq.com/)注册一个账号，然后创建移动应用。
![注册开发者账号](http://upload-images.jianshu.io/upload_images/2971226-3aa9ac41ddbf8ce9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
需要注意的是：应用签名的部分
![创建应用签名](http://upload-images.jianshu.io/upload_images/2971226-9b47fe10575f2099.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
此处应用签名建议使用线上的key的md5，关于这个需要注意的问题可以看另外一篇文章：[Android的签名总结](http://www.jianshu.com/p/1eb21e781482)   

**2. 下载SDK，拷贝相关文件到项目工程目录**  

* [开发示例Demo的下载：可以使用微信分享、登录、收藏、支付等功能](https://open.weixin.qq.com/zh_CN/htmledition/res/dev/download/sdk/WeChatSDK_sample_Android.zip)

Android Studio环境下：                                                       
在build.gradle文件中，添加如下依赖即可：

```
dependencies {
    compile 'com.tencent.mm.opensdk:wechat-sdk-android-with-mta:+'
}
```

或

```
dependencies {
    compile 'com.tencent.mm.opensdk:wechat-sdk-android-without-mta:+'
}
```

（其中，前者包含统计功能)

* 搭建开发环境

在Android Studio中新建你的工程，并保证网络设置可以成功从jcenter下载微信SDK即可。

**3. 全局初始化微信组件**

* 全局初始化微信组件，当然是Application的onCreate里（当然Activity的onCreate也是可以的，为了全局使用微信api对象方便操作）：

![始化微信](http://ou21vt4uz.bkt.clouddn.com/WXImage1.png)

**4. 请求授权登录，获取code，发送请求或响应到微信**

* 现在，你的程序要发送请求或发送响应到微信终端，可以通过IWXAPI的 sendReq 和 sendResp 两个方法来实现。

`boolean sendReq(BaseReq req);`

sendReq是第三方app主动发送消息给微信，发送完成之后会切回到第三方app界面。

`boolean sendResp(BaseResp resp);`

sendResp是微信向第三方app请求数据，第三方app回应数据之后会切回到微信界面。

sendReq的实现示例，如下图所示：

![发送请求或响应到微信](http://ou21vt4uz.bkt.clouddn.com/WXImage2.png)

* 接收微信的请求及返回值

如果你的程序需要接收微信发送的请求，或者接收发送到微信请求的响应结果，需要下面3步操作：

> a. 在你的包名相应目录下新建一个wxapi目录，并在该wxapi目录下新增一个WXEntryActivity类，该类继承自Activity

![](http://ou21vt4uz.bkt.clouddn.com/WXImage3.png)

并在manifest文件里面加上exported属性，设置为true，例如：

![](http://ou21vt4uz.bkt.clouddn.com/WXImage4.png)

> b. 实现IWXAPIEventHandler接口，微信发送的请求将回调到onReq方法，发送到微信请求的响应结果将回调到onResp方法.   
> 在WXEntryActivity中将接收到的intent及实现了IWXAPIEventHandler接口的对象传递给IWXAPI接口的handleIntent方法，示例如下图：

![](http://ou21vt4uz.bkt.clouddn.com/WXImage5.png)

当微信发送请求到你的应用，将通过IWXAPIEventHandler接口的onReq方法进行回调，类似的，应用请求微信的响应结果将通过onResp回调。

**5. 通过code获取授权口令access_token**

>小伙伴有疑问code是啥玩意：   
第三方通过code进行获取access_token的时候需要用到，code的超时时间为10分钟，一个code只能成功换取一次access_token即失效。code的临时性和一次保障了微信授权登录的安全性。第三方可通过使用https和state参数，进一步加强自身授权登录的安全性。

在onResp的回调方法中获取了code，然后通过code获取授权口令access_token

![](http://ou21vt4uz.bkt.clouddn.com/WXImage6.png)

**6. 在第5步判断access_token是否存在和过期**

在回调的onResp方法中获取code后，处理access_token是否登录过或者过期的问题：

![](http://ou21vt4uz.bkt.clouddn.com/WXImage7.png)

判断授权口令是否有效：

![](http://ou21vt4uz.bkt.clouddn.com/WXImage8.png)

**7. 如果access_token过期无效，就用refresh_token来刷新**

![](http://ou21vt4uz.bkt.clouddn.com/WXImage9.png)

**8. 使用access_token获取用户信息**

![](http://ou21vt4uz.bkt.clouddn.com/WXImage10.png)

**9. 注意事项**  
[1]如果需要混淆代码，为了保证sdk的正常使用，需要在proguard.cfg加上下面两行配置： 

```
-keep class com.tencent.mm.opensdk.** {
   *;
}
```
```
-keep class com.tencent.wxop.** {
   *;
}
```
```
-keep class com.tencent.mm.sdk.** {
   *;
}
```
[2]如果需要运行SDK Sample工程，需要通过指定的debug.keystore来进行签名：
Android Studio环境下：

```
signingConfigs {
    debug {
        storeFile file("../debug.keystore")
    }
}
```
以上就是所有的集成步骤了，获取用户信息之后，就可以调用相应的用户信息处理相应的业务需求。


## 微博登录
### 集成步骤
* 在开放平台注册成为开发者，创建移动应用
* 查看创建应用的APPKEY及APPSECRET，用来调用微博开放平台各API的身份标志
* 填写应用回调页，使OAuth2.0授权正常进行
* 在“我的应用 - 应用信息”填写应用的平台信息
* 下载SDK，拷贝相关文件到项目工程目录
* 替换成自己应用的 APP_KEY 等参数
* 创建微博API接口类对象
* 实现WbAuthListener接口
* 调用方法，认证授权
* 使用access_token获取用户信息
* 注意事项

**1.在开放平台注册成为开发者，创建移动应用**

![](http://www.sinaimg.cn/blog/developer/wiki/sdk72203.png)

如果你还不是一名开发者，请先注册成为开发者，[具体参考新手指南]( http://open.weibo.com/wiki/%E6%96%B0%E6%89%8B%E6%8C%87%E5%8D%97)

创建应用时，开发者需要谨慎选择应用对应平台，不同的平台建议使用不同APPKEY开发。

 ![](http://www.sinaimg.cn/blog/developer/wiki/sdk72201.png)
 
 **2.查看创建应用的APPKEY及APPSECRET，用来调用微博开放平台各API的身份标志**
 
 ![](http://www.sinaimg.cn/blog/developer/wiki/khd411.png)
 
 注：通常Mobile Native App没有服务器回调地址，您可以在应用控制台授权回调页处填写平台提供的默认回调页，该页面用户不可见，仅用于获取access token。 [OAuth2.0客户端默认回调页](https://api.weibo.com/oauth2/default.html)
 
 **3.填写应用回调页，使OAuth2.0授权正常进行**
 
 如果APPSECRET发生泄露，可以通过该页面中的重置按钮对其重置，如下图所示：
 
 ![](http://www.sinaimg.cn/blog/developer/wiki/khd5.jpg)
 
 **4.在“我的应用 - 应用信息”填写应用的平台信息**
 
 ![](http://www.sinaimg.cn/blog/developer/wiki/sdk72204.png)
 
 这里iPhone应用填写Apple ID和Buddle ID，Android应用填写包名，签名及下载地址。 
关于各字段含义在控制台中均有说明.

**5.下载SDK，拷贝相关文件到项目工程目录**

全新的SDK已经上传到中央仓库这里按照Android studio为例子，在项目根目录的build.gradle中设置中央仓库

```
maven { url "https://dl.bintray.com/thelasterstar/maven/" }
```

![](http://ou21vt4uz.bkt.clouddn.com/WB1.png)

在需要引入SDK的module目录的build.gradle中引入sdk-core依赖

```
dependencies {compile 'com.sina.weibo.sdk:core:2.0.6:openDefaultRelease@aar' }
```

![](http://ou21vt4uz.bkt.clouddn.com/WB2.png)

点击同步按钮，等待SDK库下载完成。 也可以直接在github上下载微博 SDK demo，参考demo接入方案。

**6.替换成自己应用的 APP_KEY 等参数**

鉴于目前有很多第三方开发直接拷贝并使用Demo中的Constants类，因此，有必要说明，第三方开发者需要将Constants类中的各种参数替换成自己应用的参数，请仔细阅读代码注释。

```
public interface Constants {
    /** 当前 DEMO 应用的 APP_KEY，第三方应用应该使用自己的 APP_KEY 替换该 APP_KEY */
    public static final String APP_KEY      = "2045436852";

    /** 
     * 当前 DEMO 应用的回调页，第三方应用可以使用自己的回调页。
     * 建议使用默认回调页：https://api.weibo.com/oauth2/default.html
     */
    public static final String REDIRECT_URL = "http://www.sina.com";

    /**
     * WeiboSDKDemo 应用对应的权限，第三方开发者一般不需要这么多，可直接设置成空即可。
     * 详情请查看 Demo 中对应的注释。
     */
    public static final String SCOPE = 
            "email,direct_messages_read,direct_messages_write,"
            + "friendships_groups_read,friendships_groups_write,statuses_to_me_read,"
            + "follow_app_official_microblog," + "invitation_write";
}
```
**7.创建微博API接口类对象**

* 首先初始化WbSdk对象(在你的应用的Application或者调用Sdk功能代码前)

```WbSdk.install(this,new AuthInfo(this, Constants.APP_KEY, Constants.REDIRECT_URL, Constants.SCOPE));
```AuthInfo维护了授权需要的基本信息，APP_KEY(开发平台生成的唯一key)、Redirect URI (授权回调)、SCOPE(需要请求的权限功能，默认参考demo中数据)。
* 初始化SsoHandler对象

```mSsoHandler = new SsoHandler(WBAuthActivity.this);
```
SsoHandler是发起授权的核心类。

**8.实现WbAuthListener接口**

```
private class SelfWbAuthListener implements com.sina.weibo.sdk.auth.WbAuthListener{
        @Override
        public void onSuccess(final Oauth2AccessToken token) {
            WBAuthActivity.this.runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    mAccessToken = token;
                    if (mAccessToken.isSessionValid()) {
                        // 显示 Token
                        updateTokenView(false);
                        // 保存 Token 到 SharedPreferences
                        AccessTokenKeeper.writeAccessToken(WBAuthActivity.this, mAccessToken);
                        Toast.makeText(WBAuthActivity.this,
                                R.string.weibosdk_demo_toast_auth_success, Toast.LENGTH_SHORT).show();
                    }
                }
            });
        }

        @Override
        public void cancel() {
            Toast.makeText(WBAuthActivity.this,
                    R.string.weibosdk_demo_toast_auth_canceled, Toast.LENGTH_LONG).show();
        }

        @Override
        public void onFailure(WbConnectErrorMessage errorMessage) {
            Toast.makeText(WBAuthActivity.this, errorMessage.getErrorMessage(), Toast.LENGTH_LONG).show();
        }
    }
```

**9.调用方法，认证授权**

* 1 Web 授权，仅web，直接调用以下函数：

```
mSsoHandler = new SsoHandler(WBAuthActivity.this);
mSsoHandler.authorizeWeb(new WbAuthListener());
```

* 2 SSO授权，仅客户端，需要调用以下函数：

```
mSsoHandler = new SsoHandler(WBAuthActivity.this);
mSsoHandler. authorizeClientSso(new WbAuthListener());
```

* 3 all In one方式授权，如果手机安转了微博客户端则使用客户端授权，没有则进行网页授权，需要调用以下函数：

```
mSsoHandler = new SsoHandler(WBAuthActivity.this);
mSsoHandler. authorize(new WbAuthListener());
```

* 要接收到授权的相关数据，必须在当前页面Activity或者Fragment的 onActivityResult方法中添加SSOHandler的调用，如下所示

```
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (mSsoHandler != null) {
        mSsoHandler.authorizeCallBack(requestCode, resultCode, data);
    }
}
```
**10.使用access_token获取用户信息**

要请求微博接口，根据用户ID获取用户信息

![](http://ou21vt4uz.bkt.clouddn.com/WB3.png)

退出登录

![](http://ou21vt4uz.bkt.clouddn.com/WB4.png)

**11.注意事项**

如果需要混淆代码，为了保证sdk的正常使用，需要在proguard.cfg加上下面两行配置： 

```
-keep class com.sina.weibo.sdk.** { *; }

```
以上就是所有的集成步骤了，获取用户信息之后，就可以调用相应的用户信息处理相应的业务需求。

## QQ登录
### 集成步骤
* 在开放平台注册成为开发者，创建移动应用
* 下载SDK，拷贝相关文件到项目工程目录，配置工程
* 全局初始化QQ组件
* 实现IUiListener接口
* 调用方法，认证授权
* 使用access_token获取用户信息

**1.在开放平台注册成为开发者，创建移动应用**

[腾讯开放平台](http://open.qq.com/)获取APP ID和APP KEY（未注册腾讯开发者账号的可能需要先注册账号），获取的过程还是还是非常容易的（不用填写任何的应用程序信息）。

![](http://img.blog.csdn.net/20160807111239942?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

**2.下载SDK，拷贝相关文件到项目工程目录，配置工程**

* 新建工程并导入SDK的jar文件

![](http://ou21vt4uz.bkt.clouddn.com/QQ1.png)

* 配置AndroidManifest
在应用的AndroidManifest.xml增加配置的<application>节点下增加以下配置（注：不配置将会导致无法调用API）；

![](http://ou21vt4uz.bkt.clouddn.com/QQ2.png)

通过以上两个步骤，工程就已经配置完成了。接下来就可以在代码里使用QQ互联的SDK进行开发了。

**3.全局初始化QQ组件**

![](http://ou21vt4uz.bkt.clouddn.com/QQ3.png)

**4.实现IUiListener接口**

![](http://ou21vt4uz.bkt.clouddn.com/QQ4.png)

**5.调用方法，认证授权**

![](http://ou21vt4uz.bkt.clouddn.com/QQ5.png)

**6.使用access_token获取用户信息**

![](http://ou21vt4uz.bkt.clouddn.com/QQ6.png)

![](http://ou21vt4uz.bkt.clouddn.com/QQ7.png)

![](http://ou21vt4uz.bkt.clouddn.com/QQ8.png)

以上就是所有的集成步骤了，获取用户信息之后，就可以调用相应的用户信息处理相应的业务需求。

## QQ网页登录

### 现象
QQ登录SDK在用户设备没有安装手机QQ客户端的情况下，默认是会调起网页授权的，但是可能是因为腾讯的某些限制，新申请的app_id都无法使用网页授权，打开后有些是跳转到下载手Q页面，有些是一直显示“正在打开授权登录页…”

### 分析
试了下市场上其他app使用QQ登录的情况，发现京东的客户端，在未安装客户端的情况下是可以打开网页授权的，抓包得到京东的授权链接：

[http://openmobile.qq.com/oauth2.0/m_authorize?status_os=5.0.2&client_id=100273020&status_userip=fec0%3A%3A10%3A5530%3Aabe3%3Abbb9%3Ac98e%2516&format=json&switch=1&status_version=21&appid_for_getting_config=100273020&status_machine=x600&pf=openmobile_android&sdkp=a&sdkv=2.4.lite&sign=88d1d495ffc03ae69a9339367172a5bf&time=1467062773&scope=all&redirect_uri=auth%3A%2F%2Ftauth.qq.com%2F&display=mobile&response_type=token&cancel_display=1](http://openmobile.qq.com/oauth2.0/m_authorize?status_os=5.0.2&client_id=100273020&status_userip=fec0%3A%3A10%3A5530%3Aabe3%3Abbb9%3Ac98e%2516&format=json&switch=1&status_version=21&appid_for_getting_config=100273020&status_machine=x600&pf=openmobile_android&sdkp=a&sdkv=2.4.lite&sign=88d1d495ffc03ae69a9339367172a5bf&time=1467062773&scope=all&redirect_uri=auth%3A%2F%2Ftauth.qq.com%2F&display=mobile&response_type=token&cancel_display=1)


同时在网上找到一个介绍说不用SDK直接打开网页授权的链接：
[https://openmobile.qq.com/oauth2.0/m_authorize?status_userip=&scope=add_share,add_topic,list_album,upload_pic,get_simple_userinfo&redirect_uri=auth%3A%2F%2Ftauth.qq.com%2F&response_type=token&client_id=100353810](https://openmobile.qq.com/oauth2.0/m_authorize?status_userip=&scope=add_share,add_topic,list_album,upload_pic,get_simple_userinfo&redirect_uri=auth%3A%2F%2Ftauth.qq.com%2F&response_type=token&client_id=100353810)

这个链接也是能打开授权页的，但是参数比较少。

那么问题来了，腾讯到底是怎么限制是否能打开授权登录页的呢？ 
尝试使用我们自己的app_id替换链接中的client_id，发现第二个链接竟然也能打开，那么也就是说应该不是根据app_id来做的限制，那就可能是其他参数了。因此人肉一个个的测试，最终发现是根据sdkv这个参数来做限制的。

那么这个值该改成多少呢？

查看了QQ互联SDK的更新记录：

```
V1.6 更新了如下内容： 
1.登录方式支持手机QQ登录。 
2.支持通过手机QQ分享信息。 
3.PCPUSH增加手机QQ渠道触达。
```
V1.6才支持手Q登录，那么之前的sdk应该都是网页授权咯？测试下，最终发现版本号1.4及以下的版本都能打开授权登录页。

查看资料发现可以用QQ登录OAuth2.0的方式进行QQ网页登录,[OAuth2.0简介](http://wiki.connect.qq.com/oauth2-0%E7%AE%80%E4%BB%8B).

**开发攻略_Client-side**

* 准备工作
* 获取Access Token
* 使用Access Token来获取用户的OpenID
* 使用Access Token以及OpenID来访问和修改用户数据

**准备工作**
 请确保您的网站已经提交接入QQ登录的申请，并成功获取到appid和appkey。[申请接入](http://connect.opensns.qq.com/apply)
 
**获取Access Token**
请求地址：
移动端应用： https://openmobile.qq.com/oauth2.0/m_authorize

请求方法：

GET

请求参数请包含如下内容：
![](http://ou21vt4uz.bkt.clouddn.com/QQ_Auth_Login_Get_AccessToken.png)

返回说明：

1. 如果用户成功登录并授权，则会跳转到指定的回调地址，并在URL后加“#”号，带上Access Token以及expires_in等参数。如果请求参数中传入了state，这里会带上原始的state值。如果redirect_uri地址后已经有“#”号，则加“&”号，带上相应的返回参数。如：
PC网站：http://graph.qq.com/demo/index.jsp?#access_token=FE04************************CCE2&expires_in=7776000&state=test    
WAP网站：http://open.z.qq.com/demo/index.jsp?#access_token=FE04************************CCE2&expires_in=7776000&state=test   
说明：expires_in是该access token的有效期，单位为秒。

Tips：

```
1. 可通过js方法：window.location.hash来获取URL中#后的参数值。
2. 建议用js设置cookie存储token。 
3. 如果用户在登录授权过程中取消登录流程，对于PC网站，登录页面直接关闭；对于WAP网站，同样跳转回指定的回调地址，并在redirect_uri地址后带上usercancel参数和原始的state值，其中usercancel值为非零，如：
http://open.z.qq.com/demo/index.jsp?#usercancel=1&state=test 
```

错误码说明：

接口调用有错误时，会返回code和msg字段，以url参数对的形式返回，value部分会进行url编码（UTF-8）。

以上是官方文档中OAuth2.0的方式进行QQ网页登录的内容，但是客户端实现不是完全一样的，大概思路是一致的。客户端是通过Webview中loadUrl方法相应的获取Access Token的url获取Access Token以及expires_in等参数，再去请求用户信息就能实现网页登录了。

**使用Access Token来获取用户的OpenID**

**1.设置redirect_uri**
成功授权后的回调地址，必须是注册appid时填写的主域名下的地址，建议设置为网站首页或网站的用户中心。注意需要将url进行URLEncode。
`private String mQQRedirectUri = "auth%3A%2F%2Ftauth.qq.com%2F";`

**2.拼接URL的访问参数**
移动端应用： https://openmobile.qq.com/oauth2.0/m_authorize

```
private String mQQWebAuthUrl = "https://openmobile.qq.com/oauth2.0/m_authorize?display=mobile&status=&scope=get_user_info,list_album,upload_pic,do_like&redirect_uri=" + mQQRedirectUri + "&response_type=token&client_id=" + mQQAppId;
```

**3.截取回调地址，获取Access Token以及expires_in等参数**
webview设置如下：
![](http://ou21vt4uz.bkt.clouddn.com/QQ_Auth_webview.png)
截取回调地址，获取参数:
![](http://ou21vt4uz.bkt.clouddn.com/QQ_Auth_webview_2.png)

**使用Access Token以及OpenID来访问和修改用户数据**
发送请求到get_user_info的URL（请将access_token，appid等参数值替换为你自己的）：

```
https://graph.qq.com/user/get_user_info?access_token=YOUR_ACCESS_TOKEN&oauth_consumer_key=YOUR_APP_ID&openid=YOUR_OPENID
```

（2）成功返回后，即可获取到用户数据：

```
{
  "ret":0,
   "msg":"",
   "nickname":"YOUR_NICK_NAME",
   ...
}
```
以上就是QQ网页登录所有的集成步骤了，获取用户信息之后，就可以调用相应的用户信息处理相应的业务需求。


