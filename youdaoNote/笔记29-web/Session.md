
-  [参考](https://www.ibm.com/developerworks/cn/java/books/javaweb_xlb/10/index.html)
- 多台服务器session共享问题;
- 


- Session需要Cookie的支持，每个Session有一个唯一的ID（name为JSESSIONID的Cookie）
- Session 机制是建立在 Cookie 机制之上的


#### 出现原因

>  Cookie 可以让服务端程序跟踪每个客户端的访问，但是每次客户端的访问都必须传回这些 Cookie，如果 Cookie 很多，这无形地增加了客户端与服务端的数据传输量，而 Session 的出现正是为了解决这个问题。

> 同一个客户端每次和服务端交互时，不需要每次都传回所有的 Cookie 值，而是只要传回一个 ID，这个 ID 是客户端第一次访问服务器的时候生成的，而且每个客户端是唯一的。这样每个客户端就有了一个唯一的 ID，客户端只要传回这个 ID 就行了，这个 ID 通常是 NANE 为 JSESIONID 的一个 Cookie。



#### 工作

> 下面详细讲一下 Session 如何基于 Cookie 来工作。实际上有三种方式能可以让 Session 正常工作：

- 基于 URL Path Parameter，默认支持。
- 基于 Cookie，如果没有修改 Context 容器的 cookies 标识，默认也是支持的。
- 基于 SSL，默认不支持，只有 connector.getAttribute("SSLEnabled") 为 TRUE 时才支持。

> 第一种情况下，当浏览器不支持 Cookie 功能时，浏览器会将用户的 SessionCookieName 重写到用户请求的 URL 参数中，它的传递格式如 /path/Servlet;name=value;name2=value2? Name3=value3，其中“Servlet；”后面的 K-V 就是要传递的 Path Parameters，服务器会从这个 Path Parameters 中拿到用户配置的 SessionCookieName。关于这个 SessionCookieName，如果在 web.xml 中配置 session-config 配置项，其 cookie-config 下的 name 属性就是这个 SessionCookieName 值。如果没有配置了 session-config 配置项，默认的 SessionCookieName 就是大家熟悉的“JSESSIONID”。需要说明的一点是，与 Session 关联的 Cookie 与其他 Cookie 没有什么不同。接着 Request 根据这个 SessionCookieName 到 Parameters 中拿到 Session ID 并设置到 request.setRequestedSessionId 中。

请注意，如果客户端也支持 Cookie，Tomcat 仍然会解析 Cookie 中的 Session ID，并会覆盖 URL 中的 Session ID。

如果是第三种情况，将会根据 javax.servlet.request.ssl_session 属性值设置 Session ID。



#### Session 如何工作

> 有了 Session ID 服务端就可以创建 HttpSession 对象了，第一次触发通过 request.getSession() 方法。如果当前的 Session ID 还没有对应的 HttpSession 对象，那么就创建一个新的，并将这个对象加到 org.apache.catalina. Manager 的 sessions 容器中保存。Manager 类将管理所有 Session 的生命周期，Session 过期将被回收，服务器关闭，Session 将被序列化到磁盘等。只要这个 HttpSession 对象存在，用户就可以根据 Session ID 来获取这个对象，也就达到了状态的保持。



#### 安全


    服务器端通过读取 Http 头部 Cookie 部分 JSESSIONID 找到用户所属的 Session

    关闭浏览器只是 JSESSIONID 这个 Cookie 被删除；服务器端的 Session 不会被删除。删除时间是通过session-timeout配置的

> 有一种网络攻击方法叫 Cookie/Session 欺骗，比如某管理员用户登录到系统了，如果我们趁他不在电脑旁边的时候把他的 JSESSIONID 复制走；然后打开浏览器访问相同的网址，通过浏览器设置上 JSESSIONID，再次刷新，你会发现已经登录成功了。也就是说服务器端其实只认 JSESSIONID，它甚至无法区分究竟有多少管理员“同时在线”。


#### 清除session

Session删除的时间是：
- 1）Session超时：超时指的是连续一定时间服务器没有收到该Session所对应客户端的请求，并且这个时间超过了服务器设置的Session超时的最大时间。
- 2）程序调用HttpSession.invalidate()
- 3）服务器关闭或服务停止