


既然 Cookie 有以上这些问题，Session 也有它的好处，为何不结合使用 Session 和 Cookie 呢？下面是分布式 Session 框架可以解决的问题：

- Session 配置的统一管理。
- Cookie 使用的监控和统一规范管理。
- Session 存储的多元化。
- Session 配置的动态修改。
- Session 加密 key 的定期修改。
充分的容灾机制，保持框架的使用稳定性。
- Session 各种存储的监控和报警支持。
- Session 框架的可扩展性，兼容更多的 session 机制如 wapSession。
- 跨域名 Session 与 Cookie
- 如何共享，现在同一个网站可能存在多个域名，如何将 Session 和 Cookie 在不同的域名之间共享是一个具有挑战性的问题。
- 

#### 总体实现思路


> 分布式 Session 框架的架构图如图 10-9 所示。
为了达成上面所说的几点目标，我们需要一个服务订阅服务器，在应用启动时可以从这个订阅服务器订阅这个应用需要的可写 Session 项和可写 Cookie 项，这些配置的 Session 和 Cookie 可以限制这个应用能够使用哪些 Session 和 Cookie，甚至可以控制 Session 和 Cookie 可读或者可写。这样可以精确地控制哪些应用可以操作哪些 Session 和 Cookie，可以有效控制 Session 的安全性和 Cookie 的数量。

![image](https://www.ibm.com/developerworks/cn/java/books/javaweb_xlb/10/image022.png)



统一通过订阅服务器推送配置可以有效地集中管理资源，所以可以省去每个应用都来配置 Cookie，简化 Cookie 的管理。如果应用要使用一个新增 Cookie，可以通过一个统一的平台来申请，申请通过才将这个配置项增加到订阅服务器。如果是一个所有应用都要使用的全局 Cookie，那么只需将这个 Cookie 通过订阅服务器统一推送过去就行了，省去了要在每个应用中手动增加 Cookie 的配置。

关于这个订阅服务器现在有很多开源的配置服务器，如 Zookeeper 集群管理服务器，可以统一管理所有服务器的配置文件。

由于应用是一个集群，所以不可能将创建的 Session 都保存在每台应用服务器的内存中，因为如果每台服务器有几十万的访问用户，服务器的内存肯定不够用，即使内存够用，这些 Session 也无法同步到这个应用的所有服务器中。所以要共享这些 Session 必须将它们存储在一个分布式缓存中，可以随时写入和读取，而且性能要很好才能满足要求。当前能满足这个要求的系统有很多，如 MemCache 或者淘宝的开源分布式缓存系统 Tair 都是很好的选择。

解决了配置和存储问题，下面看一下如何存取 Session 和 Cookie。

既然是一个分布式 Session 的处理框架，必然会重新实现 HttpSession 的操作接口，使得应用操作 Session 的对象都是我们实现的 InnerHttpSession 对象，这个操作必须在进入应用之前完成，所以可以配置一个 filter 拦截用户的请求。

先看一下如何封装 HttpSession 对象和拦截请求，图 10-10 是时序图。

我们可以在应用的 web.xml 中配置一个 SessionFilter，用于在请求到达 MVC 框架之前封装 HttpServletRequest 和 HttpServletResponse 对象，并创建我们自己的 InnerHttpSession 对象，把它设置到 request 和 response 对象中。这样应用系统通过 request.getHttpSession() 返回的就是我们创建的 InnerHttpSession 对象了，我们可以拦截 response 的 addCookies 设置的 Cookie。

在时序图中，应用创建的所有 Session 对象都会保存在 InnerHttpSession 对象中，当用户的这次访问请求完成时，Session 框架将会把这个 InnerHttpSession 的所有内容再更新到分布式缓存中，以便于这个用户通过其他服务器再次访问这个应用系统。另外，为了保证一些应用对 Session 稳定性的特殊要求可以将一些非常关键的 Session 再存储到 Cookie 中，如当分布式缓存存在问题时，可以将部分 Session 存储到 Cookie 中，这样即使分布式缓存出现问题也不会影响关键业务的正常运行。


该框架还能处理 Cookie 被盗取的问题。如您的密码没有丢失，但是您的账号却有可能被别人登录的情况，这种情况很可能就是因为您登录成功后，您的 Cookie 被别人盗取了，盗取您的 Cookie 的人将您的 Cookie 加入到他的浏览器，然后他就可以通过您的 Cookie 正常访问您的个人信息了，这是一个非常严重的问题。在这个框架中我们可以设置一个 Session 签名，当用户登录成功后我们根据用户的私密信息生成的一个签名，以表示当前这个唯一的合法登录状态，然后将这个签名作为一个 Cookie 在当前这个用户的浏览器进程中和服务器传递，用户每次访问服务器都会检查这个签名和从服务端分布式缓存中取得的 Session 重新生成的签名是否一致，如果不一致，显然这个用户的登录状态不合法，服务端将清除这个 sessionID 在分布式缓存中的 Session 信息，让用户重新登录。