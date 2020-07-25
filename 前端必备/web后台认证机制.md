# web后台认证机制

### 1.HTTP基本认证（HTTP Basic Authentication）

##### 1.1概念

HTTP Basic Auth简单点说明就是每次请求API时都提供用户的username和password。是浏览器遵守http协议实现的基本授权方式。

##### 1.2认证过程

![image-20200705201511933](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200705201513.png)

-  客户端向服务器请求数据，请求的内容可能是一个网页或者是一个ajax异步请求，此时，假设客户端尚未被验证，则客户端提供如下请求至服务器:

```json
Get /index.html HTTP/1.0 
Host:www.google.com
```

-  服务器向客户端发送验证请求代码401,（WWW-Authenticate: Basic realm=”google.com”这句话是关键，如果没有客户端不会弹出用户名和密码输入界面）服务器返回的数据大抵如下：

```json
HTTP/1.0 401 Unauthorised 
Server: SokEvo/1.0 
WWW-Authenticate: Basic realm=”google.com” 
Content-Type: text/html 
Content-Length: xxx
```

- 当符合http1.0或1.1规范的客户端（如IE，FIREFOX）收到401返回值时，将自动弹出一个登录窗口，要求用户输入用户名和密码。

  ![image-20200725102401916](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200725102412.png)

- 用户输入用户名和密码后，将用户名及密码以BASE64加密方式加密，并将密文放入前一条请求信息中，则客户端发送的第一条请求信息则变成如下内容：

```json
Get /index.html HTTP/1.0 
Host:www.google.com 
Authorization: Basic d2FuZzp3YW5n
```

> 注：`d2FuZzp3YW5n`表示加密后的用户名及密码（用户名：密码 然后通过base64加密，加密过程是浏览器默认的行为，不需要我们人为加密，我们只需要输入用户名密码即可）

- 服务器收到上述请求信息后，将 `Authorization` 字段后的用户信息取出、解密，将解密后的用户名及密码与用户数据库进行比较验证，如用户名及密码正确，服务器则根据请求，将所请求资源发送给客户端。

##### 1.3效果

客户端未认证的时候，会弹出用户名密码输入框，这个时候请求时属于 `pending` 状态，当用户输入用户名密码的时候客户端会再次发送带 `Authentication` 头的请求：

![image-20200705202133791](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200705202139.png)

认证成功之后：

![image-20200705202505488](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200705202507.png)

##### 1.4优缺点

优点：

- 基本所有流行的网页浏览器都支持基本认证。
- 是配合restful api使用的最简单的认证方式。

缺点：

- 以明文传输的密钥和口令很容易被拦截，该方案同样没有对服务器返回的信息提供保护。
- 现存的浏览器保存认证信息直到标签页或浏览器被关闭，或者用户清除历史记录。HTTP没有为服务器提供一种方法指示客户端丢弃这些被缓存的密钥。这意味着服务器端在用户不关闭浏览器的情况下，并没有一种有效的方法来让用户注销。

### 2.session-cookie认证

##### 2.1概念

Cookie认证机制就是为一次请求认证在服务端创建一个Session对象，同时在客户端的浏览器端创建了一个Cookie对象；通过客户端带上来Cookie对象来与服务器端的session对象匹配来实现状态管理的。

##### 2.2cookie原理

在浏览器第一次向服务器发送请求时，服务器在 `response` 头部设置 `Set-Cookie` 字段，浏览器收到响应就会设置 `cookie` 并存储，在下一次该浏览器向服务器发送请求时，就会在 `request` 头部自动带上 `Cookie` 字段，服务器端收到该 `cookie` 用以区分不同的浏览器。

##### 2.3认证过程

![image-20200705203806208](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200705203808.png)

- 服务器在接受客户端首次访问时在服务器端创建seesion，然后保存seesion(我们可以将seesion保存在 内存中，也可以保存在redis中，推荐使用后者)，然后给这个session生成一个唯一的标识字符串,然后在响应头中种下这个唯一标识字符串。
- 签名。这一步通过秘钥对sid进行签名处理，避免客户端修改sid。(非必需步骤)
- 浏览器中收到请求响应的时候会解析响应头，然后将sid保存在本地cookie中，浏览器在下次http请求的请求头中会带上该域名下的cookie信息。
- 服务器在接受客户端请求时会去解析请求头cookie中的sid，然后根据这个sid去找服务器端保存的该客户端的session，然后判断该请求是否合法。

##### 2.4优缺点

缺点：

- 服务器内存消耗大：用户每做一次应用认证,应用就会在服务端做一次记录,以方便用户下次请求时使用。通常来讲session保存在内存中,随着认证用户的增加,服务器的消耗就会很大。
- 在不是HTTPS协议下易受到CSRF攻击：每次发送请求，用户的信息就携带在cookie中，然而cookie很容易被挟持，这样子用户就很容易被伪造。
- 认证方式仅仅局限于在浏览器中使用，cookie是浏览器端的机制，如果在app端就无法使用cookie。

