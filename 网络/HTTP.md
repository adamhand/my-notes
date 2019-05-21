# HTTP
---
# 一、HTTP简介
HTTP协议是Hyper Text Transfer Protocol（超文本传输协议）的缩写，“超文本”即不仅仅是文本，还可以传输HTML 文件, 图片文件等。

HTTP协议工作于客户端-服务端架构为上。浏览器作为HTTP客户端通过URL向HTTP服务端即WEB服务器发送所有请求。Web服务器根据接收到的请求后，向客户端发送响应信息。

# 二、主要特点
>- 简单快速：客户向服务器请求服务时，只需传送请求方法和路径。请求方法常用的有GET、HEAD、POST。每种方法规定了客户与服务器联系的类型不同。由于HTTP协议简单，使得HTTP服务器的程序规模小，因而通信速度很快。
>- 灵活：HTTP允许传输任意类型的数据对象。正在传输的类型由Content-Type加以标记。
>- 无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。
>- 无状态：HTTP协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。
>- 支持B/S及C/S模式。

# 三、基础概念
## URL
URI 包含 URL 和 URN，目前 WEB 只有 URL 比较流行，所以见到的基本都是 URL。

>- URI（Uniform Resource Identifier，统一资源标识符）
>- URL（Uniform Resource Locator，统一资源定位符）
>- URN（Uniform Resource Name，统一资源名称）

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/CS-Notes/master/docs/pics/urlnuri.jpg">
</div>

### 三者的区别
>- URI用来唯一的标识一个资源。Web上可用的每种资源如HTML文档、图像、视频片段、程序等都是一个来URI来标识的。URI一般由三部组成：
①访问资源的命名机制
②存放资源的主机名
③资源自身的名称，由路径表示，着重强调于资源。
>- URL是一种具体的URI，即URL可以用来标识一个资源，而且还指明了如何locate这个资源。采用URL可以用一种统一的格式来描述各种信息资源，包括文件、服务器的地址和目录等。URL一般由三部组成：
①协议(或称为服务方式)
②存有该资源的主机IP地址(有时也包括端口号)
③主机资源的具体地址。如目录和文件名等
>- URN是通过名字来标识资源，比如mailto:java-net@java.sun.com。

### 一个URL的例子
`http://www.aspxfans.com:8080/news/index.asp?boardID=5&ID=24618&page=1#name`

从上面的URL可以看出，一个完整的URL包括以下几部分：

**1.协议部分：**该URL的协议部分为“http：”，这代表网页使用的是HTTP协议。在Internet中可以使用多种协议，如HTTP，FTP等等本例中使用的是HTTP协议。在"HTTP"后面的“//”为分隔符

**2.域名部分：**该URL的域名部分为“www.aspxfans.com”。一个URL中，也可以使用IP地址作为域名使用

**3.端口部分：**跟在域名后面的是端口，域名和端口之间使用“:”作为分隔符。端口不是一个URL必须的部分，如果省略端口部分，将采用默认端口

**4.虚拟目录部分：**从域名后的第一个“/”开始到最后一个“/”为止，是虚拟目录部分。虚拟目录也不是一个URL必须的部分。本例中的虚拟目录是“/news/”

**5.文件名部分：**从域名后的最后一个“/”开始到“？”为止，是文件名部分，如果没有“?”,则是从域名后的最后一个“/”开始到“#”为止，是文件部分，如果没有“？”和“#”，那么从域名后的最后一个“/”开始到结束，都是文件名部分。本例中的文件名是“index.asp”。文件名部分也不是一个URL必须的部分，如果省略该部分，则使用默认的文件名

**6.锚部分：**从“#”开始到最后，都是锚部分。本例中的锚部分是“name”。锚部分也不是一个URL必须的部分

**7.参数部分：**从“？”开始到“#”为止之间的部分为参数部分，又称搜索部分、查询部分。本例中的参数部分为“boardID=5&ID=24618&page=1”。参数可以允许有多个参数，参数与参数之间用“&”作为分隔符。


## 请求和响应报文
### 1. 请求报文
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/CS-Notes/master/docs/pics/HTTP_RequestMessageExample.png">
</div>

请求报文由**请求行（request line）、请求头部（header）、空行和请求数据**四个部分组成。

**第一部分：请求行，用来说明请求类型,要访问的资源以及所使用的HTTP版本**

GET说明请求类型为GET,[/562f25980001b1b106000338.jpg]为要访问的资源，该行的最后一部分说明使用的是HTTP1.1版本。

**第二部分：请求头部，紧接着请求行（即第一行）之后的部分，用来说明服务器要使用的附加信息**

从第二行起为请求头部，HOST将指出请求的目的地.User-Agent,服务器端和客户端脚本都能访问它,它是浏览器类型检测逻辑的重要基础.该信息由你的浏览器来定义,并且在每个请求中自动发送等等

