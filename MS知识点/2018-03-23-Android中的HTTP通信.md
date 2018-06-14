---
layout:     post
title:      Android中的HTTP通信
date:       2018-03-23
author:     pxf
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Android
---
Android中的HTTP通信
===

文章内容包括：
1.HTTP简介

2.HTTP/1.0和HTTP/1.1之间的区别

3.HTTP的请求头、响应头和状态码

4.Android中的HttpUrlConnection

# 1.Http简介
Http（Hypertext transfer protocol）定义了浏览器怎么向万维网服务器发送万维网文档，以及服务器怎么将文档发送给服务器。从层次上看，http是面向应用层协议的，它是万维网能够可靠交换文档的基础。

## http的工作流程

当用户点击一个链接（假设URL为[http://www.tsinghua.edu.cn/chn/yxsz/index.html](http://www.tsinghua.edu.cn/chn/yxsz/index.html) ），所发生的事件流程：

（1）浏览器分析连接所指向的页面的URL。


（2）浏览器向DNS请求解析www.tsinghua.edu.cn的IP地址。


（3）浏览器解析出服务器的IP地址。


（4）浏览器与服务器建立TCP连接。


（5）浏览器发出取文件指令：GET /chn/yxsz/index.html。


（6）服务器给出响应，将文件index.html发送给浏览器。


（7）释放TCP连接。


（8）浏览器显示index.html的所用信息。

## Http的特点

* （1）**支持客服/服务器（C/S）模式**
* （2）**简单快速**，客户向服务器请求服务时，只需传送请求方法和路径。请求方法常用的有GET、POST、HEAD。每种方法规定了与服务器的连接不同，由于HTTP协议简单，使得HTTP服务器的程序规模更小、因而通信更快。
* （3）HTTP是**无连接的**。无连接意味者HTTP每次只处理一个请求，服务器处理完处理完客户的请求，并且收到客户的应答后，即断开连接，可以节省传输时间。
* （4）HTTP是**无状态的**。无状态意味者HTTP协议对于事务没有记忆能力，缺少状态表示后续处理需要前面的信息，则它必须重传。这可能使它没次连接传输的信息量增大，另一方面、服务器在不需要先前信息就表现的非常快，同时是服务器更容易支持大量的并发的HTTP请求。
**PS**：虽然HTTP是无连接的协议，但HTTP使用了面向连接的运输层协议TCP，因此保证了数据的可靠传输，HTTP不用考虑数据在传输过程中被丢弃了如何重传。

## 2.HTTP/1.0和HTTP/1.1之间的区别

* HTTP/1.0的主要缺点是它使用**非持续连接**每请求一个文档需要两倍的RTT的开销。这时的协议如果一个主页有很多链接的对象（如图片），每个链接都需要建立新的TCP连接，那么每一次链接下载都会导致2×RTT的开销。

* HTTP/1.1协议很好的解决了这个问题，它使用了持续连接，万维网服务器在发送响应后的一段时间内仍然保持着这个连接，是同一个客户可以和该服务器传送后续的HTTP请求报文和响应报文。

## 3.HTTP的请求头、响应头和状态码
请求头（进入简书的请求头，可以通过Firfox浏览器通过开发者选项打开网络查看（快捷键ctrl+shift+Q））。

* GET [http://www.jianshu.com/](http://www.jianshu.com/)
* Host: www.jianshu.com
* User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:47.0) Gecko/20100101 Firefox/47.0
* Accept: text/html,application/xhtml+xml,application/xml;q=0.9,_/_;q=0.8
* Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
* Accept-Encoding: gzip, deflate
* Referer: [https://www.google.com.hk](https://www.google.com.hk/)
* Cookie: （略）
* Connection: keep-alive
* If-None-Match: W/"b4e2a47d84be2df34bb1d5b79be9c040"
* Cache-Control: max-age=0

**下面说下具体的含义**：

* GET定义了请求的方法。同样的方法共有8种(下面会列出）。
* Host：初始URL中的主机和端口。
* User-Agent：浏览器类型，如果Servlet返回的内容与浏览器类型有关则该值非常有用。
* Accept：浏览器可接受的MIME类型。
* Accept-Charset：浏览器可接受的字符集。
* Accept-Language：浏览器所希望的语言种类，当服务器能够提供一种以上的语言版本时要用到。
* Accept-Encoding：浏览器能够进行解码的数据编码方式，比如gzip。Servlet能够向支持gzip的浏览器返回经gzip编码的HTML页面。许多情形下这可以减少5到10倍的下载时间。
* Referer：包含一个URL，用户从该URL代表的页面出发访问当前请求的页面。
* Cookie：这是最重要的请求头信息之一,HTTP请求发送时，会把保存在该请求域名下的所有cookie值一起发送给web服务器。
* Connection： 表示是否需要持久连接。如果Servlet看到这里的值为“Keep- Alive”，或者看到请求使用的是HTTP 1.1（HTTP 1.1默认进行持久连接），它就可以利用持久连接的优点，当页面包含多个元素时（例如Applet，图片），显著地减少下载所需要的时间。要实现这一 点，Servlet需要在应答中发送一个Content-Length头，最简单的实现方法是：先把内容写入 ByteArrayOutputStream，然后在正式写出内容之前计算它的大小。
* Cache-Control:If-None-Match:如果内容未改变返回304代码，参数为服务器先前发送的Etag，与服务器回应的Etag比较判断是否改。
* Cache-Control:指定请求和响应遵循的缓存机制。

## 8种请求方法解释

（摘自：[http://itbilu.com/other/relate/EkwKysXIl.html](http://itbilu.com/other/relate/EkwKysXIl.html))**

* **GET**
请求会显示请求指定的资源。一般来说GET方法应该只用于数据的读取，而不应当用于会产生副作用的非幂等的操作中。GET方法请求指定的页面信息，并返回响应主体，GET被认为是不安全的方法，因为GET方法会被网络蜘蛛等任意的访问。
* **HEAD**
方法与GET方法一样，都是向服务器发出指定资源的请求。但是，服务器在响应HEAD
请求时不会回传资源的内容部分，即：响应主体。这样，我们可以不传输全部内容的情况下，就可以获取服务器的响应头信息。HEAD方法常被用于客户端查看服务器的性能。
* **POST**
请求会向指定资源提交数据，请求服务器进行处理，如：表单数据提交、文件上传等，请求数据会被包含在请求体中。POST方法是非幂等的方法，因为这个请求可能会创建新的资源或/和修改现有资源。
* **PUT**
请求会身向指定资源位置上传其最新内容，PUT方法是幂等的方法。通过该方法客户端可以将指定资源的最新数据传送给服务器取代指定的资源的内容。
* **DELETE**
请求用于请求服务器删除所请求URI（统一资源标识符，Uniform Resource Identifier）所标识的资源。DELETE请求后指定资源会被删除，DELETE方法也是幂等
的。
* **CONNECT**
该方法是HTTP/1.1协议预留的，能够将连接改为管道方式的代理服务器。通常用于[SSL](http://itbilu.com/other/relate/N16Uaoyp.html)加密服务器的链接与非加密的HTTP代理服务器的通信。
* **OPTIONS**
请求与HEAD类似，一般也是用于客户端查看服务器的性能。这个方法会请求服务器返回该资源所支持的所有HTTP请求方法，该方法会用'_'来代替资源名称，向服务器发送OPTIONS请求，可以测试服务器功能是否正常。JavaScript的[XMLHttpRequest](http://itbilu.com/javascript/js/VkiXuUcC.html)对象进行CORS跨域资源共享时，就是使用OPTIONS方法发送嗅探请求，以判断是否有对指定资源的访问权限。允许**_TRACE**请求服务器回显其收到的请求信息，该方法主要用于HTTP请求的测试或诊断。

## 响应头(同样是在请求简书首页的响应头）

*  Cache-Control: max-age=0, private, must-revalidate
*  Connection: keep-alive
*  Content-Encoding: gzip
*  Content-Type: text/html; charset=utf-8
*  Date: Sun, 19 Jun 2016 15:29:41 GMT
*  Etag: W/"e9a43aabd740855cd3fe0097faf6180d"
*  Server: nginx
*  Set-Cookie: （略）
*  Vary: Accept-Encoding
*  X-Request-Id: ce26a795-7e99-4959-a498-45f689471d7f
*  X-Runtime: 0.596683
*  x-content-type-options: nosniff
*  x-frame-options: DENY
*  x-xss-protection: 1; mode=block

**下面说下具体的含义**：

* Cache-Control指定请求和响应遵循的缓存机制。
* Connection：表示是否需要持久连接。
* Content-Encoding 文档的编码（Encode）方法。只有在解码之后才可以得到Content-Type头指定的内容类型。利用gzip压缩文档能够显著地减少HTML文档 的下载时间。
* Content- Type 表示后面的文档属于什么MIME类型。Servlet默认为text/plain，但通常需要显式地指定为text/html。由于经常要设置 * Content-Type，因此HttpServletResponse提供了一个专用的方法setContentTyep。
* Date 当前的GMT时间。你可以用setDateHeader来设置这个头以避免转换时间格式的麻烦。
* Etag 请求变量的实体标签的当前值。
* Server 服务器名字。Servlet一般不设置这个值，而是由Web服务器自己设置。
* Set-Cookie 设置和页面关联的Cookie。
* Vary 告诉下游代理是使用缓存响应还是从原始服务器请求更为详细的请求头与响应头信息，请参考：[HTTP Header 详解](http://itbilu.com/other/relate/EkwKysXIl.html)

## 状态码
* 1XX ：表示通知信息的，如请求收到了或正在处理。
* 2XX ：表示成功，如接收或知道了。
* 3XX ：表示重定向，如要完成还需采取进一步处理。
* 4XX ：表示客户的差错，如请求中有错误的语法或不能完成。
* 5XX ：表示服务器的差错，如服务器失效无法完成请求。

## HTTP协议的主要特点:
 
* 支持客户／服务器模式 
*  简单快速：客户向服务端请求服务时，只需传送请求方式和路径。 
*  灵活：允许传输任意类型的数据对象。由Content-Type加以标记。 
*  无连接：每次响应一个请求，响应完成以后就断开连接。 
*  无状态：服务器不保存浏览器的任何信息。每次提交的请求之间没有关联。

 **非持续性和持续性:**

*  HTTP1.0默认非持续性；HTTP1.1默认持续性 
*  **持续性:** 浏览器和服务器建立TCP连接后，可以请求多个对象 
*  **非持续性:** 浏览器和服务器建立TCP连接后，只能请求一个对象

* * *

**POST和GET的区别**
 
*  Post一般用于更新或者添加资源信息 Get一般用于查询操作，而且应该是安全和幂等的 
*  Post更加安全 Get会把请求的信息放到URL的后面 
*  Post传输量一般无大小限制 Get不能大于2KB 
*  Post执行效率低 Get执行效率略高

* * *

 **为什么POST效率低，Get效率高?** 
*  Get将参数拼成URL,放到header消息头里传递 
*  Post直接以键值对的形式放到消息体中传递。 
*  但两者的效率差距很小很小

* * *

# 4.Android中的HttpUrlConnection

Android中的连接主要是通过HttpUrlConnection来完成的，下面将要从**HttpUrlConnection使用、get和post传递参数、多线程下载**三个方面来看HttpUrlClient的用法：

## （1）HttpUrlConnection的使用格式：
```
//将地址转换为URL`
`URL url = new URL("http://localhost:8080/TestHttpURLConnectionPro/index.jsp"); 
// 此处的urlConnection对象实际上是根据URL的请求协议(此处是http)生成的URLConnection类的子类HttpURLConnection,故此处最好将其转化为HttpURLConnection类型的对象 
`URLConnection rulConnection = url.openConnection();
HttpURLConnection httpUrlConnection = (HttpURLConnection) rulConnection;`
```
**设置HttpUrlClient的连接参数：**
```
`// 设置是否向httpUrlConnection输出，因为这个是post请求，参数要放在。http正文内，因此需要设为true, 默认情况下是false;
httpUrlConnection.setDoOutput(true);
//设置是否从httpUrlConnection读入，默认情况下是true;
httpUrlConnection.setDoInput(true);
// Post 请求不能使用缓存
httpUrlConnection.setUseCaches(false);
// 设定传送的内容类型是可序列化的java对象
// (如果不设此项,在传送序列化对象时,当WEB服务默认的不是这种类型时可能抛java.io.EOFException) httpUrlConnection.setRequestProperty("Content-type", "application/x-java-serialized-object");
// 设定请求的方法为"POST"，默认是GET
httpUrlConnection.setRequestMethod("POST");
// 连接，从上述第2条中url.openConnection()至此的配置必须要在connect之前完成， httpUrlConnection.connect();`
```
对于HttpUrlConnection在代码中的具体用法，看下面都是一样的用法，看过就懂了

## （2）get和post方式传递参数**

### get方式

使用get方式传递参数关键在于URl，在代码中可以看出我们在url中附加了一些数据，实际上get方式就是在通过在url中附加数据来传递参数的，因此采用这种方式是很不安全的。

```
`private void doGet(){
	 try {
	 	url = url + "?name=" + URLEncoder.encode(name,"utf-8") + "&age=" + age; 
	} catch (UnsupportedEncodingException e) {
		 e.printStackTrace();
	 }
	 try {
		 URL httpUrl = new URL(url); //新建URL对象
		 HttpURLConnection conn = (HttpURLConnection) httpUrl.openConnection();//打开一个连接  
		 conn.setRequestMethod("GET");//设置请求方法为GET
		 conn.setReadTimeout(5000);//设置从服务器读取数据的超时限制为5秒
		 BufferedReader reader = new BufferedReader( new 	InputStreamReader(conn.getInputStream()));//获取服务器传递的数据输入流 String str;
		 StringBuffer sb = new StringBuffer(); //存储读取的数据
		 while((str = reader.readLine()) != null){
		//读取数据
		 sb.append(str);
	 }
	 	System.out.println("result:"+sb.toString());
	 } catch (MalformedURLException e) {
	 	e.printStackTrace(); 
	} catch (IOException e) {
	 	e.printStackTrace();
	 }
 }`
```

### POST方式

post传递参数的方式与get是不同的，它会将传递的数据写入到请求的正文中。
```
`private void doPost(){
	 try {
		 URL HttpUrl = new URL(url); 
		 HttpURLConnection conn = (HttpURLConnection) HttpUrl.openConnection();     		
		 conn.setRequestMethod("POST");   
		 conn.setReadTimeout(5000); 
		 OutputStream out = conn.getOutputStream(); //新建输出流对象 
		 String content = "name="+name+"&age="+age;//传递对象 
		 out.write(content.getBytes());//将传递对象转为字符流写入输出流中
		 //下面是对于服务器返回数据的处理
		 BufferedReader reader = new BufferedReader(new InputStreamReader(conn.getInputStream()));
		StringBuffer sb = new StringBuffer(); 
		String str; 
		while((str=reader.readLine())!=null){ 
		sb.append(str); 
	} 
		System.out.println(sb.toString()); 
	} catch (MalformedURLException e) {
	 	e.printStackTrace();
	 } catch (IOException e) {
	 	e.printStackTrace();
	 }
 }`
```
## （3）多线程下载
```
public class DownLoad {

	private Executor threadPool = Executors.newFixedThreadPool(3);
	
	private Handler handler;
	
	public DownLoad(Handler handler){
	    this.handler = handler;
	}
	static class DownLoadRunnable implements Runnable {
	
	    private String url;
	
	    private String fileName; 
	
	    private long start;
	
	    private long end;
	
	    private Handler handler;
	
	    public DownLoadRunnable(String url,String fileName,long start,long end,Handler handler){
	        this.url = url;
	        this.fileName = fileName;
	        this.start = start;
	        this.end = end;
	        this.handler = handler;
	    }
	    @Override
	    public void run() {
	        try {
	            URL httpUrl = new URL(url);
	            HttpURLConnection conn = (HttpURLConnection) httpUrl.openConnection();
	            conn.setRequestMethod("GET");
	            conn.setRequestProperty("Range","bytes="+start+"-"+end);
	            conn.setReadTimeout(5000);
	
	            RandomAccessFile access = new RandomAccessFile(new File(fileName),"rwd");
	            access.seek(start);
	            InputStream in = conn.getInputStream(); 
	            byte[] b = new byte[1024*4];
	            int len=0;
	            while((len=in.read(b))!=-1){
	                access.write(b,0,len);
	            }
	
	            if (access!=null){
	                access.close();
	            }
	            if (in!=null){
	                in.close();
	            }
	
	            Message msg = new Message();
	            msg.what = 1;
	            handler.sendMessage(msg);
	        } catch (IOException e) {
	            e.printStackTrace();
	        }
	    }
	}
	
	public void loadFile(String url) {
	    try {
	        URL httpUrl = new URL(url);
	        HttpURLConnection conn = (HttpURLConnection) httpUrl.openConnection();
	        conn.setRequestMethod("GET");
	        conn.setReadTimeout(5000);
	        int count = conn.getContentLength();
	        int block = count / 3; 
	        String fileName = getFileName(url);
	        File parent = Environment.getExternalStorageDirectory();
	        File download = new File(parent,fileName);
	
	        for (int i=0;i<3;i++){
	            long start = i*block;
	            long end = (i+1)*block-1;
	            if (i==2){
	                end = count;
	            }
	
	            DownLoadRunnable runnable = new DownLoadRunnable(url,download.getAbsolutePath(),start,end,handler);
	            threadPool.execute(runnable);
	        }
	    } catch (IOException e) {
	        e.printStackTrace();
	    }
	}
	public String getFileName(String url){
	    return url.substring(url.lastIndexOf("/")+1);
	}
}
```
# Okhttp介绍

OkHttp是一个高效的Http客户端，有如下的特点：

* 支持HTTP2/SPDY黑科技 
* socket自动选择最好路线，并支持自动重连 
* 拥有自动维护的socket连接池，减少握手次数 
* 拥有队列线程池，轻松写并发 
* 拥有Interceptors轻松处理请求与响应（比如透明GZIP压缩,LOGGING） 
* 基于Headers的缓存策略

之前也有说到,OKhttp是一个相对成熟的解决方案,在Android4.4的源码中已经将HttpURLConnection替换成了OKhttp,所以我们更有理由去相信OKhttp的强大之处.

OkHttp 处理了很多网络疑难杂症：会从很多常用的连接问题中自动恢复。如果您的服务器配置了多个IP地址，当第一个IP连接失败的时候，OkHttp会自动尝试下一个IP。OkHttp还处理了代理服务器问题和SSL握手失败问题。

使用 OkHttp 无需重写您程序中的网络代码。OkHttp实现了几乎和java.net.HttpURLConnection一样的API。如果你用了 Apache HttpClient，则OkHttp也提供了一个对应的okhttp-apache 模块。

 **这里看到一处关于OKhttp的使用问题** 

* 注：在国内使用OkHttp会因为这个问题导致部分酷派手机用户无法联网，所以对于大众app来说，需要等待这个bug修复后再使用。或者尝试使用OkHttp的老版本。 

* 截止到目前，OkHttp一直没有修复，并把修复计划延迟到了OkHttp2.3中。不是所有设备都能重现，仅少量设备会出现这个问题。（如果问题这么明显，OkHttp早就修复了）

## 注意如果使用jar需要导入以下两个包

1.  okhttp3 [下载](http://download.csdn.net/detail/checkiming/9882297)
2.  okio[下载](http://download.csdn.net/detail/checkiming/9882301)

## 地址

[OKhttp官方主页地址](http://square.github.io/okhttp/)

## get参数

```

// 01\. 定义okhttp
OkHttpClient okHttpClient_get = new OkHttpClient();
// 02.请求体
Request request = new Request.Builder()
.get()//get请求方式
.url("http://10.0.3.2:8080/WebServiceTest/servlet/ServcieTest?name=sy")//网址
.build();
Response response = okHttpClient_get.newCall(request).execute();
if (response.isSuccessful()) {
    // 打印数据
    System.out.println(response.body().string());
} else {       
     throw new IOException("Unexpected code " + response);
}

-  enqueue是异步方法

```

## post请求参数

```
// 定义okhttp
OkHttpClient okHttpClient_post_kv = new OkHttpClient();
// 定义请求体
// 执行okhttp
RequestBody body = new FormBody.Builder()
                    .add("name", "sy")//添加参数体
                    .add("age", "18")
                    .build();
Request request = new Request.Builder()
    .post(body) //请求参数
    .url("http://10.0.3.2:8080/WebServiceTest/servlet/ServcieTest")
    .build();
Response response = okHttpClient_post_kv.newCall(request).execute();
System.out.println(response.body().string());

-  enqueue是异步方法
```

## post请求json

```
OkHttpClient okHttpClient_post_json = new OkHttpClient();
String json = "{\n" + " "age": "18",\n" + " "name": "sy"\n" + "}";
RequestBody body =
RequestBody.create(MediaType.parse("application/json;charset=utf-8"), json);
Request request = new Request.Builder()
.post(body)
.url("http://10.0.3.2:8080/WebServiceTest/servlet/ServcieTest")
.build();
Response response = okHttpClient_post_json.newCall(request).execute();
System.out.println(response.body().string());

```

## 上传图片

```
OkHttpClient okHttpClient_upload = new OkHttpClient();
File file = new File(Environment.getExternalStorageDirectory() + "/download", "file.txt");
RequestBody body = RequestBody.create(MediaType.parse("application/octet-stream"), file);
Request request = new Request.Builder()
    .post(body)
    .url("http://10.0.3.2:8080/WebServiceTest/servlet/ServcieTest")
    .build();
Response response = okHttpClient_upload.newCall(request).execute();
System.out.println(response.body().string());
```

## 下载图片

```
OkHttpClient okHttpClient_down = new OkHttpClient();
Request request =
new Request.Builder()
.get()
.url("http://10.0.3.2:8080/WebServiceTest/p22.jpg")
.build();
okHttpClient_down.newCall(request).enqueue(MainActivity.this);
```

```
 /**
 * 超时错误,服务器无响应
 * 
 * @param call
 * @param e
 */
@Override
public void onFailure(Call call, IOException e)
{

}

/**
 * 服务器响应
 * 
 * @param call
 * @param response
 * @throws IOException
 */
@Override
public void onResponse(Call call, Response response)
    throws IOException
{
    InputStream inputStream = response.body().byteStream();
    final BitmapDrawable bitmapDrawable = new BitmapDrawable(inputStream);

    runOnUiThread(new Runnable()
    {
        @Override
        public void run()
        {

            mIv_main_load_image.setImageDrawable(bitmapDrawable);
        }
    });

}

```

* 常用api记录 

```
 OkHttpClient client=new OkHttpClient.Builder() 
 .connectTimeout(60, TimeUnit.SECONDS) //设置连接超时 
 .readTimeout(60, TimeUnit.SECONDS) //设置读超时 
 .writeTimeout(60,TimeUnit.SECONDS) //设置写超时 
 .retryOnConnectionFailure(true) //是否自动重连 
 .build(); //构建OkHttpClient对象 

 client.dispatcher().executorService().shutdown(); //清除并关闭线程池 
 client.connectionPool().evictAll(); //清除并关闭连接池 
 client.cache().close(); //清除cache

```
用之前的OkHttpClient对象创建一个新的OkHttpClient对象 公用线程池

```
OkHttpClient eagerClient = client.newBuilder() 
.readTimeout(500, TimeUnit.MILLISECONDS)
.build();
Request request = new Request.Builder()
.url(“[https://api.github.com/repos/square/okhttpissues](https://api.github.com/repos/square/okhttpissues)“) 
//设置访问url 
.get() //类似的有post、delete、patch、head、put等方法，对应不同的网络请求方法 
.header(“User-Agent”, “OkHttpHeaders.java”) //设置header 
.addHeader(“Accept”, “application/json; q=0.5”) //添加header 
.removeHeader(“User-Agent”) //移除header 
.headers(new Headers.Builder().add(“User-Agent”, “OkHttp Headers.java”).build())//移除原有所有header，并设置新header 
.addHeader(“Accept”, “application/vnd.github.v3+json”) 
.build(); //构建request
```

* RequestBody还有两个子类：FormBody和MultipartBody。 

```
 RequestBody formBody = new FormBody.Builder() //提交表单键值对 
 .add(“platform”, “android”) //添加键值对 
 .add(“name”, “XXX”) 
 .add(“subject”, “Hello”) 
 .addEncoded(URLEncoder.encode(“详细”,”GBK”), //添加已编码的键值对 
 URLEncoder.encode(“无”,”GBK”)) 
 .add(“描述”,”你好”) //其实会自动编码，但是无法控制编码格式 
 .build();

RequestBody requestBody = new MultipartBody.Builder()
.setType(MultipartBody.FORM)
.addPart(
        Headers.of("Content-Disposition", "form-data; name=\"title\""),
        RequestBody.create(null, "Logo"))
.addPart(
        Headers.of("Content-Disposition", "form-data; name=\"image\""),
        RequestBody.create(MediaType.parse("image/png"), new File("website/static/logo.png")))
.addFormDataPart("discription","beautiful")
.build();
```

## Call对象
```
Call call=client.newCall(request); //获取Call对象 
Response response=call.execute(); //同步执行网络请求，不要在主线程执行 
call.enqueue(new Callback()); //异步执行网络请求 
call.cancel(); //取消请求 
call.isCanceled(); //查询是否取消 
call.isExecuted(); //查询是否被执行过 
ResponseBody 
body.contentLength(); //body的长度 
String content=body.string(); //以字符串形式解码body 
byte[] byteContent=body.bytes(); //以字节数组形式解码body 
InputStreamReader reader=body.charStream(); //将body以字符流的形式解码 
InputStream inputStream=body.byteStream(); //将body以字节流的形式解码
```