### 3.Token认证

##### 3.1概念

当用户第一次登录后，服务器生成一个token并将此token返回给客户端，以后客户端只需带上这个token前来请求数据即可，**无需再次带上用户名和密码。**

最简单的token组成:<u>uid</u>(用户唯一的身份标识)、<u>time</u>(当前时间的时间戳)、<u>sign</u>(签名，由token的前几位+盐以哈希算法压缩成一定长的十六进制字符串，可以防止恶意第三方拼接token请求服务器)。还可以把不变的参数也放进token，避免多次查库。

##### 3.2认证过程

![image-20200705205845605](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200705205846.png)

- 客户端使用用户名跟密码请求登录
- 服务端收到请求，去验证用户名与密码
- 验证成功后，服务端会签发一个 `Token`，再把这个 `Token` 发送给客户端
- 客户端收到 `Token` 以后可以把它存储起来，比如放在 `Cookie` 里或者`Local Storage` 里
- 客户端每次向服务端请求资源的时候需要带着服务端签发的 `Token`
- 服务端收到请求，然后去验证客户端请求里面带着的 `Token`（request头部添加Authorization），如果验证成功，就向客户端返回请求的数据 ，如果不成功返回401错误码，鉴权失败。

##### 3.3优缺点

优点：

- 认证方式可以支持多种客户端，而不仅是浏览器。
- 支持跨域访问（cookie不支持跨域是因为cookie本身的同源策略所导致，但是token完全由应用管理，所以它可以避开同源策略）
- 无状态（服务器扩展性强）：token机制在服务端不需要存储session信息，因为Token 自身包含了所有登录用户的信息，只需要在客户端的cookie或本地介质存储状态信息。
- 不使用 `cookie` 就可以规避CSRF攻击。

缺点：

- 加密解密消耗使得 `token` 认证比 `session-cookie` 更消耗性能。
- `token` 比 `sessionId` 大，更占带宽。

##### 3.4扩展问题

###### token生成机制？

![image-20200705212433242](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200705212434.png)

- 将荷载payload，以及Header信息进行Base64加密，形成密文payload密文，header密文。
- 将payload密文和header密文用英文句号连接在一起，用服务端秘钥进行HS256（对称算法）加密，生成签名。
- 将payload密文，header密文以及签名用英文句号连接在一起形成”header密文+” .“+”payload密文“+“.”+“签名”“这种形式的token。

> 为啥使用HS256（对称算法）？
>
> 非对称算法存在一方签发，另一方验证的情况。但是在这里，签发和验证都是同一方，所以对称加密算法就能达到要求，而对称算法比非对称算法要快得多（可达数十倍差距）。
>
> 更进一步思考，对称加密算法除了加密，还带有还原加密内容的功能，而这一功能在对 Token 签名时并无必要——既然不需要解密，摘要（散列）算法就会更快。可以指定密码的散列算法，自然是 HMAC。（但是这里还是用到了对称算法HS256）

###### token解析机制？

用户请求时携带此token（分为三部分，header密文，payload密文，签名）到服务端，服务器是怎么检验Token合法性的呢？服务器需要确认两件事情（Token有没有过期以及Token是否是自己签发的）

- 服务端通过用Base64解密第二部分（payload密文），可以知道荷载中授权时间，以及有效期。通过这个与当前时间对比发现token是否过期。
- 服务端使用原来的秘钥与密文(header密文+"."+payload密文)同样进行HS256运算，然后用生成的签名与token携带的签名进行对比，若一致说明token合法，不一致说明原文被修改。

###### token存在cookie中为什么可以规避CSRF攻击？

即使在客户端使用cookie存储token，cookie也<u>仅仅是一个存储机制而不是用于认证</u>。

因为就算黑客劫持了 cookie中的token,但是服务器不通过获取cookie去认证，所以黑客获得了token也无用。

比如：

1.使用url参数传递

```js
//网络请求函数：
function () {
    //...
    let token = Utils.getCookie(Const.TOKEN);
    url += `?_security_token=${token}`;
    return ajax(url, payload);
}
```

![image-20200705211745837](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200705211747.png)

2.通过HTTP Request Headers 头部传入到服务器

```js
//axios的处理
axios.interceptors.request.use(config => {
  const token = Cookie.get('_security_token') || Cookie.get('_security_token_inc') || '';
  config.headers.common["Security-Token"] = token;
  return config;
});
//或者自己的网络库
function() {
    //...
    return ajax(host + params.url, {
                data: params.opts,
                method: params.method,
                contentType,
                headers: {_security_token: '02_zMKe_99eah5s0G1'}
    });
}
```

![image-20200705211909335](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200705211910.png)

3.放在`Authorization`请求首部

> Authorization首部说明
>
> Authorization首部是由客户端发送，以向服务器回应自己的身份验证信息，客户端在收到服务器的401 Authentication Required响应之后，需要在请求中包含该首部。
>
> 基本用法：Authorization: <authentication-scheme> <authentication-param>