**第三部分：空行，请求头部后面的空行是必须的**

即使第四部分的请求数据为空，也必须有空行

**第四部分：请求数据也叫主体，可以添加任意的其他数据**

### 2. 响应报文
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/CS-Notes/master/docs/pics/HTTP_ResponseMessageExample.png">
</div>

HTTP响应也由四个部分组成，分别是：**状态行、消息报头、空行和响应正文**。

**第一部分：状态行，由HTTP协议版本号， 状态码， 状态消息 三部分组成。**

第一行为状态行，（HTTP/1.1）表明HTTP版本为1.1版本，状态码为200，状态消息为（ok）

**第二部分：消息报头，用来说明客户端要使用的一些附加信息**

第二行和第三行为消息报头，
Date:生成响应的日期和时间；Content-Type:指定了MIME类型的HTML(text/html),编码类型是UTF-8

**第三部分：空行，消息报头后面的空行是必须的**

**第四部分：响应正文，服务器返回给客户端的文本信息。**

空行后面的html部分为响应正文。

# 四、HTTP 方法
根据HTTP标准，HTTP请求可以使用多种请求方法。HTTP1.0定义了三种请求方法： GET, POST 和 HEAD方法。HTTP1.1新增了五种请求方法：OPTIONS, PUT, DELETE, TRACE 和 CONNECT 方法。后来又引入了PATCH方法，是对PUT方法的补充。

## GET
请求指定的页面信息，并返回实体主体。

## HEAD
类似于get请求，只不过返回的响应中没有具体的内容，用于获取报头。主要用于确认 URL 的有效性以及资源更新的日期时间等。

## POST
POST 主要用来传输数据，而 GET 主要用来获取资源。POST请求可能会导致新的资源的建立和/或已有资源的修改。

## PUT
从客户端向服务器传送的数据取代指定的文档的内容。由于自身不带验证机制，任何人都可以上传文件，因此存在安全性问题，一般不使用该方法。

## DELETE
请求服务器删除指定的页面。与 PUT 功能相反，并且同样不带验证机制。

## CONNECT
要求在与代理服务器通信时建立隧道

使用 SSL（Secure Sockets Layer，安全套接层）和 TLS（Transport Layer Security，传输层安全）协议把通信内容加密后经网络隧道传输。

> CONNECT www.example.com:443 HTTP/1.1

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/CS-Notes/master/docs/pics/dc00f70e-c5c8-4d20-baf1-2d70014a97e3.jpg">
</div>

## OPTIONS
查询支持的方法

查询指定的 URL 能够支持的方法。

会返回 Allow: GET, POST, HEAD, OPTIONS 这样的内容。

## TRACE
追踪路径

服务器会将通信路径返回给客户端。

发送请求时，在 Max-Forwards 首部字段中填入数值，每经过一个服务器就会减 1，当数值为 0 时就停止传输。

通常不会使用 TRACE，并且它容易受到 XST 攻击（Cross-Site Tracing，跨站追踪）。主要用于测试或诊断。

## PATCH
对资源进行部分修改

PUT 也可以用于修改资源，但是只能完全替代原始资源，PATCH 允许部分修改。

# 五、HTTP 状态码
服务器返回的 **响应报文** 中第一行为状态行，包含了状态码以及原因短语，用来告知客户端请求的结果。

|状态码|	类别	|原因短语|
|-|-|-|
|1XX	|Informational（信息性状态码）	|接收的请求正在处理|
|2XX	|Success（成功状态码）	|请求正常处理完毕|
|3XX	|Redirection（重定向状态码）	|需要进行附加操作以完成请求|
|4XX	|Client Error（客户端错误状态码）	|服务器无法处理请求|
|5XX	|Server Error（服务器错误状态码）	|服务器处理请求出错|

## 1XX 信息
>- 100 Continue ：表明到目前为止都很正常，客户端可以继续发送请求或者忽略这个响应。

## 2XX 成功
>- 200 OK
>- 204 No Content ：请求已经成功处理，但是返回的响应报文不包含实体的主体部分。一般在只需要从客户端往服务器发送信息，而不需要返回数据时使用。
>- 206 Partial Content ：表示客户端进行了范围请求，响应报文包含由 Content-Range 指定范围的实体内容。

## 3XX 重定向
>- 301 Moved Permanently ：永久性重定向
>- 302 Found ：临时性重定向
>- 303 See Other ：和 302 有着相同的功能，但是 303 明确要求客户端应该采用 GET 方法获取资源。
>- 注：虽然 HTTP 协议规定 301、302 状态下重定向时不允许把 POST 方法改成 GET 方法，但是大多数浏览器都会在 301、302 和 303 状态下的重定向把 POST 方法改成 GET 方法。
>- 304 Not Modified ：如果请求报文首部包含一些条件，例如：If-Match，If-Modified-Since，If-None-Match，If-Range，If-Unmodified-Since，如果不满足条件，则服务器会返回 304 状态码。
>- 307 Temporary Redirect ：临时重定向，与 302 的含义类似，但是 307 要求浏览器不会把重定向请求的 POST 方法改成 GET 方法。

