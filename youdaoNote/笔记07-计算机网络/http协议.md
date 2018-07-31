## http协议

[【思维导图】深入理解HTTPS原理、过程 ](https://mp.weixin.qq.com/s?__biz=MzI4OTA3NDQ0Nw==&mid=2455544464&idx=1&sn=1bfa7e8f323015b41cb8a2792cd023eb&chksm=fb9cbef0cceb37e6e0586c5156917962be344a3d8b0d40d3c21cb2ef082a94938b2435820e96&scene=21#wechat_redirect)

-   ● HTTP构建于TCP/IP协议之上，默认端口号是80

-   ● HTTP是无连接无状态的

### 一.简介

- 1.Header请求头
- 2.响应头
- 3.状态码

HTTP 协议是以 **ASCII** 码传输，

### 二.请求报文

#### 1.请求方法
HTTP定义了与服务器交互的不同方法，最基本的方法有4种，分别是GET，POST，PUT，DELETE。URL全称是资源描述符，我们可以这样认为：一个URL地址，它用于描述一个网络上的资源，而 HTTP 中的GET，POST，PUT，DELETE就对应着对这个资源的查，增，改，删4个操作。

POST 提交数据的方式
HTTP 协议中规定 POST 提交的数据必须在 body 部分中，但是协议中没有规定数据使用哪种编码方式或者数据格式。实际上，开发者完全可以自己决定消息主体的格式，只要最后发送的 HTTP 请求满足上面的格式就可以。

服务端通常是根据请求头（headers）中的 Content-Type 字段来获知请求中的消息主体是用何种方式编码，再对主体进行解析。所以说到 POST 提交数据方案，包含了 Content-Type 和消息主体编码方式两部分。下面就正式开始介绍它们：
  ● application/x-www-form-urlencoded
这是最常见的 POST 数据提交方式。浏览器的原生 <form> 表单，如果不设置 enctype 属性，那么最终就会以 application/x-www-form-urlencoded 方式提交数据。上个小节当中的例子便是使用了这种提交方式。可以看到 body 当中的内容和 GET 请求是完全相同的。
>   ● multipart/form-data
这又是一个常见的 POST 数据提交的方式。我们使用表单上传文件时，必须让 <form> 表单的 enctype 等于 multipart/form-data。直接来看一个请求示例：

```
POST http://www.example.com HTTP/1.1
Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryrGKCBY7qhFd3TrwA

------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="text"

title
------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="file"; filename="chrome.png"
Content-Type: image/png

PNG ... content of chrome.png ...
------WebKitFormBoundaryrGKCBY7qhFd3TrwA--
```

这个例子稍微复杂点。首先生成了一个 boundary 用于分割不同的字段，为了避免与正文内容重复，boundary 很长很复杂。然后 Content-Type 里指明了数据是以 multipart/form-data 来编码，本次请求的 boundary 是什么内容。消息主体里按照字段个数又分为多个结构类似的部分，每部分都是以 --boundary 开始，紧接着是内容描述信息，然后是回车，最后是字段具体内容（文本或二进制）。如果传输的是文件，还要包含文件名和文件类型信息。消息主体最后以 --boundary-- 标示结束。关于 multipart/form-data 的详细定义，请前往 RFC1867 查看（或者相对友好一点的 MDN 文档）。
这种方式一般用来上传文件，各大服务端语言对它也有着良好的支持。


### 三.响应报文
HTTP 响应与 HTTP 请求相似，HTTP响应也由3个部分构成，分别是：
-   ● 状态行
-   ● 响应头(Response Header)
-   ● 响应正文
状态行由协议版本、数字形式的状态代码、及相应的状态描述，各元素之间以空格分隔。
常见的状态码有如下几种：
 
```
  ● 200 OK 客户端请求成功
  ● 301 Moved Permanently 请求永久重定向
  ● 302 Moved Temporarily 请求临时重定向
  ● 304 Not Modified 文件未修改，可以直接使用缓存的文件。
  ● 400 Bad Request 由于客户端请求有语法错误，不能被服务器所理解。
  ● 401 Unauthorized 请求未经授权。这个状态代码必须和WWW-Authenticate报头域一起使用
  ● 403 Forbidden 服务器收到请求，但是拒绝提供服务。服务器通常会在响应正文中给出不提供服务的原因
  ● 404 Not Found 请求的资源不存在，例如，输入了错误的URL
  ● 500 Internal Server Error 服务器发生不可预期的错误，导致无法完成客户端的请求。
  ● 503 Service Unavailable 服务器当前不能够处理客户端的请求，在一段时间之后，服务器可能会恢复正常。
```

### 四.持久连接
我们知道 HTTP 协议采用“请求-应答”模式，当使用普通模式，即非 Keep-Alive 模式时，每个请求/应答客户和服务器都要新建一个连接，完成之后立即断开连接（HTTP协议为无连接的协议）；当使用 Keep-Alive 模式（又称持久连接、连接重用）时，Keep-Alive 功能使客户端到服务器端的连接持续有效，当出现对服务器的后继请求时，Keep-Alive 功能避免了建立或者重新建立连接。
 
```
  ● HTTP Keep-Alive 简单说就是保持当前的TCP连接，避免了重新建立连接。
  ● HTTP 长连接不可能一直保持，例如 Keep-Alive: timeout=5, max=100，表示这个TCP通道可以保持5秒，max=100，表示这个长连接最多接收100次请求就断开。
  ● HTTP是一个无状态协议，这意味着每个请求都是独立的，Keep-Alive没能改变这个结果。另外，Keep-Alive也不能保证客户端和服务器之间的连接一定是活跃的，在HTTP1.1版本中也如此。唯一能保证的就是当连接被关闭时你能得到一个通知，所以不应该让程序依赖于Keep-Alive的保持连接特性，否则会有意想不到的后果。
  ● 使用长连接之后，客户端、服务端怎么知道本次传输结束呢？两部分：1. 判断传输数据是否达到了Content-Length 指示的大小；2. 动态生成的文件没有 Content-Length ，它是分块传输（chunked），这时候就要根据 chunked 编码来判断，chunked 编码的数据在最后有一个空 chunked 块，表明本次传输数据结束，详见这里。
```


### 五.HTTP Pipelining（HTTP 管线化）
默认情况下 HTTP 协议中每个传输层连接只能承载一个 HTTP 请求和响应，浏览器会在收到上一个请求的响应之后，再发送下一个请求。在使用持久连接的情况下，某个连接上消息的传递类似于请求1 -> 响应1 -> 请求2 -> 响应2 -> 请求3 -> 响应3。
HTTP Pipelining（管线化）是将多个 HTTP 请求整批提交的技术，在传送过程中不需等待服务端的回应。使用 HTTP Pipelining 技术之后，某个连接上的消息变成了类似这样请求1 -> 请求2 -> 请求3 -> 响应1 -> 响应2 -> 响应3。
注意下面几点：

```
● 管线化机制通过持久连接（persistent connection）完成，仅 HTTP/1.1 支持此技术（HTTP/1.0不支持）
  ● 只有 GET 和 HEAD 请求可以进行管线化，而 POST 则有所限制
  ● 初次创建连接时不应启动管线机制，因为对方（服务器）不一定支持 HTTP/1.1 版本的协议
  ● 管线化不会影响响应到来的顺序，如上面的例子所示，响应返回的顺序并未改变
  ● HTTP /1.1 要求服务器端支持管线化，但并不要求服务器端也对响应进行管线化处理，只是要求对于管线化的请求不失败即可
  ● 由于上面提到的服务器端问题，开启管线化很可能并不会带来大幅度的性能提升，而且很多服务器端和代理程序对管线化的支持并不好，因此现代浏览器如 Chrome 和 Firefox 默认并未开启管线化支持
```



### 六.会话跟踪
  1. 什么是会话？
客户端打开与服务器的连接发出请求到服务器响应客户端请求的全过程称之为会话。
  2. 什么是会话跟踪？
会话跟踪指的是对同一个用户对服务器的连续的请求和接受响应的监视。
  3. 为什么需要会话跟踪？
浏览器与服务器之间的通信是通过HTTP协议进行通信的，而HTTP协议是”无状态”的协议，它不能保存客户的信息，即一次响应完成之后连接就断开了，下一次的请求需要重新连接，这样就需要判断是否是同一个用户，所以才有会话跟踪技术来实现这种要求。
  1. 会话跟踪常用的方法:
      a. URL重写
URL(统一资源定位符)是Web上特定页面的地址，URL重写的技术就是在URL结尾添加一个附加数据以标识该会话,把会话ID通过URL的信息传递过去，以便在服务器端进行识别不同的用户。
      b. 隐藏表单域
将会话ID添加到HTML表单元素中提交到服务器，此表单元素并不在客户端显示
      c. Cookie
Cookie是Web服务器发送给客户端的一小段信息，客户端请求时可以读取该信息发送到服务器端，进而进行用户的识别。对于客户端的每次请求，服务器都会将Cookie发送到客户端,在客户端可以进行保存,以便下次使用。
客户端可以采用两种方式来保存这个Cookie对象，一种方式是保存在客户端内存中，称为临时Cookie，浏览器关闭后这个Cookie对象将消失。另外一种方式是保存在客户机的磁盘上，称为永久Cookie。以后客户端只要访问该网站，就会将这个Cookie再次发送到服务器上，前提是这个Cookie在有效期内，这样就实现了对客户的跟踪。
Cookie是可以被禁止的。
      d. Session:
每一个用户都有一个不同的session，各个用户之间是不能共享的，是每个用户所独享的，在session中可以存放信息。
在服务器端会创建一个session对象，产生一个sessionID来标识这个session对象，然后将这个sessionID放入到Cookie中发送到客户端，下一次访问时，sessionID会发送到服务器，在服务器端进行识别不同的用户。
Session的实现依赖于Cookie，如果Cookie被禁用，那么session也将失效。


### 七.跨站攻击
-   ● CSRF（Cross-site request forgery，跨站请求伪造）
CSRF(XSRF) 顾名思义，是伪造请求，冒充用户在站内的正常操作。
例如，一论坛网站的发贴是通过 GET 请求访问，点击发贴之后 JS 把发贴内容拼接成目标 URL 并访问：
  http://example.com/bbs/create_post.php?title=标题&content=内容

那么，我们只需要在论坛中发一帖，包含一链接：
  http://example.com/bbs/create_post.php?title=我是脑残&content=哈哈

只要有用户点击了这个链接，那么他们的帐户就会在不知情的情况下发布了这一帖子。可能这只是个恶作剧，但是既然发贴的请求可以伪造，那么删帖、转帐、改密码、发邮件全都可以伪造。
如何防范 CSRF 攻击？可以注意以下几点：
      ○ 关键操作只接受POST请求
      ○ 验证码
CSRF攻击的过程，往往是在用户不知情的情况下构造网络请求。所以如果使用验证码，那么每次操作都需要用户进行互动，从而简单有效的防御了CSRF攻击。
但是如果你在一个网站作出任何举动都要输入验证码会严重影响用户体验，所以验证码一般只出现在特殊操作里面，或者在注册时候使用。
      ○ 检测 Referer
常见的互联网页面与页面之间是存在联系的，比如你在www.baidu.com应该是找不到通往www.google.com的链接的，再比如你在论坛留言，那么不管你留言后重定向到哪里去了，之前的那个网址一定会包含留言的输入框，这个之前的网址就会保留在新页面头文件的 Referer 中
通过检查Referer的值，我们就可以判断这个请求是合法的还是非法的，但是问题出在服务器不是任何时候都能接受到Referer的值，所以 Referer Check 一般用于监控 CSRF 攻击的发生，而不用来抵御攻击。
      ○ Token
目前主流的做法是使用 Token 抵御 CSRF 攻击。下面通过分析 CSRF 攻击来理解为什么 Token 能够有效
CSRF攻击要成功的条件在于攻击者能够预测所有的参数从而构造出合法的请求。所以根据不可预测性原则，我们可以对参数进行加密从而防止CSRF攻击。
另一个更通用的做法是保持原有参数不变，另外添加一个参数Token，其值是随机的。这样攻击者因为不知道Token而无法构造出合法的请求进行攻击。
Token 使用原则
       
```
■ Token 要足够随机————只有这样才算不可预测
          ■ Token 是一次性的，即每次请求成功后要更新Token————这样可以增加攻击难度，增加预测难度
          ■ Token 要注意保密性————敏感操作使用 post，防止 Token 出现在 URL 中
注意：过滤用户输入的内容不能阻挡 csrf，我们需要做的是过滤请求的来源。
```

-   ● XSS（Cross Site Scripting，跨站脚本攻击）
XSS 全称“跨站脚本”，是注入攻击的一种。其特点是不对服务器端造成任何伤害，而是通过一些正常的站内交互途径，例如发布评论，提交含有 JavaScript 的内容文本。这时服务器端如果没有过滤或转义掉这些脚本，作为内容发布到了页面上，其他用户访问这个页面的时候就会运行这些脚本。
运行预期之外的脚本带来的后果有很多中，可能只是简单的恶作剧——一个关不掉的窗口：
 
```
while (true) {
      alert("你关不掉我~");
  }
```

也可以是盗号或者其他未授权的操作。
XSS 是实现 CSRF 的诸多途径中的一条，但绝对不是唯一的一条。一般习惯上把通过 XSS 来实现的 CSRF 称为 XSRF。
如何防御 XSS 攻击？
理论上，所有可输入的地方没有对输入数据进行处理的话，都会存在XSS漏洞，漏洞的危害取决于攻击代码的威力，攻击代码也不局限于script。防御 XSS 攻击最简单直接的方法，就是过滤用户的输入。
如果不需要用户输入 HTML，可以直接对用户的输入进行 HTML escape 。下面一小段脚本：
 
```
<script>window.location.href=”http://www.baidu.com”;</script>

经过 escape 之后就成了：
  &lt;script&gt;window.location.href=&quot;http://www.baidu.com&quot;&lt;/script&gt;
```

它现在会像普通文本一样显示出来，变得无毒无害，不能执行了。
当我们需要用户输入 HTML 的时候，需要对用户输入的内容做更加小心细致的处理。仅仅粗暴地去掉 script 标签是没有用的，任何一个合法 HTML 标签都可以添加 onclick 一类的事件属性来执行 JavaScript。更好的方法可能是，将用户的输入使用 HTML 解析库进行解析，获取其中的数据。然后根据用户原有的标签属性，重新构建 HTML 元素树。构建的过程中，所有的标签、属性都只从白名单中拿取。