在传输时，`Authorization`首部的`authentication-scheme`需要设置为`Bearer`，请求示例：

```json
GET /resource HTTP/1.1
Host: server.example.com
Authorization: Bearer mF_9.B5f-4.1JqM
```

###### 实际应用遇到的常见问题

当token未到期，用户却注销账号后，怎么处理这个token?

- 服务器保存这个被注销的token,以便下次收到使用这个仍在有效期内的 Token 时判其无效;
- 若是前端可以控制，前端一旦注销成功，丢弃本地保存（cookie或者localStorage）；
- 可以使用refresh token。

refresh token 的流程：

![image-20200706201200545](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200706201205.png)

![image-20200706201238285](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200706201240.png)

### 4.基于JWT的Token认证机制实现

> JWT规定了Token的具体实现方式

##### 4.1概念

WT 的原理是，服务器认证以后，生成一个 JSON 对象，发回给用户。用户与服务端通信的时候，都要发回这个 JSON 对象。服务器完全只靠这个对象认定用户身份。为了防止用户篡改数据，服务器在生成这个对象的时候，会加上签名。

##### 4.2JWT的结构

- Header（头部）
- Payload（负载）
- Signature（签名）

###### Header（头部）

Header 部分是一个 JSON 对象，描述 JWT 的元数据，通常是下面的样子:

```js
{
  "alg": "HS256",	//alg属性表示签名的算法（algorithm），默认是 HMAC SHA256（写成 HS256）
  "typ": "JWT"		//typ属性表示这个令牌（token）的类型（type），JWT 令牌统一写为JWT
}
```

最后，将上面的 JSON 对象使用 Base64URL 算法转成字符串。

###### Payload（负载）

Payload 部分也是一个 JSON 对象，用来存放实际需要传递的数据。JWT 规定了7个官方字段，供选用。

- iss (issuer)：签发人
- exp (expiration time)：过期时间
- sub (subject)：主题
- aud (audience)：受众
- nbf (Not Before)：生效时间
- iat (Issued At)：签发时间
- jti (JWT ID)：编号

注意，JWT 默认是不加密的，任何人都可以读到，所以不要把秘密信息放在这个部分。

这个 JSON 对象也要使用 Base64URL 算法转成字符串。

###### Signature（签名）

Signature 部分是对前两部分的签名，防止数据篡改。

首先，需要指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照下面的公式产生签名。

```js
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

算出签名以后，把 Header、Payload、Signature 三个部分拼成一个字符串，每个部分之间用"点"（`.`）分隔，就可以返回给用户。

##### 4.3认证过程

参考token认证的过程，3.2节

##### 4.4特点

- JWT 默认是不加密，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次。
- JWT 不加密的情况下，不能将秘密数据写入 JWT。
- JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数。
- JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。
- JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。
- 为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输。

### 5.基于OAuth(开放授权)认证（主要是规定授权流程）

##### 5.1概念

OAuth（Open Authorization）是一个开放标准，允许用户授权第三方网站访问他们存储在另外的服务提供者上的信息，而不需要将用户名和密码提供给第三方网站或分享他们数据的所有内容，为了保护用户数据的安全和隐私，第三方网站访问用户数据前都需要显式的向用户征求授权。我们常见的提供OAuth认证服务的厂商有支付宝，QQ,微信。

##### 5.2认证过程

GitHub第三方登录示例：

![image-20200706212225503](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200706212228.png)

- 在GitHub中备案第三方应用，拿到属于它的客户端ID和客户端密钥。在`github-settings-developer settings`中创建一个OAuth App。并填写相关内容。填写完成后Github会给你一个客户端ID和客户端密钥。

![image-20200706212257560](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200706212259.png)

- 此时在你的第三方网站就可以提供一个Github登录链接，用户点击该链接后会跳转到Github。这一步拿着客户端ID向Github请求授权码code。
- 用户跳转到Github，输入Github的用户名密码，表示用户同意使用Github身份登录第三方网站。此时就会带着授权码code跳回第三方网站。跳回的地址在创建该OAuth时已经设置好了。
- 第三方网站收到授权码，就可以拿着授权码、客户端ID和客户端密钥去向Github请求access_token令牌。
- Github收到请求，向第三方网站颁发令牌。
- 第三方网站收到令牌，就可以暂时拥有Github一些请求的权限，比如说拿到用户信息，拿到这个用户信息之后就可以构建自己第三方网站的token，做相关的鉴权操作。

##### 5.3优点

- 客户端不接触用户密码，服务端更易集中保护
- 广泛传播并被持续采用
- 支持短寿命和封装的Token.
- 资源服务器和授权服务器解耦
- 集中式授权，简化客户端
- HTTP/JSON友好，易于请求和传递Token.
- 考虑多种客户端架构场景
- 客户端可以具有不同的信任级别
- 

参考资料：

http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html

https://juejin.im/post/5d67662ee51d45621655353f#heading-11