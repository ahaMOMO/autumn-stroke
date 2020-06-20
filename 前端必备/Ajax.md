# Ajax

### **1.** **实现一个原生ajax**

##### 1）创建XMLHttpRequest对象

```
var xhr = new XMLHttpRequest();
```

##### 2）设置请求参数并发送数据

```
xhr.open('get', 'aabb.php', true);//true``为异步
Xhr.send(data);
```

##### 3）设置回调函数-一个处理服务器响应的函数

##### 4）获取异步对象的readyState属性

该属性存有服务器响应的状态信息。每当readyState改变时，onreadystatechange函数就会被执行。

##### 5）判断响应报文的状态

若为200说明服务器正常运行并返回响应数据。

##### 6）读取响应数据

可以通过reponseTest属性来取回服务器返回来的数据。

```
xhr.onreadystatechange = function() {
    if(xhr.readyState==4) {
        if(xhr.status==200) {
        console.log(xhr.responseText);
        }
    }
}
```

### 2.XMLHttpRequest对象的常用方法

- abort():停止当前请求
- getAllResponseHeaders():把HTTP请求的所有响应首部作为键值对返回
- getResponseHeader("header"):返回指定首部的值
- open(method,url):建立对服务器的调用，还有3个可选参数，是否异步、用户名、密码
- send(content):向服务器发送请求
- setRequestHeader(header,value):把制定首部设置为提供的值

### 3.readyState状态码

- 0 － （未初始化）还没有调用send()方法
- 1 － （载入）已调用send()，正在发送请求，服务器连接已建立
- 2 － （载入完成）send()方法执行完成，已经接收到全部响应内容
- 3 － （交互）正在解析响应内容
- 4 － （完成）响应内容解析完成，可以在客户端调用了

### 4.ajax、fetch、axios的区别以及各自优缺点？

##### **1）**  **ajax**

Ajax是指一种创建交互式网页应用的网页开发技术，并且可以做到无需重新加载整个网页的情况下，能够更新部分网页，也叫作局部更新。使用ajax发送请求是依靠于一个对象，叫XmlHttpRequest对象，通过这个对象我们可以从服务器获取到数据，然后再渲染到我们的页面上。IE7以下通过ActiveXObject这个对象来创建。

**优点：**

1）局部更新，原生支持不需要任何插件

2）可以监听上传进度。

**缺点：**

1）嵌套回调，难以处理

2）这个基础上又JqueryAjax,缺点是需要引入整个jq文件，文件较大，成本过高不合理。

**2）**  **fetch**

Fetch是ajax非常好的一个替代品,解决了原生ajax的回调地狱的问题。Fetch能做到这一点，是因为Fetch API是基于Promise设计的。并且fetch调用非常简单，因为它是挂在BOM上的，属于全局的方法（是window的一个方法）。一定记住fetch不是ajax的进一步封装，而是原生js，没有使用XMLHttpRequest对象。

```
// Example POST method implementation:

postData('http://example.com/answer', {answer: 42})
  .then(data => console.log(data)) // JSON from `response.json()` call
  .catch(error => console.error(error))

function postData(url, data) {
  // Default options are marked with *
  return fetch(url, {
    body: JSON.stringify(data), // must match 'Content-Type' header
    cache: 'no-cache', // *default, no-cache, reload, force-cache, only-if-cached
    credentials: 'same-origin', // include, same-origin, *omit
    headers: {
      'user-agent': 'Mozilla/4.0 MDN Example',
      'content-type': 'application/json'
    },
    method: 'POST', // *GET, POST, PUT, DELETE, etc.
    mode: 'cors', // no-cors, cors, *same-origin
    redirect: 'follow', // manual, *follow, error
    referrer: 'no-referrer', // *client, no-referrer
  })
  .then(response => response.json()) // parses response to JSON
}
```

```
// fetch
fetch(url).then(response = > {
    if (response.ok) {
        response.json()
    }
}).then(data = > console.log(data)).
catch (err = > console.log(err))
```

可以结合async/await使之像同步代码一样：

```
async function test() {
    let response = await fetch(url);
    let data = await response.json();
    console.log(data)
}
```

**优点：**

1)更加底层，提供更丰富的API(request(可将要传送过去的url、header等封装成一个request对象)、 response（响应回来的对象，有header、ok等属性）)

2)解决回调地狱，使用起来更简洁。

**缺点：**

1）当接收到一个代表错误的 HTTP 状态码时，从 fetch() 返回的 Promise 不会被标记为 reject，即使响应的 HTTP 状态码是 404 或 500。相反，它会将 Promise 状态标记为 resolve （但是会将 resolve 的返回值的 ok 属性设置为 false ），仅当网络故障时或请求被阻止时，才会标记为 reject。

2)  `fetch()` 不会接受跨域 cookies；你也不能使用 `fetch()` 建立起跨域会话。其他网站的 `Set-Cookie` 头部字段将会被无视。

3)  默认情况下, fetch 不会从服务端发送或接收任何 cookies, 如果站点依赖于用户 session，则会导致未经认证的请求（要发送 cookies，必须设置 credentials :'include'选项）。

4）fetch不支持abort，不支持超时控制，使用setTimeout及Promise.reject的实现的超时控制并不能阻止请求过程继续在后台运行，造成了大量的浪费。

5）fetch没有办法原生监测请求的进度，而XHR可以。

6）所有的IE浏览器都不会支持 fetch()方法，需要使用ployfill。

##### **3）**  axios

axios开始受到更多的欢迎了。其实axios也是对原生XHR的一种封装，不过是Promise实现版本。
它是一个用于浏览器和 nodejs 的 HTTP 客户端，符合最新的ES规范。

```
 axios({
     method: 'post',
     url: '/abc/login',
     data: {
         userName: 'Lan',
         password: '123'
     }
})
.then(function (response) {
	console.log(response);
})
.catch(function (error) {
	console.log(error);
});

```

**优点：**

1）axios体积比较小，也没有上面fetch的各种问题。

axios 是一个基于Promise 用于浏览器和 nodejs 的 HTTP 客户端。它本身具有以下特征：

2）从浏览器中创建 XMLHttpRequest 

3）从 node.js 发出 http 请求 

4）支持 Promise API 

5）拦截请求和响应 

6）转换请求和响应数据 

7）取消请求 

8）自动转换JSON数据 

9）客户端支持防止CSRF/XSRF。