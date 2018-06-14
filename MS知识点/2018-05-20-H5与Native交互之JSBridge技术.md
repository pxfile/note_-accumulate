H5与Native交互之JSBridge技术
===

在android中，native与js的通讯方式与ios类似，ios中的通过schema方式在android中也是支持的。

# javascript调用native方式

目前在android中有三种调用native的方式：

## 1.通过schema方式，使用`shouldOverrideUrlLoading`方法对url协议进行解析。

这种js的调用方式与ios的一样，使用iframe来调用native代码。 

## 2.通过在webview页面里直接注入原生js代码方式，使用`addJavascriptInterface`方法来实现。 

在android里实现如下：

```java
class JSInterface {  
    @JavascriptInterface //注意这个代码一定要加上
    public String getUserData() {
        return "UserData";
    }
}
webView.addJavascriptInterface(new JSInterface(), "AndroidJS");  

```

上面的代码就是在页面的window对象里注入了`AndroidJS`对象。在js里可以直接调用

```javascript
alert(AndroidJS.getUserData()) //UserDate  

```

# 3.使用prompt,console.log,alert方式，这三个方法对js里是属性原生的

在android webview这一层是可以重写这三个方法的。一般我们使用prompt，因为这个在js里使用的不多，用来和native通讯副作用比较少。

```java
class YouzanWebChromeClient extends WebChromeClient {  
    @Override
    public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
        // 这里就可以对js的prompt进行处理，通过result返回结果
    }
    @Override
    public boolean onConsoleMessage(ConsoleMessage consoleMessage) {

    }
    @Override
    public boolean onJsAlert(WebView view, String url, String message, JsResult result) {

    }
}

```

# Native调用javascript方式

在android里是使用webview的`loadUrl`进行调用的，如：

```java
// 调用js中的JSBridge.trigger方法
webView.loadUrl("javascript:JSBridge.trigger('webviewReady')");  
```

# 1、加载网页地址

 mBridgeWebview.loadUrl("file:///android_asset/web.html");  

# 2、注册handler，接收来自js的数据data，并可通过onCallBack回传数据

```
mBridgeWebview.registerHandler("submitFromWeb", new BridgeHandler() {
            @Override
            public void handler(String data, CallBackFunction function) {
                if (!TextUtils.isEmpty(data)) {
                    mEditText.setText("通过调用Native方法接收数据：\n" + data);
                }
                function.onCallBack("Native已经接收到数据：" + data + "，请确认！");
            }
        });
```
对应的js代码如下
```
function useClick() {
            var name = document.getElementById("uname").value;
            var pwd = document.getElementById("psw").value;
            var data = "name = " + name + ", password = " + pwd;

            window.WebViewJavascriptBridge.callHandler(
                'submitFromWeb', {'info':data},//data发送给native
                function(responseData) {//接收onCallBack回传回来的数据
                    document.getElementById("show").innerHTML = responseData;
                }
            );
     }
```
上面这种方法是通过native和js定义相同的标识进行通信，还有种方法可直接在js中通过send方法发送，并在native中创建DefaultHandler接收


```
mBridgeWebview.setDefaultHandler(new DefaultHandler(){
            @Override
            public void handler(String data, CallBackFunction function) {
                function.onCallBack("Native已收到消息！");
            }
        }) 
```
对应js代码如下
```
//h5直接通过send向Native发送消息，在DefaultHandler的handler方法里接收，并可通过onCallBack方法回传
    function sendClick() {
        var name = document.getElementById("uname").value;
        var pwd = document.getElementById("psw").value;
        var data = "name = " + name + ", password = " + pwd;

        window.WebViewJavascriptBridge.send(
            data,
            function(responseData) {
                document.getElementById("show").innerHTML = responseData
            }
        );
    }
```
##### 3、定义callHandler，多用于在初始化界面时native向js发送数据渲染界面，同时也可获取来自js的数据

```
mBridgeWebview.callHandler("functionInJs", new Gson().toJson(new UserInfo("liuw", "123456")), new CallBackFunction() {
            @Override
            public void onCallBack(String data) {
                mEditText.setText("向h5发送初始化数据成功，接收h5返回值为：\n" + data);
            }
        });  
```

对应的js代码如下
```
function connectWebViewJavascriptBridge(callback) {
            if (window.WebViewJavascriptBridge) {
                callback(WebViewJavascriptBridge)
            } else {
                document.addEventListener(
                    'WebViewJavascriptBridgeReady'
                    , function() {
                        callback(WebViewJavascriptBridge)
                    },
                    false
                );
            }
        }

// 第一连接时初始化bridage
connectWebViewJavascriptBridge(function(bridge) {
            //注册handler等待java代码调用
            //初始化时获取数据是调用此处代码
            //参数：标识，要传递到JAVA的数据，回调方法。
            //JAVA代码响应的方法：mBridgeWebview.callHandler("functionInJs", new Gson().toJson(实体类对象), new CallBackFunction(){onCallBack(String data)}
            bridge.registerHandler("functionInJs", function(data, responseCallback) {
                document.getElementById("show").innerHTML = ("data from Java: = " + data);
                var responseData = "I am javascript, Data reception success!";
                responseCallback(responseData);
            });
})
```
 
# 4、Native中send方法的使用

这里需要注意下，该方法对应js中的bridge.init处理，此处需加CallBackFunction方法,如果只使用mBridgeWebview.send("")；如：mBridgeWebview.send("hello");，会导致js中只收到通知，接收不到值

```
mBridgeWebview.send("来自java的发送消息！！！", new CallBackFunction() {
            @Override
            public void onCallBack(String data) {
                Toast.makeText(MainActivity.this, "bridge.init初始化数据成功" + data, Toast.LENGTH_SHORT).show();
            }
        });
```

对应的js代码如下

```
function connectWebViewJavascriptBridge(callback) {
            if (window.WebViewJavascriptBridge) {
                callback(WebViewJavascriptBridge)
            } else {
                document.addEventListener(
                    'WebViewJavascriptBridgeReady'
                    , function() {
                        callback(WebViewJavascriptBridge)
                    },
                    false
                );
            }
        }

    // 第一连接时初始化bridage
    connectWebViewJavascriptBridge(function(bridge) {
            //也注册默认的Handler，用来接收java调用的send(string,CallBackFunction)方法
            bridge.init(function(message, responseCallback) {
                console.log('JS got a message', message);
                var data = {
                    'Javascript Responds': '测试中文!'
                };
                console.log('JS responding with', data);
                responseCallback(data);
            });
    })
```
# 5、再介绍一种常规的Native直接向js传递数据的方法

```
  //直接调用nativeFunction方法向H5发送数据    mBridgeWebview.loadUrl("javascript:nativeFunction('" + data + "')");  
```
对应的js代码如下
```
function nativeFunction(data) {    document.getElementById("show").innerHTML = data;
    }
```