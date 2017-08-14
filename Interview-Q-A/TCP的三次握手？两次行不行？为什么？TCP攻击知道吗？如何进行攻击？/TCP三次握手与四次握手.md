#TCP三次握手与四次握手

引言

TCP三次握手和四次挥手不管是在开发还是面试中都是一个非常重要的知识点，它是我们优化web程序性能的基础。但是大部分教材都对这部分解释的比较抽象，本文我们就利用wireshark来抓包以真正体会整个流程的细节。

**三次握手**

根据下面这幅图我们来看一下TCP三次握手。p.s: 每个箭头代表一次握手。

![](http://img.mp.itc.cn/upload/20170728/d78b461faf214b3fa51babdbf67b0c27.jpg)

tcp三次握手

**第一次握手**

client发送一个SYN(J)包给server，然后等待server的ACK回复，进入SYN-SENT状态。p.s: SYN为synchronize的缩写，ACK为acknowledgment的缩写。

**第二次握手**

server接收到SYN(seq=J)包后就返回一个ACK(J+1)包以及一个自己的SYN(K)包，然后等待client的ACK回复，server进入SYN-RECIVED状态。

**第三次握手**

client接收到server发回的ACK(J+1)包后，进入ESTABLISHED状态。然后根据server发来的SYN(K)包，返回给等待中的server一个ACK(K+1)包。等待中的server收到ACK回复，也把自己的状态设置为ESTABLISHED。到此TCP三次握手完成，client与server可以正常进行通信了。

**为什么要进行三次握手**

我们来看一下为什么需要进行三次握手，两次握手难道不行么？这里我们用一个生活中的具体例子来解释就很好理解了。我们可以将三次握手中的客户端和服务器之间的握手过程比喻成A和B通信的过程：

* 在第一次通信过程中，A向B发送信息之后，B收到信息后可以确认自己的收信能力和A的发信能力没有问题。

* 在第二次通信中，B向A发送信息之后，A可以确认自己的发信能力和B的收信能力没有问题，但是B不知道自己的发信能力到底如何，所以就需要第三次通信。

* 在第三次通信中，A向B发送信息之后，B就可以确认自己的发信能力没有问题。

**wireshark**

上面分析还不够形象，很容易忘记，下面我们利用wireshark来证明一下上面的分析过程。从下面的的输出就可以很容易看出来，必须要经过前面的三次tcp请求才会有起一次http请求。

第一次请求客户端发送一个SYN包，序列号是0。

![](http://img.mp.itc.cn/upload/20170728/86c6f8ad040f4d9f8a28ec2fbe8af78b_th.jpg)

wireshark-tcp-01

第二次请求服务器会发送一个SYN和一个ACK包，序列号是0，ack号是1。

![](http://img.mp.itc.cn/upload/20170728/4ecee24e2ffe4ff793d5365a4fc66ee3_th.jpg)

wireshark-tcp-02

第三次本地客户端请求会发送一个ACK包，序列号是1，ack号是1来回复服务器。

![](http://img.mp.itc.cn/upload/20170728/a50a172d31de46efb78db4eb5b08f969_th.jpg)

wireshark-tcp-03

**四次挥手**

以下面这张图为例，我们来分析一下TCP四次挥手的过程。

![](http://img.mp.itc.cn/upload/20170728/58fbb0f2151848619c149115b01b07ea.jpg)

tcp四次挥手

**第一次挥手**

client发送一个FIN(M)包，此时client进入FIN-WAIT-1状态，这表明client已经没有数据要发送了。

**第二次挥手**

server收到了client发来的FIN(M)包后，向client发回一个ACK(M+1)包，此时server进入CLOSE-WAIT状态，client进入FIN-WAIT-2状态。

**第三次挥手**

server向client发送FIN(N)包，请求关闭连接，同时server进入LAST-ACK状态。

**第四次挥手**

client收到server发送的FIN(N)包，进入TIME-WAIT状态。向server发送ACK(N+1)包，server收到client的ACK(N+1)包以后，进入CLOSE状态；client等待一段时间还没有得到回复后判断server已正式关闭，进入CLOSE状态。