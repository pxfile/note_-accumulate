Android中的HTTP通信
===

# Session就是一种保存上下文信息的机制，它通过SessionID来区分不同的客户。请简单描述它的三种保存Session ID的方式。

*   ①使用Cookie：保存session id的方式可以采用Cookie，这样在交互过程中浏览器可以自动的按照规则把这个标识发送给服务器。

*   ②URL重写：由于Cookie可以被人为的禁止，必须有其它的机制以便在Cookie被禁止时仍然能够把Session ID传递回服务器，经常采用的一种技术叫做URL重写，就是把Session ID附加在URL路径的后面，附加的方式也有两种，一种是作为URL路径的附加信息，另一种是作为查询字符串附加在URL后面。网络在整个交互过程中始终保持状态，就必须在每个客户端可能请求的路径后面都包含这个Session ID。

*   ③表单隐藏字段：另一种技术叫做表单隐藏字段。就是服务器会自动修改表单，添加一个隐藏字段，以便在表单提交时能够把Session ID传递回服务器。

# 请解释什么叫TCP和UDP。

答案

*   ①TCP叫传输控制协议，它是一种可靠的连接，即有服务器和客户端，只有当双方建立好连接后，才能互相发送数据。（TCP---传输控制协议，提供的是面向连接、可靠的字节流服务。当客户和服务器彼此交换数据前，必须先在双方之间建立一个TCP连接，之后才能传输数据。TCP提供超时重发，丢弃重复数据，检验数据，流量控制等功能，保证数据能从一端传到另一端。）

*   ②UDP叫用户数据报协议，它是一种不可靠的连接，即发送发在没有建立好连接的情况下，就可以直接向目标主机发送数据。（UDP---用户数据报协议，是一个简单的面向数据报的运输层协议。UDP不提供可靠性，它只是把应用程序传给IP层的数据报发送出去，但是并不能保证它们能到达目的地。由于UDP在传输数据报前不用在客户和服务器之间建立一个连接，且没有超时重发等机制，故而传输速度很快。）

# 请简析TCP和UDP的区别。

答案：http://blog.sina.com.cn/s/blog_493309600100clrw.html

*   TCP---传输控制协议，提供的是面向连接、可靠的字节流服务。当客户和服务器彼此交换数据前，必须先在双方之间建立一个TCP连接，之后才能传输数据。TCP提供超时重发，丢弃重复数据，检验数据，流量控制等功能，保证数据能从一端传到另一端。

*   UDP---用户数据报协议，是一个简单的面向数据报的运输层协议。UDP不提供可靠性，它只是把应用程序传给IP层的数据报发送出去，但是并不能保证它们能到达目的地。由于UDP在传输数据报前不用在客户和服务器之间建立一个连接，且没有超时重发等机制，故而传输速度很快。

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
（5）服务器给出响应，将文件index.html发送给浏览器。
（6）释放TCP连接。
（7）浏览器显示index.html的所用信息。

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

