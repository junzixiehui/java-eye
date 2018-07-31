

- tomcat 是servlet容器;
- tomcat并发量  吞吐量 性能


![image](http://www.10tiao.com/img.do?url=http%3A//mmbiz.qpic.cn/mmbiz/yhNp1Qsicg6SJO7gmLXqAf5SmoIMwEOaF6SOGWB6sZicBgcFTHWZ52BibGujUN8vCc9HjYHCD5IoCQwDv95Q1pRyw/0%3Fwx_fmt%3Dpng)



#### 原理

> 服务器的原理 基于 socket 与 serverSocket 实现；
 serverSocket 接受到请求，然后进行封装 request 与 response；
 
 
 - servlet 容器为 servlet 请求调用它的 service 方法。servlet 容器传递一个 javax.servlet.ServletRequest 对象和 javax.servlet.ServletResponse 对象。ServletRequest 对象包括客户端的 HTTP 请求信息，而 ServletResponse 对象封装 servlet 的响应。在 servlet 的生命周期中，service 方法将会给调用多次
 - 
 
#### catalina容器

> 处理请求

- Engine  整个servlet容器
- host 包含数个上下文的虚拟主机
- context  一个web应用，包含多个servlet
- wrapper 一个servlert


#### 生命周期

> 保持组件统一启动和关闭的  实现接口Lifecycle
