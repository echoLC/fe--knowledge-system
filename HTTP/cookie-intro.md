# 详解Cookie
## 背景
我们知道 HTTP 是一个无状态的协议，这也是为了保持 HTTP 协议简单的特性。但是在一些购物网站，我们需要知道当前的用户是谁，因为用户会做一些添加购物车、下单的操作。所以我们需要在 HTTP 通信的时候通过一种方式，让服务端能够知道请求用户的状态，实现的方式就是：Cookie。

Cookie 是什么？它就是存储在浏览器本地的一小块数据，只要我们通过 API的方式写入了 Cookie 后，它会在浏览器下一次发起请求的时候携带在请求头中发送给服务器，这样服务器就能知道用户的状态了。

## Cookie的作用
Cookie 曾经一度被滥用来存储客户端数据，因为当时浏览器没有很好的存储客户端数据的方式，但是自从 HTML5 的标准出现，各种存储方式：Session Storage、Local Storage、IndexDB 等百花齐放，Cookie 回归到它的本质，用来告诉服务端当前请求用户的状态。

现在 Cookie 主要有以下的作用：
- 会话状态管理（如用户登录状态、购物车）。
- 个性化设置（如用户自定义设置、主题等）。
- 浏览器行为跟踪（如跟踪分析用户行为等）。
  
## 创建Cookie
客户端和服务端都有可以在用户浏览器中种下 Cookie 的方式。

### Set-Cookie响应头部和Cookie请求头部
如果交给服务端处理，服务端一般是在收到请求后，在响应头里添加一个 `Set-Cookie` 选项，例如：
```
Set-Cookie: username=test;tel=18x00212093
```

通过以上的响应头部，服务端就是告诉客户端保存 Cookie 信息。

之后，客户端发送给服务端的每一次请求，浏览器都会将之前保存的 Cookie 信息通过请求头部再发送给服务器。例如：
```
GET /sample_page.html HTTP/1.1
Host: developer.mozilla.org
Cookie: username=test;tel=18x00212093
```

### JavaScript通过document.cookie设置Cookie
客户端同样可以通过 `document.cookie` API 创建新的 Cookie，语法很简单：
```javascript
document.cookie = 'username=test'; 
document.cookie = 'tel=18x00212093'; 
```
通过访问 `document.cookie` 访问 cookie：
```javascript
document.cookie
// log username=test;tel=18x00212093
```
客户端可以设置和访问 Cookie，这意味着想要获取用发动攻击的人也可以通过 XSS 方式获取Cookie，所以这样对用户来说存在一定的安全隐患。下面会介绍如何通过设置一些属性，解决一些安全问题。

## Cookie 的有效期
如果我们不手动给 Cookie 设置一个有效期，Cookie 默认是一个会话 Cookie，关闭浏览器，Cookie 就被删除了，也就是说它只有在会话期间有效。需要注意的是，有些浏览器提供了会话恢复功能，这种情况下即使关闭了浏览器，会话期Cookie也会被保留下来，就好像浏览器从来没有关闭一样。

我们可以通过设置 Cookie 的 `Expires` 或者 `Max-Age` 属性，使得 Cookie 成为一个持久性的 Cookie，例如：
```
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2020 07:28:00 GMT;
```

## Cookie 的Secure和HttpOnly属性
如果我们给 Cookie 添加了 `Secure` 属性，则就是标记 Cookie 信息只能通过被HTTPS协议加密过的请求发送给服务端，我们知道 HTTPS 协议的加密传输，一定程度上保证了传输数据的安全性，但是仍不建议把敏感的信息存在 Cookie 里面。

前面我们说到如果客户端可以通过 `document.cookie` 获取 Cookie，那么就存在被 XSS 攻击的可能性。所以，如果给 Cookie 信息设置了 `HttpOnly` 属性，那么通过JavaScript的 `document.cookie` API 就无法访问 Cookie。下面是一个完整的例子：
```
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2020 07:28:00 GMT; Secure; HttpOnly
```
## Cookie 的作用域
我们可以通过 `domain` 和 `path` 属性定义 Cookie 的作用域：即Cookie应该发送给哪些URL。

`domain` 属性指定哪些主机可以接收 Cookie，如果不指定，默认为当前文档的主机（不包含子域名）。如果指定了 `domain`，则一般包含子域名。例如，如果设置 domain=mozilla.org，则 Cookie 也包含在子域名中（如developer.mozilla.org）。

`path` 标识指定了主机下的哪些路径可以接受 Cookie（该URL路径必须存在于请求URL中）。以字符 %x2F ("/") 作为路径分隔符，子路径也会被匹配。例如，设置 Path=/docs，则以下地址都会匹配：
- /docs
- /docs/Web/
- /docs/Web/HTTP

## SameSite Cookies
通过设置 `SameSite` 属性要求 Cookie 在跨站请求时不会被发送，从而可以阻止跨站请求伪造攻击: [CSRF](https://developer.mozilla.org/zh-CN/docs/Glossary/CSRF)。SameSite 是相对较新的一个字段，兼容性没有那么好，但是基本上所有的主流浏览器都是支持的：[主流浏览器兼容情况](https://developer.mozilla.org/en-US/docs/Web/HTTP/headers/Set-Cookie#Browser_compatibility)。

下面是一个例子：
```
Set-Cookie: key=value; SameSite=Strict
```
SameSite可以有下面三种值：
- **None**，浏览器会在同站请求、跨站请求下继续发送 cookies，不区分大小写。
- **Strict**，浏览器将只在访问相同站点时发送 cookie。（在原有Cookies的限制条件上的加强，如上文“Cookie的作用域” 所述）。
- **Lax**，在新版本浏览器中，为默认选项，Same-Site Cookies 将会为一些跨站子请求保留，如图片加载或者frames的调用，但只有当用户从外部站点导航到URL时才会发送。如link链接。

如果浏览器不支持 SameSite，相当于设置值为 `None`，Cookies会被包含在任何请求中——包括跨站请求。在新版本的浏览器中，SameSite 的默认属性是 `SameSite=Lax`。如果想要指定Cookies在同站、跨站请求都被发送，那么需要明确指定 SameSite 为 `None`。

## 追踪和隐私
每个Cookie都会有与之关联的域（Domain），如果Cookie的域和页面的域相同，那么我们称这个 Cookie为第一方 Cookie（first-party cookie），如果 Cookie 的域和页面的域不同，则称之为第三方Cookie（third-party cookie.）。现在很多的广告平台就是通过追踪第三方 Cookie 的方式来判定用户的来源，所以前一段时间，Chrome 调整默认的 `SameSite` 值为 `Lax` 后，很多的广告平台通过 Cookie 来跟踪用户的方式就失效了。淘宝和天猫的互跳也是通过第三方 Cookie 的方式实现的，所以在 Chrome 更新后，有那么一小段时间，登录淘宝后，跳转到天猫，登录信息失效。不过，阿里的技术人员也是很快地解决了这个问题。