## 4XX 客户端错误
>- 400 Bad Request ：请求报文中存在语法错误。
>- 401 Unauthorized ：该状态码表示发送的请求需要有认证信息（BASIC 认证、DIGEST 认证）。如果之前已进行过一次请求，则表示用户认证失败。
>- 403 Forbidden ：请求被拒绝。
>- 404 Not Found

## 5XX 服务器错误
>- 500 Internal Server Error ：服务器正在执行请求时发生错误。
>- 503 Service Unavailable ：服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。

# 六、HTTP 首部
有 4 种类型的首部字段：**通用首部字段、请求首部字段、响应首部字段和实体首部字段**。

各种首部字段及其含义如下（不需要全记，仅供查阅）：

## 通用首部字段
|首部字段名	|说明|
|-|-|
|Cache-Control	|控制缓存的行为|
|Connection	|控制不再转发给代理的首部字段、管理持久连接|
|Date	|创建报文的日期时间|
|Pragma	|报文指令|
|Trailer	|报文末端的首部一览|
|Transfer-Encoding	|指定报文主体的传输编码方式|
|Upgrade	|升级为其他协议|
|Via	|代理服务器的相关信息|
|Warning	|错误通知|

##请求首部字段
|首部字段名	|说明|
|-|-|
|Accept	|用户代理可处理的媒体类型|
|Accept-Charset	|优先的字符集|
|Accept-Encoding	|优先的内容编码|
|Accept-Language	|优先的语言（自然语言）|
|Authorization	|Web 认证信息|
|Expect	|期待服务器的特定行为|
|From	|用户的电子邮箱地址|
|Host	|请求资源所在服务器|
|If-Match	|比较实体标记（ETag）|
|If-Modified-Since	|比较资源的更新时间|
|If-None-Match	|比较实体标记（与 If-Match 相反）|
|If-Range	|资源未更新时发送实体 Byte 的范围请求|
|If-Unmodified-Since	|比较资源的更新时间（与 If-Modified-Since 相反）|
|Max-Forwards	|最大传输逐跳数|
|Proxy-Authorization	|代理服务器要求客户端的认证信息|
|Range	|实体的字节范围请求|
|Referer	|对请求中 URI 的原始获取方|
|TE	|传输编码的优先级|
|User-Agent	|HTTP 客户端程序的信息|

## 响应首部字段
|首部字段名	|说明|
|-|-|
|Accept-Ranges	|是否接受字节范围请求|
|Age	|推算资源创建经过时间|
|ETag	|资源的匹配信息|
|Location	|令客户端重定向至指定 URI|
|Proxy-Authenticate	|代理服务器对客户端的认证信息|
|Retry-After	|对再次发起请求的时机要求|
|Server	|HTTP 服务器的安装信息|
|Vary	|代理服务器缓存的管理信息|
|WWW-Authenticate	|服务器对客户端的认证信息|

## 实体首部字段
|首部字段名	|说明|
|-|-|
|Allow	|资源可支持的 HTTP 方法|
|Content-Encoding	|实体主体适用的编码方式|
|Content-Language	|实体主体的自然语言|
|Content-Length	|实体主体的大小|
|Content-Location	|替代对应资源的 URI|
|Content-MD5	|实体主体的报文摘要|
|Content-Range	|实体主体的位置范围
|Content-Type	|实体主体的媒体类型|
|Expires	|实体主体过期的日期时间|
|Last-Modified	|资源的最后修改日期时间|

# 七、具体应用
## 连接管理
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/CS-Notes/master/docs/pics/HTTP1_x_Connections.png" width="600">
</div>

### 1. 短连接与长连接
当浏览器访问一个包含多张图片的 HTML 页面时，除了请求访问 HTML 页面资源，还会请求图片资源。如果每进行一次 HTTP 通信就要新建一个 TCP 连接，那么开销会很大。

长连接只需要建立一次 TCP 连接就能进行多次 HTTP 通信。

>- 从 HTTP/1.1 开始默认是长连接的，如果要断开连接，需要由客户端或者服务器端提出断开，使用 Connection : close；
>- 在 HTTP/1.1 之前默认是短连接的，如果需要使用长连接，则使用 Connection : Keep-Alive。

那么http如何判断一个报文结束？
所有http不外乎2情况：

- 无entity body，则"\r\n\r\n"(两个回车符)，判断。
- 有entity body，则用"Content-Length“字段值判断。

