[Fetch API](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch) 提供了一个 JavaScript 接口，用于访问和操纵 HTTP 管道的一些具体部分，例如请求和响应。它还提供了一个全局 [`fetch()`](https://developer.mozilla.org/zh-CN/docs/Web/API/GlobalFetch/fetch) 方法，该方法提供了一种简单，合理的方式来跨网络异步获取资源。fetch并不是度xmlHttpRequest的封装，而是浏览器提供的一个新的api接口。

基本上主流的浏览器都支持，ie8以下是不支持的。可以使用`self.fetch`检查浏览器是否支持fetch

请注意，`fetch` 规范与 `jQuery.ajax()` 主要有三种方式的不同：

- 当接收到一个代表错误的 HTTP 状态码时，从 `fetch()` 返回的 Promise **不会被标记为 reject，** 即使响应的 HTTP 状态码是 404 或 500。相反，它会将 Promise 状态标记为 resolve （但是会将 resolve 的返回值的 `ok` 属性设置为 false ），仅当网络故障时或请求被阻止时，才会标记为 reject。
- `fetch()` **不会接受跨域 cookies；**你也不能使用 `fetch()` 建立起跨域会话。其他网站的 `Set-Cookie` 头部字段将会被无视。
- `fetch` **不会发送 cookies**。除非你使用了*credentials* 的[初始化选项](https://developer.mozilla.org/zh-CN/docs/Web/API/WindowOrWorkerGlobalScope/fetch#Parameters)。

用户发送网络请求的方式有很多，如from表单，原生的xmlHttpRequest，jquery的ajax，axios等等

而这些功能个都是需要使用额外的包的，现在，fetch是可以直接使用的相比于ajax的使用要简单得多

如，发送请求

```js
fetch(url|Request,init).then(res=>{return res.json();})
```

请求的第一个参数可以是普通的url，也可以是一个文件的路径或者request对象`var myRequest = new Request('flowers.jpg');，当请求成功后将会返回一个Promise对象，并且调用res对象的的方法返回的也是一个promise对象

res中的部分方法

- [`arrayBuffer()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Body/arrayBuffer)
- [`blob()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Body/blob)
- [`json()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Body/json)
- [`text()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Body/text)
- [`formData()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Body/formData)

`res.json()`就是将返回的数据以json格式表示

init是一个可选对象

一个配置项对象，包括所有对请求的设置。可选的参数有：

- `method`: 请求使用的方法，如 `GET、``POST。`
- `headers`: 请求的头信息，形式为 [`Headers`](https://developer.mozilla.org/zh-CN/docs/Web/API/Headers) 的对象或包含 [`ByteString`](https://developer.mozilla.org/zh-CN/docs/Web/API/ByteString) 值的对象字面量。
- `body`: 请求的 body 信息：可能是一个 [`Blob`](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob)、[`BufferSource`](https://developer.mozilla.org/zh-CN/docs/Web/API/BufferSource)、[`FormData`](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData)、[`URLSearchParams`](https://developer.mozilla.org/zh-CN/docs/Web/API/URLSearchParams) 或者 [`USVString`](https://developer.mozilla.org/zh-CN/docs/Web/API/USVString) 对象。注意 GET 或 HEAD 方法的请求不能包含 body 信息。
- `mode`: 请求的模式，如 `cors、` `no-cors 或者` `same-origin。`
- `credentials`: 请求的 credentials，如 `omit、``same-origin 或者` `include。为了在当前域名内自动发送 cookie ， 必须提供这个选项， 从 Chrome 50 开始， 这个属性也可以接受` [`FederatedCredential`](https://developer.mozilla.org/zh-CN/docs/Web/API/FederatedCredential) 实例或是一个 [`PasswordCredential`](https://developer.mozilla.org/zh-CN/docs/Web/API/PasswordCredential) 实例。
- `cache`:  请求的 cache 模式: `default `、 `no-store 、` `reload 、` `no-cache 、` `force-cache `或者 `only-if-cached 。`
- `redirect`: 可用的 redirect 模式: `follow` (自动重定向), `error` (如果产生重定向将自动终止并且抛出一个错误), 或者 `manual` (手动处理重定向). 在Chrome中，Chrome 47之前的默认值是 follow，从 Chrome 47开始是 manual。
- `referrer`: 一个 [`USVString`](https://developer.mozilla.org/zh-CN/docs/Web/API/USVString) 可以是 `no-referrer、``client`或一个 URL。默认是 `client。`
- `referrerPolicy`: 指定了HTTP头部referer字段的值。可能为以下值之一： `no-referrer、` `no-referrer-when-downgrade、` `origin、` `origin-when-cross-origin、` `unsafe-url 。`
- `integrity`: 包括请求的  [subresource integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity) 值 （ 例如： `sha256-BpfBw7ivV8q2jLiT13fxDYAe2tJllusRSZ273h2nFSE=）。`

header对象

```js
myHeaders = new Headers({
  "Content-Type": "text/plain",
  "Content-Length": content.length.toString(),
  "X-Custom-Header": "ProcessThisImmediately",
});
//获取
myHeaders.get("Content-Length")
myHeaders.getAll("X-Custom-Header")
//设置
myHeaders.set("Content-Type", "text/html");
//添加
myHeaders.append("X-Custom-Header", "AnotherValue");
//删除
myHeaders.delete("X-Custom-Header");
```



[传统 Ajax 已死，Fetch 永生](https://segmentfault.com/a/1190000003810652)