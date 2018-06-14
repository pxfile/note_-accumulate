# Android原生三方分享
## 微信分享
根据官方API的[Android接入指南](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=1417751808&token=&lang=zh_CN)接入微信SDK，导入相应的jar包，下载官方[demo](https://open.weixin.qq.com/zh_CN/htmledition/res/dev/download/sdk/WeChatSDK_sample_Android.zip)参考

* 微信分享是指第三方App通过接入该功能，让用户可以从App分享文字、图片、音乐、视频、网页至微信好友会话、朋友圈或添加到微信收藏。

* 微信分享及收藏功能已向全体开发者开放，开发者在微信开放平台帐号下申请App并通过审核后，即可获得微信分享及收藏权限。

* 微信分享及收藏目前支持文字、图片、音乐、视频、网页共五种类型。开发者在App中在集成微信SDK后，可调用接口实现，以下依次是文字分享、图片分享、音乐分享、视频分享、网站分享的示例。

**新增能力：移动应用支持小程序类型分享**

* 移动应用分享功能支持小程序类型分享，要求发起分享的App与小程序属于同一微信开放平台帐号。支持分享小程序类型消息至好友会话，不支持“分享至朋友圈”及“收藏”。

* 微信客户端版本要求：6.5.6及以上微信客户端版本。为兼容旧版本客户端，若客户端版本低于6.5.6，小程序类型分享将自动转成网页类型分享。

**分享或收藏的目标场景，通过修改scene场景值实现。**

* 发送到聊天界面——WXSceneSession
* 发送到朋友圈——WXSceneTimeline
* 添加到微信收藏——WXSceneFavorite

**文字类型分享**

```

    /**
      * 分享文字
      */
    private void shareText(int targetScene, ShareBean shareBean) {
        ShareItem shareItem = getShareItem(targetScene, shareBean);
        WXTextObject textObj = new WXTextObject();
        textObj.text = shareItem.content;

        // 用WXTextObject对象初始化一个WXMediaMessage对象
        WXMediaMessage msg = new WXMediaMessage();
        msg.mediaObject = textObj;
        // 发送文本类型的消息时，title字段不起作用
        // msg.title = "Will be ignored";
        msg.title = shareItem.title;
        msg.description = shareItem.content;

        // 构造一个Req
        SendMessageToWX.Req req = new SendMessageToWX.Req();
        req.transaction = buildTransaction("text"); // transaction字段用于唯一标识一个请求
        req.message = msg;

        // req.scene = SendMessageToWX.Req.WXSceneTimeline;// 表示发送场景为朋友圈，这个代表分享到朋友圈
        req.scene = targetScene;//表示发送场景为好友对话，这个代表分享给好友
        // 调用api接口发送数据到微信
        InitPlatformManager.mWXApi.sendReq(req);
    }
```

**图片类型分享**

```
/**
     * 分享在线图片
     */
    private void shareImage(int targetScene) {
        if (null == mThumbBmp) {
            return;
        }
        WXImageObject imgObj = new WXImageObject(mThumbBmp);

        WXMediaMessage msg = new WXMediaMessage();
        msg.mediaObject = imgObj;

        Bitmap thumbBmp = Bitmap.createScaledBitmap(mThumbBmp, THUMB_SIZE, THUMB_SIZE, true);
        mThumbBmp.recycle();
        msg.thumbData = Utils.bmpToByteArray(thumbBmp, true);

        SendMessageToWX.Req req = new SendMessageToWX.Req();
        req.transaction = buildTransaction("img");
        req.message = msg;
        req.scene = targetScene;
        InitPlatformManager.mWXApi.sendReq(req);
    }

```
**SD卡图片类型分享**

```
/**
     * 分享sdcard图片
     */
    private void shareLocalImage(int targetScene, ShareBean shareBean) {
        //TODO 判断图片是否存在
        WXImageObject imageObject = new WXImageObject();
        if (TextUtils.isEmpty(shareBean.image)) {
            return;
        }
        imageObject.setImagePath(shareBean.image);

        WXMediaMessage msg = new WXMediaMessage();
        msg.mediaObject = imageObject;

        Bitmap bmp = BitmapFactory.decodeFile(shareBean.image);
        if (null == bmp) {
            return;
        }
        Bitmap thumb = Bitmap.createScaledBitmap(bmp, THUMB_SIZE, THUMB_SIZE, true);
        bmp.recycle();
        msg.thumbData = Utils.bmpToByteArray(thumb, true);

        SendMessageToWX.Req req = new SendMessageToWX.Req();
        req.transaction = buildTransaction("img");
        req.message = msg;
        req.scene = targetScene;
        InitPlatformManager.mWXApi.sendReq(req);
    }

```
**音乐类型分享**

```
/**
     * 分享音乐url
     */
    private void shareMusic(int targetScene, ShareBean shareBean) {
        ShareItem shareItem = getShareItem(targetScene, shareBean);
        WXMusicObject music = new WXMusicObject();
        music.musicUrl = shareBean.url;

        WXMediaMessage msg = new WXMediaMessage();
        msg.mediaObject = music;
        msg.title = shareItem.title;
        msg.description = shareItem.content;

        Bitmap thumbBmp = Bitmap.createScaledBitmap(mThumbBmp, THUMB_SIZE, THUMB_SIZE, true);
        mThumbBmp.recycle();
        msg.thumbData = Utils.bmpToByteArray(thumbBmp, true);

        SendMessageToWX.Req req = new SendMessageToWX.Req();
        req.transaction = buildTransaction("music");
        req.message = msg;
        req.scene = targetScene;
        InitPlatformManager.mWXApi.sendReq(req);
    }
```
**低带宽音乐类型分享**

```
/**
     * 分享低带宽音乐url
     */
    private void shareLowBandwidthMusic(int targetScene, ShareBean shareBean) {
        ShareItem shareItem = getShareItem(targetScene, shareBean);
        WXMusicObject music = new WXMusicObject();
        music.musicLowBandUrl = shareBean.url;

        WXMediaMessage msg = new WXMediaMessage();
        msg.mediaObject = music;
        msg.title = shareItem.title;
        msg.description = shareItem.content;

        Bitmap thumbBmp = Bitmap.createScaledBitmap(mThumbBmp, THUMB_SIZE, THUMB_SIZE, true);
        mThumbBmp.recycle();
        msg.thumbData = Utils.bmpToByteArray(thumbBmp, true);

        SendMessageToWX.Req req = new SendMessageToWX.Req();
        req.transaction = buildTransaction("music");
        req.message = msg;
        req.scene = targetScene;
        InitPlatformManager.mWXApi.sendReq(req);
    }
```
**视频类型分享**

```
 /**
     * 分享视频url
     */
    private void shareVideo(int targetScene, ShareBean shareBean) {
        ShareItem shareItem = getShareItem(targetScene, shareBean);
        WXVideoObject video = new WXVideoObject();
        video.videoUrl = shareBean.url;

        WXMediaMessage msg = new WXMediaMessage(video);
        msg.title = shareItem.title;
        msg.description = shareItem.content;
        Bitmap thumbBmp = Bitmap.createScaledBitmap(mThumbBmp, THUMB_SIZE, THUMB_SIZE, true);
        mThumbBmp.recycle();
        msg.thumbData = Utils.bmpToByteArray(thumbBmp, true);

        SendMessageToWX.Req req = new SendMessageToWX.Req();
        req.transaction = buildTransaction("video");
        req.message = msg;
        req.scene = targetScene;
        InitPlatformManager.mWXApi.sendReq(req);
    }
```

**低带宽视频类型分享**

```
/**
     * 分享低带宽视频url
     */
    private void shareLowBandwidthVideo(int targetScene, ShareBean shareBean) {
        ShareItem shareItem = getShareItem(targetScene, shareBean);
        WXVideoObject video = new WXVideoObject();
        video.videoUrl = shareBean.url;

        WXMediaMessage msg = new WXMediaMessage(video);
        msg.title = shareItem.title;
        msg.description = shareItem.content;

        SendMessageToWX.Req req = new SendMessageToWX.Req();
        req.transaction = buildTransaction("video");
        req.message = msg;
        req.scene = targetScene;
        InitPlatformManager.mWXApi.sendReq(req);
    }
```

**网页类型分享**

```
/**
     * 分享网页
     */
    private void shareWebpage(int targetScene, ShareBean shareBean) {
        ShareItem shareItem = getShareItem(targetScene, shareBean);
        WXWebpageObject webpage = new WXWebpageObject();
        webpage.webpageUrl = shareBean.url;

        WXMediaMessage msg = new WXMediaMessage(webpage);
        msg.title = shareItem.title;
        msg.description = shareItem.content;

        Bitmap thumbBmp = Bitmap.createScaledBitmap(mThumbBmp, THUMB_SIZE, THUMB_SIZE, true);
        mThumbBmp.recycle();
        msg.thumbData = Utils.bmpToByteArray(thumbBmp, true);

        SendMessageToWX.Req req = new SendMessageToWX.Req();
        req.transaction = buildTransaction("webpage");
        req.message = msg;
        req.scene = targetScene;
        InitPlatformManager.mWXApi.sendReq(req);
    }
```

在onResp方法中进行分享的回调监听

```
// 第三方应用发送到微信的请求处理后的响应结果，会回调到该方法
    @Override
    public void onResp(BaseResp resp) {
        switch (resp.getType()) {
            case ConstantsAPI.COMMAND_SENDAUTH:
                //登录
                switch (resp.errCode) {
                    case BaseResp.ErrCode.ERR_OK:
                        //处理登录成功...
                        break;
                    case BaseResp.ErrCode.ERR_USER_CANCEL:
                        //处理认证取消...
                        break;
                    case BaseResp.ErrCode.ERR_AUTH_DENIED:
                        //处理认证被拒绝...
                        break;
                    default:
                        //认证失败                               break;                }
                break;
            case ConstantsAPI.COMMAND_SENDMESSAGE_TO_WX:
                //分享
                switch (resp.errCode) {
                    case BaseResp.ErrCode.ERR_OK:
                        //处理分享成功...
                        break;
                    case BaseResp.ErrCode.ERR_USER_CANCEL:
                        //处理分享取消...
                        break;
                    case BaseResp.ErrCode.ERR_AUTH_DENIED:
                        //处理分享失败...
                        break;
                    default:
                        break;
                }
                break;
            default:
                break;
        }
    }
```

## 微博分享
我们到[官方文档](https://github.com/sinaweibosdk/weibo_android_sdk)下载相应的官方demo并且多看看项目的ReadeMe，写的很详细让你少走弯路。
集成完微博sdk后调用API就可以分享了，如下代码：

**注：微博图片传的是Bitmap类型的**，所以我们得通过url自己下载图片才能分享

**第三方应用发送请求消息到微博，唤起微博分享界面。**

```
/**
     * 第三方应用发送请求消息到微博，唤起微博分享界面。
     */
    private void sendMultiMessage(ShareBean shareBean) {
        WeiboMultiMessage weiBoMessage = new WeiboMultiMessage();
        if (null != shareBean.weibo && !TextUtils.isEmpty(shareBean.weibo.title) && !TextUtils.isEmpty(shareBean.weibo.content)) {
            //分享文字
            weiBoMessage.textObject = getTextObj(shareBean);
        }
        if (null != mThumbBmp) {
            //分享图片
            weiBoMessage.imageObject = getImageObj();
        }
        weiBoMessage.mediaObject = getWebpageObj(shareBean);
        mShareHandler.shareMessage(weiBoMessage, false);
    }
```

**分享文字，创建文本消息对象。**

```
/**
     * 创建文本消息对象。
     */
    private TextObject getTextObj(ShareBean shareBean) {
        TextObject textObject = new TextObject();
        textObject.text = shareBean.weibo.content;
        textObject.title = shareBean.weibo.title;
        textObject.actionUrl = shareBean.url;
        return textObject;
    }
```

**分享图片，创建图片消息对象**

```
/**
     * 创建图片消息对象。
     */
    private ImageObject getImageObj() {
        ImageObject imageObject = new ImageObject();
        Bitmap bitmap = mThumbBmp;
        imageObject.setImageObject(bitmap);
        return imageObject;
    }
```

**创建多媒体（网页）消息对象**

```
/**
     * 创建多媒体（网页）消息对象。
     */
    private WebpageObject getWebpageObj(ShareBean shareBean) {
        WebpageObject mediaObject = new WebpageObject();
        mediaObject.identify = Utility.generateGUID();
        mediaObject.title = shareBean.weibo.title;
        mediaObject.description = shareBean.weibo.content;
        Bitmap bitmap = mThumbBmp;
        // 设置 Bitmap 类型的图片到视频对象里  设置缩略图。 注意：最终压缩过的缩略图大小不得超过 32kb。
        mediaObject.setThumbImage(bitmap);
        mediaObject.actionUrl = shareBean.url;
        mediaObject.defaultText = shareBean.weibo.content;
        return mediaObject;
    }
```

**微博分享回调**

```
 @Override
    protected void onNewIntent(Intent intent){
        super.onNewIntent(intent);
        //微博享回调
        if (mShareHandler != null) {
            mShareHandler.doResultIntent(intent, ShareWeiBoUtil.this);
        }
    }
```

**处理相应的回调**

```
@Override
    public void onWbShareSuccess() {
        //分享成功
}

    @Override
    public void onWbShareCancel() {
        //分享取消
  }

    @Override
    public void onWbShareFail() {
        //分享失败
    }
```

## QQ，QQ空间分享
根据官方文档，开发者注册、应用创建，开发环境配置集成SDK，进行内容分享

**新建Tencent实例**

```
private Tencent mTencent;// 新建Tencent实例用于调用分享方法
@Override
protected void onCreate(Bundle savedInstanceState) {
　　super.onCreate(savedInstanceState);
　　setContentView(R.layout.activity_main);
　　mTencent = Tencent.createInstance("your APP ID",getApplicationContext()); 
}
```

**分享回调接口的实现**

自定义分享回调接口：

```
class MyIUiListener implements IUiListener {
　　@Override
　　public void onComplete(Object o) {
　　　　// 操作成功 
　　} 
　　@Override
　　public void onError(UiError uiError) { 
　　　　// 分享异常
　　} 
　　@Override
　　public void onCancel() {
　　　　// 取消分享
　　}
}
```

重写Activity或者Fragment的onActivityResult方法，否则不能正常的监听分享状态，具体代码如下：

```
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
　　// TODO Auto-generated method stub
　　super.onActivityResult(requestCode, resultCode, data);
　　Tencent.onActivityResultData(requestCode, resultCode, data, mIUiListener);
　　if (requestCode == Constants.REQUEST_API) {
　　　　if (resultCode == Constants.REQUEST_QQ_SHARE || resultCode == Constants.REQUEST_QZONE_SHARE || resultCode == Constants.REQUEST_OLD_SHARE) {
　　　　　　Tencent.handleResultData(data, mIUiListener);
　　　　}
　　}
}
```
**分享消息到QQ**

```
private void shareByQQ(Activity activity, int shareType, ShareBean shareBean) {
        final Bundle params = new Bundle();
        switch (shareType) {
            case QQShare.SHARE_TO_QQ_TYPE_DEFAULT:
                //默认分享-图文并存
                params.putInt(QQShare.SHARE_TO_QQ_KEY_TYPE, QQShare.SHARE_TO_QQ_TYPE_DEFAULT);
                params.putString(QQShare.SHARE_TO_QQ_TITLE, shareBean.qq.title);// 分享的标题, 最长30个字符
                params.putString(QQShare.SHARE_TO_QQ_SUMMARY, shareBean.qq.content);// 分享的消息摘要，最长40个字
                params.putString(QQShare.SHARE_TO_QQ_TARGET_URL, shareBean.url);// 这条分享消息被好友点击后的跳转URL。
                params.putString(QQShare.SHARE_TO_QQ_IMAGE_URL, shareBean.image);// 分享图片的URL或者本地路径
                break;
            case QQShare.SHARE_TO_QQ_TYPE_IMAGE:
                //分享纯图片
                params.putInt(QQShare.SHARE_TO_QQ_KEY_TYPE, QQShare.SHARE_TO_QQ_TYPE_IMAGE);// 设置分享类型为纯图片分享
                params.putString(QQShare.SHARE_TO_QQ_IMAGE_LOCAL_URL, shareBean.image);// 需要分享的本地图片URL
                break;
            case QQShare.SHARE_TO_QQ_TYPE_AUDIO:
                //分享音频
                params.putInt(QQShare.SHARE_TO_QQ_KEY_TYPE, QQShare.SHARE_TO_QQ_TYPE_AUDIO);
                params.putString(QQShare.SHARE_TO_QQ_TITLE, shareBean.qq.title);
                params.putString(QQShare.SHARE_TO_QQ_SUMMARY, shareBean.qq.content);
                params.putString(QQShare.SHARE_TO_QQ_TARGET_URL, shareBean.url);
                params.putString(QQShare.SHARE_TO_QQ_IMAGE_URL, shareBean.image);
                params.putString(QQShare.SHARE_TO_QQ_AUDIO_URL, shareBean.url);
                break;
            case QQShare.SHARE_TO_QQ_TYPE_APP:
                //分享应用
                params.putInt(QQShare.SHARE_TO_QQ_KEY_TYPE, QQShare.SHARE_TO_QQ_TYPE_APP);
                params.putString(QQShare.SHARE_TO_QQ_TITLE, shareBean.qq.title);
                params.putString(QQShare.SHARE_TO_QQ_SUMMARY, shareBean.qq.content);
                params.putString(QQShare.SHARE_TO_QQ_IMAGE_URL, shareBean.image);
                break;
            default:
                break;
        }
        /**
         * 分享额外选项，两种类型可选（默认是不隐藏分享到QZone按钮且不自动打开分享到QZone的对话框）：
         QQShare.SHARE_TO_QQ_FLAG_QZONE_AUTO_OPEN，分享时自动打开分享到QZone的对话框。
         QQShare.SHARE_TO_QQ_FLAG_QZONE_ITEM_HIDE，分享时隐藏分享到QZone按钮
         */
        params.putInt(QQShare.SHARE_TO_QQ_EXT_INT, QQShare.SHARE_TO_QQ_FLAG_QZONE_ITEM_HIDE);//其它附加功能
        params.putString(QQShare.SHARE_TO_QQ_APP_NAME, activity.getString(R.string.app_name));// 手Q客户端顶部，替换“返回”按钮文字，如果为空，用返回代替
        //开始分享
        doShareToQQ(activity, params);
    }

    private void doShareToQQ(final Activity activity, final Bundle params) {
        // QQ分享要在主线程做
        ThreadManager.getMainHandler().post(new Runnable() {

            @Override
            public void run() {
                if (null != InitPlatformManager.mTencent) {
                    InitPlatformManager.mTencent.shareToQQ(activity, params, qqShareListener);
                }
            }
        });
    }
```

**QQ空间同理**


**分享消息到QQ空间**


```
private void shareByQzone(Activity activity, int shareType, ShareBean shareBean) {
        final Bundle params = new Bundle();
        switch (shareType) {
            case QzoneShare.SHARE_TO_QZONE_TYPE_IMAGE_TEXT:
                //默认分享-图文并存
                params.putInt(QzoneShare.SHARE_TO_QZONE_KEY_TYPE, QzoneShare.SHARE_TO_QZONE_TYPE_IMAGE_TEXT);
                params.putString(QzoneShare.SHARE_TO_QQ_TITLE, shareBean.qq.title);// 标题
                params.putString(QzoneShare.SHARE_TO_QQ_SUMMARY, shareBean.qq.content);// 摘要
                params.putString(QzoneShare.SHARE_TO_QQ_TARGET_URL, shareBean.url);// 内容地址
                ArrayList<String> imageUrls = new ArrayList<>();
                imageUrls.add(shareBean.image);
                params.putStringArrayList(QzoneShare.SHARE_TO_QQ_IMAGE_URL, imageUrls);// 网络图片地址
                break;
            case QzoneShare.SHARE_TO_QZONE_TYPE_IMAGE:
                //分享纯图片
                params.putInt(QzoneShare.SHARE_TO_QZONE_KEY_TYPE, QzoneShare.SHARE_TO_QZONE_TYPE_IMAGE);// 设置分享类型为纯图片分享
                ArrayList<String> images = new ArrayList<>();
                images.add(shareBean.image);
                params.putStringArrayList(QzoneShare.SHARE_TO_QQ_IMAGE_URL, images);// 网络图片地址
                break;
            default:
                break;
        }
        //开始分享
        doShareToQQ(activity, params);
    }

    private void doShareToQQ(final Activity activity, final Bundle params) {
        // QQ分享要在主线程做
        ThreadManager.getMainHandler().post(new Runnable() {

            @Override
            public void run() {
                if (null != InitPlatformManager.mTencent) {
                    InitPlatformManager.mTencent.shareToQzone(activity, params, qZoneShareListener);
                }
            }
        });
    }
```

**注意事项**

**APP ID**
* 一定要替换成自己申请的appid——运行前检查AndroidManifest.xml中与Activity中Tencent.createInstance内使用的appid是否正常。

* 注意在AndroidMaifest.xml中，需要填写tencent您的appid，appid前多了个tencent！

**应用权限的添加**

* Q空间和QQ SSO授权的Activity注册