参考[http消息包的结束标记](https://bbs.csdn.net/topics/40299857?list=lz)

### 2. 流水线
默认情况下，HTTP 请求是按顺序发出的，下一个请求只有在当前请求收到响应之后才会被发出。由于会受到网络延迟和带宽的限制，在下一个请求被发送到服务器之前，可能需要等待很长时间。

流水线是在同一条长连接上发出连续的请求，而不用等待响应返回，这样可以避免连接延迟。

## Cookie
HTTP 协议是无状态的，主要是为了让 HTTP 协议尽可能简单，使得它能够处理大量事务。HTTP/1.1 引入 Cookie 来保存状态信息。

Cookie 是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器之后向同一服务器再次发起请求时被携带上，用于告知服务端两个请求是否来自同一浏览器。由于之后每次请求都会需要携带 Cookie 数据，因此会带来额外的性能开销（尤其是在移动环境下）。

Cookie 曾一度用于客户端数据的存储，因为当时并没有其它合适的存储办法而作为唯一的存储手段，但现在随着现代浏览器开始支持各种各样的存储方式，Cookie 渐渐被淘汰。新的浏览器 API 已经允许开发者直接将数据存储到本地，如使用 Web storage API（本地存储和会话存储）或 IndexedDB。

### 1. 用途
>- 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
>- 个性化设置（如用户自定义设置、主题等）
>- 浏览器行为跟踪（如跟踪分析用户行为等）

### 2. 创建过程
服务器发送的响应报文包含 Set-Cookie 首部字段，客户端得到响应报文后把 Cookie 内容保存到浏览器中。
```html
HTTP/1.0 200 OK
Content-type: text/html
Set-Cookie: yummy_cookie=choco
Set-Cookie: tasty_cookie=strawberry

[page content]
```
客户端之后对同一个服务器发送请求时，会从浏览器中取出 Cookie 信息并通过 Cookie 请求首部字段发送给服务器。
```html
GET /sample_page.html HTTP/1.1
Host: www.example.org
Cookie: yummy_cookie=choco; tasty_cookie=strawberry
```

### 3. 分类
>- 会话期 Cookie：浏览器关闭之后它会被自动删除，也就是说它仅在会话期内有效。
>- 持久性 Cookie：指定一个特定的过期时间（Expires）或有效期（max-age）之后就成为了持久性的 Cookie。
```html
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT;
```

###4. 作用域
Domain 标识指定了哪些主机可以接受 Cookie。如果不指定，默认为当前文档的主机（不包含子域名）。如果指定了 Domain，则一般包含子域名。例如，如果设置 Domain=mozilla.org，则 Cookie 也包含在子域名中（如 developer.mozilla.org）。

Path 标识指定了主机下的哪些路径可以接受 Cookie（该 URL 路径必须存在于请求 URL 中）。以字符 %x2F ("/") 作为路径分隔符，子路径也会被匹配。例如，设置 Path=/docs，则以下地址都会匹配：

>- /docs
>- /docs/Web/
>- /docs/Web/HTTP

### 5. JavaScript
通过 document.cookie 属性可创建新的 Cookie，也可通过该属性访问非 HttpOnly 标记的 Cookie。
```html
document.cookie = "yummy_cookie=choco";
document.cookie = "tasty_cookie=strawberry";
console.log(document.cookie);
```

### 6. HttpOnly
标记为 HttpOnly 的 Cookie 不能被 JavaScript 脚本调用。跨站脚本攻击 (XSS) 常常使用 JavaScript 的 document.cookie API 窃取用户的 Cookie 信息，因此使用 HttpOnly 标记可以在一定程度上避免 XSS 攻击。
```html
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT; Secure; HttpOnly
```

### 7. Secure
标记为 Secure 的 Cookie 只能通过被 HTTPS 协议加密过的请求发送给服务端。但即便设置了 Secure 标记，敏感信息也不应该通过 Cookie 传输，因为 Cookie 有其固有的不安全性，Secure 标记也无法提供确实的安全保障。

### 8. Session
除了可以将用户信息通过 Cookie 存储在用户浏览器中，也可以利用 Session 存储在服务器端，存储在服务器端的信息更加安全。

Session 可以存储在服务器上的文件、数据库或者内存中。也可以将 Session 存储在 Redis 这种内存型数据库中，效率会更高。

使用 Session 维护用户登录状态的过程如下：

>- 用户进行登录时，用户提交包含用户名和密码的表单，放入 HTTP 请求报文中；
>- 服务器验证该用户名和密码，如果正确则把用户信息存储到 Redis 中，它在 Redis 中的 Key 称为 Session ID；
>- 服务器返回的响应报文的 Set-Cookie 首部字段包含了这个 Session ID，客户端收到响应报文之后将该 Cookie 值存入浏览器中；
>- 客户端之后对同一个服务器进行请求时会包含该 Cookie 值，服务器收到之后提取出 Session ID，从 Redis 中取出用户信息，继续之前的业务操作。

应该注意 Session ID 的安全性问题，不能让它被恶意攻击者轻易获取，那么就不能产生一个容易被猜到的 Session ID 值。此外，还需要经常重新生成 Session ID。在对安全性要求极高的场景下，例如转账等操作，除了使用 Session 管理用户状态之外，还需要对用户进行重新验证，比如重新输入密码，或者使用短信验证码等方式。

### 9. 浏览器禁用 Cookie
此时无法使用 Cookie 来保存用户信息，只能使用 Session。除此之外，不能再将 Session ID 存放到 Cookie 中，而是使用 URL 重写技术，将 Session ID 作为 URL 的参数进行传递。

### 10. Cookie 与 Session 选择
>- Cookie 只能存储 ASCII 码字符串，而 Session 则可以存取任何类型的数据，因此在考虑数据复杂性时首选 Session；
>- Cookie 存储在浏览器中，容易被恶意查看。如果非要将一些隐私数据存在 Cookie 中，可以将 Cookie 值进行加密，然后在服务器进行解密；
>- 对于大型网站，如果用户所有的信息都存储在 Session 中，那么开销是非常大的，因此不建议将所有的用户信息都存储到 Session 中。

## 缓存
### 1. 优点
>- 缓解服务器压力；
>- 降低客户端获取资源的延迟：缓存通常位于内存中，读取缓存的速度更快。并且缓存在地理位置上也有可能比源服务器来得近，例如浏览器缓存。

### 2. 实现方法
>- 让代理服务器进行缓存；
>- 让客户端浏览器进行缓存。

### 3. Cache-Control
HTTP/1.1 通过 Cache-Control 首部字段来控制缓存。
#### **3.1 禁止进行缓存**
no-store 指令规定不能对请求或响应的任何一部分进行缓存。
```html
Cache-Control: no-store
```

#### **3.2 强制确认缓存**
no-cache 指令规定缓存服务器需要先向源服务器验证缓存资源的有效性，只有当缓存资源有效才将能使用该缓存对客户端的请求进行响应。
```html
Cache-Control: no-cache
```

#### **3.3 私有缓存和公共缓存**
private 指令规定了将资源作为私有缓存，只能被单独用户所使用，一般存储在用户浏览器中。
```html
Cache-Control: private
```
public 指令规定了将资源作为公共缓存，可以被多个用户所使用，一般存储在代理服务器中。
```html
Cache-Control: public
```

#### **3.4 缓存过期机制**
max-age 指令出现在请求报文中，并且缓存资源的缓存时间小于该指令指定的时间，那么就能接受该缓存。

max-age 指令出现在响应报文中，表示缓存资源在缓存服务器中保存的时间。
```html
Cache-Control: max-age=31536000
```
Expires 首部字段也可以用于告知缓存服务器该资源什么时候会过期。
```html
Expires: Wed, 04 Jul 2012 08:26:05 GMT
```
>- 在 HTTP/1.1 中，会优先处理 max-age 指令；
>- 在 HTTP/1.0 中，max-age 指令会被忽略掉。

### 4. 缓存验证
需要先了解 ETag 首部字段的含义，它是资源的唯一标识。URL 不能唯一表示资源，例如 `http://www.google.com/` 有中文和英文两个资源，只有 ETag 才能对这两个资源进行唯一标识。
```html
ETag: "82e22293907ce725faf67773957acd12"
```
可以将缓存资源的 ETag 值放入 If-None-Match 首部，服务器收到该请求后，判断缓存资源的 ETag 值和资源的最新 ETag 值是否一致，如果一致则表示缓存资源有效，返回 304 Not Modified。
```html
If-None-Match: "82e22293907ce725faf67773957acd12"
```
Last-Modified 首部字段也可以用于缓存验证，它包含在源服务器发送的响应报文中，指示源服务器对资源的最后修改时间。但是它是一种弱校验器，因为只能精确到一秒，所以它通常作为 ETag 的备用方案。如果响应首部字段里含有这个信息，客户端可以在后续的请求中带上 If-Modified-Since 来验证缓存。服务器只在所请求的资源在给定的日期时间之后对内容进行过修改的情况下才会将资源返回，状态码为 200 OK。如果请求的资源从那时起未经修改，那么返回一个不带有消息主体的 304 Not Modified 响应。
```html
Last-Modified: Wed, 21 Oct 2015 07:28:00 GMT
```
```html
If-Modified-Since: Wed, 21 Oct 2015 07:28:00 GMT
```

## 内容协商
通过内容协商返回最合适的内容，例如根据浏览器的默认语言选择返回中文界面还是英文界面。

### 1. 类型
#### **1.1 服务端驱动型**

客户端设置特定的 HTTP 首部字段，例如 Accept、Accept-Charset、Accept-Encoding、Accept-Language，服务器根据这些字段返回特定的资源。

它存在以下问题：

>- 服务器很难知道客户端浏览器的全部信息；
>- 客户端提供的信息相当冗长（HTTP/2 协议的首部压缩机制缓解了这个问题），并且存在隐私风险（HTTP 指纹识别技术）；
>- 给定的资源需要返回不同的展现形式，共享缓存的效率会降低，而服务器端的实现会越来越复杂。

#### **1.2 代理驱动型**

服务器返回 300 Multiple Choices 或者 406 Not Acceptable，客户端从中选出最合适的那个资源。

### 2. Vary
```html
Vary: Accept-Language
```
在使用内容协商的情况下，只有当缓存服务器中的缓存满足内容协商条件时，才能使用该缓存，否则应该向源服务器请求该资源。

例如，一个客户端发送了一个包含 Accept-Language 首部字段的请求之后，源服务器返回的响应包含 Vary: Accept-Language 内容，缓存服务器对这个响应进行缓存之后，在客户端下一次访问同一个 URL 资源，并且 Accept-Language 与缓存中的对应的值相同时才会返回该缓存。

## 内容编码
内容编码将实体主体进行压缩，从而减少传输的数据量。

常用的内容编码有：gzip、compress、deflate、identity。

浏览器发送 Accept-Encoding 首部，其中包含有它所支持的压缩算法，以及各自的优先级。服务器则从中选择一种，使用该算法对响应的消息主体进行压缩，并且发送 Content-Encoding 首部来告知浏览器它选择了哪一种算法。由于该内容协商过程是基于编码类型来选择资源的展现形式的，在响应的 Vary 首部至少要包含 Content-Encoding。

## 范围请求
如果网络出现中断，服务器只发送了一部分数据，范围请求可以使得客户端只请求服务器未发送的那部分数据，从而避免服务器重新发送所有数据。

### 1. Range
在请求报文中添加 Range 首部字段指定请求的范围。
```html
GET /z4d4kWk.jpg HTTP/1.1
Host: i.imgur.com
Range: bytes=0-1023
```
请求成功的话服务器返回的响应包含 206 Partial Content 状态码。
```html
HTTP/1.1 206 Partial Content
Content-Range: bytes 0-1023/146515
Content-Length: 1024
...
(binary content)
```
### 2. Accept-Ranges
响应首部字段 Accept-Ranges 用于告知客户端是否能处理范围请求，可以处理使用 bytes，否则使用 none。
```html
Accept-Ranges: bytes
```
### 3. 响应状态码
>- 在请求成功的情况下，服务器会返回 206 Partial Content 状态码。
>- 在请求的范围越界的情况下，服务器会返回 416 Requested Range Not Satisfiable 状态码。
>- 在不支持范围请求的情况下，服务器会返回 200 OK 状态码。

## 分块传输编码
Chunked Transfer Coding，可以把数据分割成多块，让浏览器逐步显示页面。

## 多部分对象集合
一份报文主体内可含有多种类型的实体同时发送，每个部分之间用 boundary 字段定义的分隔符进行分隔，每个部分都可以有首部字段。

例如，上传多个表单时可以使用如下方式：
```html
Content-Type: multipart/form-data; boundary=AaB03x

--AaB03x
Content-Disposition: form-data; name="submit-name"

Larry
--AaB03x
Content-Disposition: form-data; name="files"; filename="file1.txt"
Content-Type: text/plain

... contents of file1.txt ...
--AaB03x--
```

## 虚拟主机
HTTP/1.1 使用虚拟主机技术，使得一台服务器拥有多个域名，并且在逻辑上可以看成多个服务器。

## 通信数据转发
### 1. 代理
代理服务器接受客户端的请求，并且转发给其它服务器。

使用代理的主要目的是：

>- 缓存
>- 负载均衡
>- 网络访问控制
>- 访问日志记录

代理服务器分为正向代理和反向代理两种：

>- 用户察觉得到正向代理的存在。

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/CS-Notes/master/docs/pics/a314bb79-5b18-4e63-a976-3448bffa6f1b.png">
</div>
>- 而反向代理一般位于内部网络中，用户察觉不到。

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/CS-Notes/master/docs/notes/pics/2d09a847-b854-439c-9198-b29c65810944.png">
</div>

### 2. 网关
与代理服务器不同的是，网关服务器会将 HTTP 转化为其它协议进行通信，从而请求其它非 HTTP 服务器的服务。

### 3. 隧道
使用 SSL 等加密手段，在客户端和服务器之间建立一条安全的通信线路。

# 八、HTTPs
HTTP 有以下安全性问题：

>- 使用明文进行通信，内容可能会被窃听；
>- 不验证通信方的身份，通信方的身份有可能遭遇伪装；
>- 无法证明报文的完整性，报文有可能遭篡改。

HTTPs 并不是新协议，而是让 HTTP 先和 SSL（Secure Sockets Layer）通信，再由 SSL 和 TCP 通信，也就是说 HTTPs 使用了隧道进行通信。

通过使用 SSL，HTTPs 具有了加密（防窃听）、认证（防伪装）和完整性保护（防篡改）。

<div align="center">    
<img src="https://raw.githubusercontent.com/adamhand/CS-Notes/master/docs/pics/ssl-offloading.jpg" width="600">
</div>

## 加密
### 1. 对称密钥加密
对称密钥加密（Symmetric-Key Encryption），加密和解密使用同一密钥。

>- 优点：运算速度快；
>- 缺点：无法安全地将密钥传输给通信方。

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/CS-Notes/master/docs/pics/7fffa4b8-b36d-471f-ad0c-a88ee763bb76.png" width="500">
</div>

### 2.非对称密钥加密
非对称密钥加密，又称公开密钥加密（Public-Key Encryption），加密和解密使用不同的密钥。

公开密钥所有人都可以获得，通信发送方获得接收方的公开密钥之后，就可以使用公开密钥进行加密，接收方收到通信内容后使用私有密钥解密。

非对称密钥除了用来加密，还可以用来进行签名。因为私有密钥无法被其他人获取，因此通信发送方使用其私有密钥进行签名，通信接收方使用发送方的公开密钥对签名进行解密，就能判断这个签名是否正确。

>- 优点：可以更安全地将公开密钥传输给通信发送方；
>- 缺点：运算速度慢。

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/CS-Notes/master/docs/pics/39ccb299-ee99-4dd1-b8b4-2f9ec9495cb4.png" width="500">
</div>

### 3. HTTPs 采用的加密方式
**HTTPs 采用混合的加密机制，使用非对称密钥加密用于传输对称密钥来保证传输过程的安全性，之后使用对称密钥加密进行通信来保证通信过程的效率。**（下图中的 Session Key 就是对称密钥）

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/CS-Notes/master/docs/pics/How-HTTPS-Works.png" width="500">
</div>

## 认证
通过使用 证书 来对通信方进行认证。

数字证书认证机构（CA，Certificate Authority）是客户端与服务器双方都可信赖的第三方机构。

服务器的运营人员向 CA 提出公开密钥的申请，CA 在判明提出申请者的身份之后，会对已申请的公开密钥做数字签名，然后分配这个已签名的公开密钥，并将该公开密钥放入公开密钥证书后绑定在一起。

进行 HTTPs 通信时，服务器会把证书发送给客户端。客户端取得其中的公开密钥之后，先使用数字签名进行验证，如果验证通过，就可以开始通信了。

通信开始时，客户端需要使用服务器的公开密钥将自己的私有密钥传输给服务器，之后再进行对称密钥加密。
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/CS-Notes/master/docs/pics/2017-06-11-ca.png" width="600">
</div>

## 完整性保护
SSL 提供报文摘要功能来进行完整性保护。

HTTP 也提供了 MD5 报文摘要功能，但不是安全的。例如报文内容被篡改之后，同时重新计算 MD5 的值，通信接收方是无法意识到发生了篡改。

HTTPs 的报文摘要功能之所以安全，是因为它结合了加密和认证这两个操作。试想一下，加密之后的报文，遭到篡改之后，也很难重新计算报文摘要，因为无法轻易获取明文。

##HTTPs 的缺点
>- 因为需要进行加密解密等过程，因此速度会更慢；
>- 需要支付证书授权的高额费用。

# 九、HTTP/2.0
## HTTP/1.x 缺陷
HTTP/1.x 实现简单是以牺牲性能为代价的：

>- 客户端需要使用多个连接才能实现并发和缩短延迟；
>- 不会压缩请求和响应首部，从而导致不必要的网络流量；
>- 不支持有效的资源优先级，致使底层 TCP 连接的利用率低下。

## 二进制分帧层
HTTP/2.0 将报文分成 HEADERS 帧和 DATA 帧，它们都是二进制格式的。
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/CS-Notes/master/docs/pics/86e6a91d-a285-447a-9345-c5484b8d0c47.png" width="400">
</div>

在通信过程中，只会有一个 TCP 连接存在，它承载了任意数量的双向数据流（Stream）。

>- 一个数据流（Stream）都有一个唯一标识符和可选的优先级信息，用于承载双向信息。
>- 消息（Message）是与逻辑请求或响应对应的完整的一系列帧。
>- 帧（Frame）是最小的通信单位，来自不同数据流的帧可以交错发送，然后再根据每个帧头的数据流标识符重新组装。

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/CS-Notes/master/docs/pics/af198da1-2480-4043-b07f-a3b91a88b815.png" width="450">
</div>

##服务端推送
HTTP/2.0 在客户端请求一个资源时，会把相关的资源一起发送给客户端，客户端就不需要再次发起请求了。例如客户端请求 page.html 页面，服务端就把 script.js 和 style.css 等与之相关的资源一起发给客户端。
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/CS-Notes/master/docs/pics/e3f1657c-80fc-4dfa-9643-bf51abd201c6.png" width="600">
</div>

## 首部压缩
HTTP/1.1 的首部带有大量信息，而且每次都要重复发送。

HTTP/2.0 要求客户端和服务器同时维护和更新一个包含之前见过的首部字段表，从而避免了重复传输。

不仅如此，HTTP/2.0 也使用 Huffman 编码对首部字段进行压缩。
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/CS-Notes/master/docs/pics/_u4E0B_u8F7D.png" width="500">
</div>

# 十、HTTP/1.1 新特性
>- 详细内容请见上文
>- 默认是长连接
>- 支持流水线
>- 支持同时打开多个 TCP 连接
>- 支持虚拟主机
>- 新增状态码 100
>- 支持分块传输编码
>- 新增缓存处理指令 max-age

# 十一、GET 和 POST 比较

## 数据传输方式
GET提交的数据会放在URL之后，以?分割URL和传输数据，参数之间以&相连，如`EditPosts.aspx?name=test1&id=123456`。 POST方法是把提交的数据放在HTTP包的Body中。(一个形象地比喻是：**当执行GET请求的时候，要给汽车贴上GET的标签（设置method为GET），而且要求把传送的数据放在车顶上（url中）以方便记录。如果是POST请求，就要在车上贴上POST的标签，并把货物放在车厢里。**)

因为 URL 只支持 ASCII 码，因此 GET 的参数中如果存在中文等字符就需要先进行编码。例如 `中文` 会转换为 %E4%B8%AD%E6%96%87，而空格会转换为 %20。POST 参考支持标准字符集。

## 保密性
GET方式提交数据，会带来安全问题，比如一个登录页面，通过GET方式提交数据时，用户名和密码将出现在URL上，如果页面可以被缓存或者其他人可以访问这台机器，就可以从历史记录获得该用户的账号和密码

但是，不能因为 POST 参数存储在实体主体中就认为它的安全性更高，因为照样可以通过一些抓包工具（Fiddler）查看。

## 幂等性
幂等的 HTTP 方法，同样的请求被执行一次与连续执行多次的效果是一样的，服务器的状态也是一样的。换句话说就是，幂等方法不应该具有副作用（统计用途除外）。

所有的安全方法也都是幂等的。

在正确实现的条件下，GET，HEAD，PUT 和 DELETE 等方法都是幂等的，而 POST 方法不是。

## 传输数据的大小
首先声明：HTTP协议没有对传输的数据大小进行限制，HTTP协议规范也没有对URL长度进行限制。

而在实际开发中存在的限制主要有：

**GET:**特定浏览器和服务器对URL长度有限制，例如 IE对URL长度的限制是2083字节(2K+35)。对于其他浏览器，如Netscape、FireFox等，理论上没有长度限制，其限制取决于操作系 统的支持。

因此对于GET提交时，传输数据就会受到URL长度的 限制。

**POST:**由于不是通过URL传值，理论上数据不受 限。但实际各个WEB服务器会规定对post提交数据大小进行限制，Apache、IIS6都有各自的配置。

## 安全
安全的 HTTP 方法不会改变服务器状态，也就是说它只是可读的。

GET 方法是安全的，而 POST 却不是，因为 POST 的目的是传送实体主体内容，这个内容可能是用户上传的表单数据，上传成功之后，服务器可能把这个数据存储到数据库中，因此状态也就发生了改变。

安全的方法除了 GET 之外还有：HEAD、OPTIONS。

不安全的方法除了 POST 之外还有 PUT、DELETE。

## 速度
GET产生一个TCP数据包；POST产生两个TCP数据包。

对于GET方式的请求，浏览器会把http header和data一并发送出去，服务器响应200（返回数据）；而对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok（返回数据）。

# 参考
[关于HTTP协议，一篇就够了](https://www.jianshu.com/p/80e25cb1d81a)
[HTTP](https://github.com/CyC2018/CS-Notes/blob/master/notes/HTTP.md#%E4%B9%9Dget-%E5%92%8C-post-%E6%AF%94%E8%BE%83)
[GET和POST两种基本请求方法的区别](https://www.cnblogs.com/logsharing/p/8448446.html)
