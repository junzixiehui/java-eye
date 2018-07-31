

客户端访问用户与服务端交互状态.

> 由于http是无状态，所以浏览器会保存一些key/value值。

- 安全问题  伪造cookie
- cookie大小数量限制
- 


#### 结构

- **NAME=VALUE**  	键值对，可以设置要保存的 Key/Value，注意这里的 NAME 不能和其他属性项的名字一样
- Expires	过期时间，在设置的某个时间点后该 Cookie 就会失效，如 expires=Wednesday, 09-Nov-99 23:12:40 GMT
- Domain	生成该 Cookie 的域名，如 domain="xulingbo.net"
- Path	该 Cookie 是在当前的哪个路径下生成的，如 path=/wp-admin/
- Port	该 Cookie 在什么端口下可以回传服务端，如果有多个端口，以逗号隔开，如 Port="80,81,8080"
- Max-Age	最大失效时间，与 Version 0 不同的是这里设置的是在多少秒后失效
- Version	通过 Set-Cookie2 设置的响应头创建必须符合 RFC2965 规范，如果通过 Set-Cookie 响应头设置，默认值为 0，如果要设置为 1，则该 Cookie 要遵循 RFC 2109 规范
- 


> 真正构建 Cookie 是在 org.apache.catalina.connector. Response 类中完成的，调用 generateCookieString 方法将 Cookie 对象构造成一个字符串，构造的字符串的格式如 userName=“junshan”;Version=“1”; Domain=“xulingbo.net”; Max-Age=1000。然后将这个字符串命名为 Set-Cookie 添加到 MimeHeaders 中。

> 在这里有几点需要注意：

> 创建的 Cookie 的 NAME 不能和 Set-Cookie 或者 Set-Cookie2 的属性项值一样，如果一样会抛 IllegalArgumentException 异常。
创建 Cookie 的 NAME 和 VALUE 的值不能设置成非 ASSIC 字符，如果要使用中文，可以通过 URLEncoder 将其编码，否则将会抛 IllegalArgumentException 异常。
当 NAME 和 VALUE 的值出现一些 TOKEN 字符（如“\”、“,”等）时，构建返回头会将该 Cookie 的 Version 自动设置为 1。
当该 Cookie 的属性项中出现 Version 为 1 的属性项时，构建 HTTP 响应头同样会将 Version 设置为 1。
不知道您有没有注意一个问题，就是当我们通过 response.addCookie 创建多个 Cookie 时，这些 Cookie 最终是在一个 Header 项中还是以独立的 Header 存在的，通俗地说也就是我们每次创建 Cookie 时是否都是创建一个以 NAME 为 Set-Cookie 的 MimeHeaders ？答案是肯定的。从上面的时序图中可以看出每次调用 addCookie 的时候，最终都会创建一个 Header，但是我们还不知道最终在请求返回时构造 HTTP 响应头是否将相同 Header 标识的 Set-Cookie 值进行合并。


#### 安全问题

> Cookie 通过把所有要保存的数据通过 HTTP 协议的头部从客户端传递到服务端，又从服务端再传回到客户端，所有的数据都存储在客户端的浏览器里，所以这些 Cookie 数据可以被访问到，就像我们前面通过 Firefox 的插件 HttpFox 可以看到所有的 Cookie 值。不仅可以查看 Cookie，甚至可以通过 Firecookie 插件添加、修改 Cookie，所以 Cookie 的安全性受到了很大的挑战。
相比较而言 Session 的安全性要高很多，因为 Session 是将数据保存在服务端，只是通过 Cookie 传递一个 SessionID 而已，所以 Session 更适合存储用户隐私和重要的数据。