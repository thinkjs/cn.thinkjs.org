## Context / 上下文

Context 是 Koa 中处理用户请求中的一个对象，贯穿整个请求生命周期。一般在 middleware、controller、logic 中使用，简称为 `ctx`。

```js
// 在 middleware 中使用 ctx 对象
module.exports = options => {
  // 调用时 ctx 会作为第一个参数传递进来
  return (ctx, next) => {
    ...
  }
}
```

```js
// 在 controller 中使用 ctx 对象
module.exports = class extends think.Controller {
  indexAction() {
    // controller 中 ctx 作为类的属性存在，属性名为 ctx
    // controller 实例化时会自动把 ctx 传递进来
    const ip = this.ctx.ip;
  }
}
```

框架里继承了该对象，并通过 Extend 机制扩展了很多非常有用的属性和方法。

### Koa 内置 API

#### ctx.req

Node 的 [request](https://nodejs.org/api/http.html#http_class_http_incomingmessage) 对象。

#### ctx.res

Node 的 [response](https://nodejs.org/api/http.html#http_class_http_serverresponse) 对象。

**不支持** 绕开 Koa 对 response 的处理。 避免使用如下 node 属性:

- `res.statusCode`
- `res.writeHead()`
- `res.write()`
- `res.end()`

#### ctx.request

Koa 的 [Request](http://koajs.com/#request) 对象。

#### ctx.response

Koa 的 [Response](http://koajs.com/#response) 对象。

#### ctx.state

在中间件之间传递信息以及将信息发送给模板时，推荐的命名空间。避免直接在 ctx 上加属性，这样可能会覆盖掉已有的属性，导致出现奇怪的问题。

```js
ctx.state.user = await User.find(id);
```

这样后续在 controller 里可以通过 `this.ctx.state.user` 来获取对应的值。

```js
module.exports = class extends think.Controller {
  indexAction() {
    const user = this.ctx.state.user;
  }
}
```

#### ctx.app

应用实例引用，等同于 `think.app`。

#### ~~ctx.cookies.get(name, [options])~~

获取 cookie，不建议使用，推荐 [ctx.cookie(name)](#toc-a67)

#### ~~ctx.cookies.set(name, value, [options])~~

设置 cookie，不建议使用，推荐 [ctx.cookie(name, value, options)](#toc-a67)

#### ctx.throw([msg], [status], [properties])

辅助方法，抛出包含 `.status` 属性的错误，默认为 `500`。该方法让 Koa 能够根据实际情况响应。并且支持如下组合：

```js
ctx.throw(403)
ctx.throw('name required', 400)
ctx.throw(400, 'name required')
ctx.throw('something exploded')
```

例如 `this.throw('name required', 400)` 等价于：

```js
let err = new Error('name required');
err.status = 400;
throw err;
```

注意，这些是用户级别的错误，被标记了 `err.expose`，即这些消息可以用于响应客户端。显然，当你不想泄露失败细节的时候，不能用它来传递错误消息。

你可以传递一个 `properties` 对象，该对象会被合并到 error 中，有助于修改传递给上游中间件的极其友好的错误。

```js
ctx.throw(401, 'access_denied', { user: user });
ctx.throw('access_denied', { user: user });
```

Koa 使用 [http-errors](https://github.com/jshttp/http-errors) 创建错误对象。

#### ctx.assert(value, [msg], [status], [properties])

当 `!value`为真时抛出错误的辅助方法，与 `.throw()` 相似。类似于 node 的 [assert()](http://nodejs.org/api/assert.html) 方法。

```
this.assert(this.user, 401, 'User not found. Please login!');
```

Koa 使用 [http-assert](https://github.com/jshttp/http-assert) 实现断言.

#### ctx.respond

如不想使用 Koa 内置的 response 处理方法，可以设置 `ctx.respond = false;`。这时你可以自己设置原始的 `res` 对象来处理响应。

注意这样使用是 __不__被 Koa 支持的，因为这样有可能会破坏 Koa 的中间件和 Koa 本身提供的功能。这种用法只是作为一种 hack ，为那些想要在Koa中使用传统的`fn(req, res)`的方法和中间件的人提供一种便捷方式。

#### ctx.header

获取所有的 header 信息，等同于 `ctx.request.header`。

```js
const headers = ctx.headers;
```

#### ctx.headers

获取所有的 header 信息，等同于 `ctx.header`。

#### ctx.method

获取请求类型，大写。如：`GET`、`POST`、`DELETE`。

```js
const method = ctx.method;
```

#### ctx.method=

设置请求类型（并不会修改当前 HTTP 请求的真实类型），对有些中间件的场景下可能有用，如：`methodOverride()`。

```js
ctx.method = 'COMMAND';
```

#### ctx.url

获取请求地址。

#### ctx.url=

设置请求地址，对 URL rewrite 有用。

#### ctx.originalUrl

获取原始的请求 URL

#### ctx.origin

获取请求源 URL，包括协议和主机。

```js
ctx.origin
// => http://example.com
```

#### ctx.href

获取请求完整的 URL，包括协议、主机和 url。

```js
ctx.href
// => http://example.com/foo/bar?q=1
```

#### ctx.path

获取请求路径名。

#### ctx.path=

设置请求路径名，如果查询参数存在则保留。

#### ctx.query

获取解析后的查询参数，如果不存在查询参数则返回一个空对象。注意这个方法不支持嵌套参数的解析。

例如 `"color=blue&size=small"`

```js
{
  color: 'blue',
  size: 'small'
}
```
#### ctx.query=

通过给定的对象设置查询参数。注意这个赋值方法不支持嵌套对象。

```js
ctx.query = { next: '/login' }
```
#### ctx.querystring

获取原始查询字符串，不带问号。

#### ctx.querystring=

设置原始查询字符串。

#### ctx.search

获取原始查询字符串，带问号。

#### ctx.search=

设置原始查询字符串。

#### ctx.host

如果存在则获取主机（hostname:port）。当 app.proxy 为 true 时，使用 X-Forwarded-Host 的值，否则使用 Host 的值。

#### ctx.hostname

如果存在则获取主机名。当 app.proxy 为 true 时，使用 X-Forwarded-Host 的值，否则使用  Host 的值。

#### ctx.charset

如果存在则为请求的字符集，否则为 undefined：

```js
ctx.charset
// => "utf-8"
```
#### ctx.fresh

检查请求的缓存是否可用，即内容没有发生改变。此方法用来验证协商缓存 `If-None-Match / ETag`、`If-Modified-Since` 和 `Last-Modified`。此方法应该在设置了以上一个或多个响应头部的时候调用。

```js
// freshness check requires status 20x or 304
ctx.status = 200;
ctx.set('ETag', '123');

// cache is ok
if (ctx.fresh) {
  ctx.status = 304;
  return;
}

// cache is stale
// fetch new data
ctx.body = await db.find('something');
```
#### ctx.stale

和 ctx.fresh 相反。

#### ctx.socket

获取请求的套接字实体。

#### ctx.protocol

获取请求的协议类型，值为 `https` 或者 `http`，当 `app.proxy` 配置为 true 值支持从 `X-Forwarded-Proto` header 里获取。

具体的判断策略为：如果 `req.socket.encrypted` 为真，那么直接返回 `https`，否则如果配置了 `app.proxy` 为 true，那么从 `X-Forwarded-Proto` header 里获取，默认值为 `http`。

这么做是因为有时候并不会让 Node.js 直接对外提供服务，而是在前面用 web server（如：nginx）做反向代理，由 web server 来提供 HTTP(S) 服务，web server 与 Node.js 之间始终用 HTTP 交互。

这时候 Node.js 拿到的协议始终都是 `http`，真实的协议只有 web server 知道，所以要让 Node.js 拿到真实的协议时，就需要 web server 与 Node.js 定义特殊的字段来获取，推荐的自定义 header 为 `X-Forwarded-Proto`。为了安全性，只有设置了 `app.proxy` 为 true 是才会这样获取（`production.js` 里默认配置了为 true）。

```sh
ssl on;
# SSL certificate
ssl_certificate /usr/local/nginx/ssl/domain.crt;
ssl_certificate_key /usr/local/nginx/ssl/domain.key;

location = /index.js {
  proxy_http_version 1.1;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header Host $http_host;
  proxy_set_header X-Forwarded-Proto "https"; # 这里告知 Node.js 当前协议是 https
  proxy_set_header X-NginX-Proxy true;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "upgrade";
  proxy_pass http://127.0.0.1:$node_port$request_uri;
  proxy_redirect off;
}
```

#### ctx.secure

ctx.protocol == 'https' 的包装方法，检查该请求是否使用TLS隧道。

#### ctx.ip

获取客户端请求的地址。当 app.proxy 为 true 时使用 X-Forwarded-For 的值。

#### ctx.ips

当 X-Forwarded-For 存在并启用 app.proxy 时，返回回一个ip数组，这个数组是按上游到下游的顺序排序的。如果禁用则返回空数组。


#### ctx.subdomains

返回子域名数组。

子域名是主机主域名之前以点符号进行分割的部分。一个应用的默认的域名是由主机后面的两部分组成的。通过 app.subdomainOffset 可以改变默认设置。

一个简单的例子，假如有一个域名是 `tobi.ferrets.example.com`, 如果 app.subdomainOffset 没有设置，ctx.subdomains 的值为 ["ferrets", "tobi"]。如果 app.subdomainOffset 设置为3，ctx.subdomains 的值为 ["tobi"]。

#### ctx.is(...types)

检查此次请求头部`Content-Typ` 字段是否包含给定的 mime 类型。如果没有请求体，则返回 null。如果没有内容类型或者没有匹配到给定的 mime 类型则返回 false。如果匹配到了就返回相应的 content-type。

```js
// With Content-Type: text/html; charset=utf-8
ctx.is('html'); // => 'html'
ctx.is('text/html'); // => 'text/html'
ctx.is('text/*', 'text/html'); // => 'text/html'

// When Content-Type is application/json
ctx.is('json', 'urlencoded'); // => 'json'
ctx.is('application/json'); // => 'application/json'
ctx.is('html', 'application/*'); // => 'application/json'

ctx.is('html'); // => false
```

一个例子，如果你想让一个路由只接受图片，可以这样编码：

```js
if (ctx.is('image/*')) {
  // process
} else {
  ctx.throw(415, 'images only!');
}
```
#### ctx.accepts(types)

检查是否支持给定的类型，如果支持则返回优先级最高的的类型，否则返回false。类型的值可能是一个或者多个 mime 类型字符串，例如："application/json"、文件扩展名为 "json" 或者一个数组 ["json", "html", "text/plain"]。

```js
// Accept: text/html
ctx.accepts('html');
// => "html"

// Accept: text/*, application/json
ctx.accepts('html');
// => "html"
ctx.accepts('text/html');
// => "text/html"
ctx.accepts('json', 'text');
// => "json"
ctx.accepts('application/json');
// => "application/json"

// Accept: text/*, application/json
ctx.accepts('image/png');
ctx.accepts('png');
// => false

// Accept: text/*;q=.5, application/json
ctx.accepts(['html', 'json']);
ctx.accepts('html', 'json');
// => "json"

// No Accept header
ctx.accepts('html', 'json');
// => "html"
ctx.accepts('json', 'html');
// => "json"
```

可以多次调用 ctx.accepts() 方法，或者使用分支语句：

```js
switch (ctx.accepts('json', 'html', 'text')) {
  case 'json': break;
  case 'html': break;
  case 'text': break;
  default: ctx.throw(406, 'json, html, or text only');
}
```
#### ctx.acceptsEncodings(encodings)

检查是否支持编码，如果支持返回优先级最高的编码，否则返回 false。注意你应该将 identity 作为编码之一。

```js
// Accept-Encoding: gzip
ctx.acceptsEncodings('gzip', 'deflate', 'identity');
// => "gzip"

ctx.acceptsEncodings(['gzip', 'deflate', 'identity']);
// => "gzip"
```

如果没有给参数则返回所有支持的编码数组：

```js
// Accept-Encoding: gzip, deflate
ctx.acceptsEncodings();
// => ["gzip", "deflate", "identity"]
```
注意：如果客户端明确指定 `identity;q=0`，就不支持了 identity 编码（实际上就是不编码），此时这个方法将返回 false，你需要处理这个极端情况。

#### ctx.acceptsCharsets(charsets)

检查是否支持字符集，如果支持则返回优先级最高的字符集，否则返回 false。

```js
// Accept-Charset: utf-8, iso-8859-1;q=0.2, utf-7;q=0.5
ctx.acceptsCharsets('utf-8', 'utf-7');
// => "utf-8"

ctx.acceptsCharsets(['utf-7', 'utf-8']);
// => "utf-8"
```

如果没有参数则返回所有支持字符集的数组，按优先级排序。

```js
// Accept-Charset: utf-8, iso-8859-1;q=0.2, utf-7;q=0.5
ctx.acceptsCharsets();
// => ["utf-8", "utf-7", "iso-8859-1"]
```
#### ctx.acceptsLanguages(langs)

检查是否支持语言，如果支持返回优先级最高的语言，否则返回 false。

```js
// Accept-Language: en;q=0.8, es, pt
ctx.acceptsLanguages('es', 'en');
// => "es"

ctx.acceptsLanguages(['en', 'es']);
// => "es"
```

如果没有参数则返回所有支持语言的数组，按优先级排序。

```js
// Accept-Language: en;q=0.8, es, pt
ctx.acceptsLanguages();
// => ["es", "pt", "en"]
```

#### ctx.get(field)

返回请求头部指定字段的值。

```js
const host = ctx.get('host');
```

#### ctx.body

获取响应体。

#### ctx.body=

设置响应体，支持以下几种：

* 字符串

  Content-Type 默认为 text/html 或者 text/plain，这两种的字符集都是 utf-8，同时会设置 Content-Length 字段。

* Buffer

  Content-Type 默认为 application/octet-stream，也会设置 Content-Length 字段。

* 管道流

  Content-Type 默认为 application/octet-stream。

  当响应体为流时，会自动添加一个 onerror 的异常事件监听器来捕获任何异常。另外，当请求被关闭时（甚至在这之前），这个流会被销毁掉。如果你不想要这两个特性，就不要将响应体设置为流。你不会希望在代理中设置响应体为HTTP流的，因为这样会破坏基础连接。

  更多内容请看：https://github.com/koajs/koa/pull/612。

  这个例子中添加了异常捕获并且不会自动销毁 HTTP 流：

  ```js
  const PassThrough = require('stream').PassThrough;

  app.use(function * (next) {
    ctx.body = someHTTPStream.on('error', ctx.onerror).pipe(PassThrough());
  });
  ```
  
* json字符串化的对象或数组

  Content-Type 默认为 application/json。包括文本对象和数组。

* 设置为null, 没有响应内容。

  如果没有设置 ctx.status，Koa 自动将响应状态码设置为 200 或者 204。

#### ctx.status

获取响应状态码，Koa 的 `response.status` 默认设置为404，而 node 中 `res.statusCode` 默认设置为 200。

#### ctx.status=

通过数字代码设置响应状态：

* 100 "continue"
* 101 "switching protocols"
* 102 "processing"
* 200 "ok"
* 201 "created"
* 202 "accepted"
* 203 "non-authoritative information"
* 204 "no content"
* 205 "reset content"
* 206 "partial content"
* 207 "multi-status"
* 208 "already reported"
* 226 "im used"
* 300 "multiple choices"
* 301 "moved permanently"
* 302 "found"
* 303 "see other"
* 304 "not modified"
* 305 "use proxy"
* 307 "temporary redirect"
* 308 "permanent redirect"
* 400 "bad request"
* 401 "unauthorized"
* 402 "payment required"
* 403 "forbidden"
* 404 "not found"
* 405 "method not allowed"
* 406 "not acceptable"
* 407 "proxy authentication required"
* 408 "request timeout"
* 409 "conflict"
* 410 "gone"
* 411 "length required"
* 412 "precondition failed"
* 413 "payload too large"
* 414 "uri too long"
* 415 "unsupported media type"
* 416 "range not satisfiable"
* 417 "expectation failed"
* 422 "unprocessable entity"
* 423 "locked"
* 424 "failed dependency"
* 426 "upgrade required"
* 428 "precondition required"
* 429 "too many requests"
* 431 "request header fields too large"
* 500 "internal server error"
* 501 "not implemented"
* 502 "bad gateway"
* 503 "service unavailable"
* 504 "gateway timeout"
* 505 "http version not supported"
* 506 "variant also negotiates"
* 507 "insufficient storage"
* 508 "loop detected"
* 510 "not extended"
* 511 "network authentication required"

注意：不要担心记不住这些字符串，如果你有字符错误会将会抛出一个异常并显示这个列表，能够快速纠正你的错误。

#### ctx.message

获取响应的状态消息。response.message 默认和 response.status 相关。

#### ctx.message=

设置响应状态消息。

#### ctx.length=

设置响应头部 Content-Length 的值。

#### ctx.length

如果存在获取响应头部 Content-Length 的长度，或者从 ctx.body 中计算出来，或者是undefined。

#### ctx.type

获取响应头部 Content-Type 的值，不包括字符集等参数。

```js
const ct = ctx.type;
// => "image/png"
```

#### ctx.type=

通过 mime 字符串或者文件扩展名设置响应头部 Content-Type 的值。

```js
ctx.type = 'text/plain; charset=utf-8';
ctx.type = 'image/png';
ctx.type = '.png';
ctx.type = 'png';
```

注意：在适当的时候为你选择一个字符集，例如当 response.type = 'html' 时 默认为 "utf-8"，如果明确的指定完整的类型 response.type = 'text/html'，不设置字符集。

#### ctx.headerSent

检查响应头部是否已经发送给客户端。用于检查客户是否可能被错误通知。

#### ctx.redirect(url, [alt])

对指定的 url 进行 302跳转。

"back"是特殊字符串，当 Referrer 不存在或者使用'/'进行跳转时，用来提供 Referrer 支持。

```js
ctx.redirect('back');
ctx.redirect('back', '/index.html');
ctx.redirect('/login');
ctx.redirect('http://google.com');
```

如果想要修改默认状态码 302，可以在这个方法调用前后指定。要修改响应体，需要在这个方法调用之后指定。

```js
ctx.status = 301;
ctx.redirect('/cart');
ctx.body = 'Redirecting to shopping cart';
```
#### ctx.attachment([filename])

将响应头 Content-Disposition 设置为 "attachment" 通知客户端可以进行下载。可以选择设置下载时的文件名。

#### ctx.set(fields)

通过一个对象设置多个响应头部字段：

```js
ctx.set({
  'Etag': '1234',
  'Last-Modified': date
});
```

#### ctx.append(field, value)

通过给定的值追加响应头部。

```js
ctx.append('Link', '<http://127.0.0.1/>');
```

#### ctx.remove(field)

移除指定的响应头部。

#### ctx.lastModified=

使用UTC格式的时间字符串设置最后修改时间。你可以设置 Date 对象实例或者时间字符串。

```js
ctx.lastModified = new Date();
```
#### ctx.etag=

设置包含包装的响应的ETag。注意没有 response.etag 的取值方法。

```js
ctx.etag = crypto.createHash('md5').update(ctx.body).digest('hex');
```
### 框架扩展 API

#### ctx.module

路由解析后的模块名，单模块项目下该属性值始终为空。默认是通过 [think-router](https://github.com/thinkjs/think-router) 模块解析。

```js
module.exports = class extends think.Controller {
  __before() {
    // 获取解析后的 module
    // 由于 module 已经被 node 使用，所以这里建议变量名不要为 module
    const m = this.ctx.module;
  }
}
```

#### ctx.controller

路由解析后的控制器名，默认是通过 [think-router](https://github.com/thinkjs/think-router) 模块解析。

```js
module.exports = class extends think.Controller {
  __before() {
    // 获取解析后的 controller
    const controller = this.ctx.controller;
  }
}
```

#### ctx.action

路由解析后的操作名，默认是通过 [think-router](https://github.com/thinkjs/think-router) 模块解析。

```js
module.exports = class extends think.Controller {
  __before() {
    // 获取解析后的 action
    const action = this.ctx.action;
  }
}
```

#### ctx.userAgent

可以通过 `ctx.userAgent` 属性获取用户的 userAgent。

```js
const userAgent = ctx.userAgent;
if(userAgent.indexOf('spider')){
  ...
}
```

#### ctx.isGet

可以通过 `ctx.isGet` 判断当前请求类型是否是 `GET`。

```js
const isGet = ctx.isGet;
if(isGet){
  ...
}
```

#### ctx.isPost

可以通过 `ctx.isPost` 判断当前请求类型是否是 `POST`。

```js
const isPost = ctx.isPost;
if(isPost){
  ...
}
```

#### ctx.isCli

可以通过 `ctx.isCli` 判断当前请求类型是否是 `CLI`（命令行调用）。

```js
const isCli = ctx.isCli;
if(isCli){
  ...
}
```

#### ctx.referer(onlyHost)

* `onlyHost` {Boolean} 是否只返回 host
* `return` {String}

获取请求的 referer。

```
const referer1 = ctx.referer(); // http://www.thinkjs.org/doc.html
const referer2 = ctx.referer(true); // www.thinkjs.org
```

#### ctx.referrer(onlyHost)

等同于 `referer` 方法。

#### ctx.isMethod(method)

* `method` {String} 请求类型
* `return` {Boolean}

判断当前请求类型与 method 是否相同。

```js
const isPut = ctx.isMethod('PUT');
```

#### ctx.isAjax(method)

* `method` {String} 请求类型
* `return` {Boolean}

判断是否是 ajax 请求（通过 header 中 `x-requested-with` 值是否为 `XMLHttpRequest` 判断），如果执行了 method，那么也会判断请求类型是否一致。

```js
const isAjax = ctx.isAjax();
const isPostAjax = ctx.isAjax('POST');
```

#### ctx.isJsonp(callbackField)

* `callbackField` {String} callback 字段名，默认值为 `this.config('jsonpCallbackField')`
* `return` {Boolean}

判断是否是 jsonp 请求。

```js
const isJsonp = ctx.isJson('callback');
if(isJsonp){
  ctx.jsonp(data);
}
```

#### ctx.jsonp(data, callbackField)

* `data` {Mixed} 要输出的数据
* `callbackField` {String} callback 字段名，默认值为 `this.config('jsonpCallbackField')`
* `return` {Boolean} false

输出 jsonp 格式的数据，返回值为 false。可以通过配置 `jsonContentType` 指定返回的 `Content-Type`。

```js
ctx.jsonp({name: 'test'});

//output
jsonp111({
  name: 'test'
})
```

#### ctx.json(data)

* `data` {Mixed} 要输出的数据
* `return` {Boolean} false

输出 json 格式的数据，返回值为 false。可以通过配置 `jsonContentType` 指定返回的 `Content-Type`。

```js
ctx.json({name: 'test'});

//output
{
  name: 'test'
}
```

#### ctx.success(data, message)

* `data` {Mixed} 要输出的数据
* `message` {String} errmsg 字段的数据
* `return` {Boolean} false

输出带有 `errno` 和 `errmsg` 格式的数据。其中 `errno` 值为 0，`errmsg` 值为 message。

```js
{
  errno: 0,
  errmsg: '',
  data: ...
}
```

字段名 `errno` 和 `errmsg` 可以通过配置 `errnoField` 和 `errmsgField` 来修改。

#### ctx.fail(errno, errmsg, data)

* `errno` {Number} 错误号
* `errmsg` {String} 错误信息
* `data` {Mixed} 额外的错误数据
* `return` {Boolean} false

```js
{
  errno: 1000,
  errmsg: 'no permission',
  data: ''
}
```

字段名 `errno` 和 `errmsg` 可以通过配置 `errnoField` 和 `errmsgField` 来修改。

#### ctx.expires(time)

* `time` {Number} 缓存的时间，单位是毫秒。可以 `1s`，`1m` 这样的时间
* `return` {undefined}

设置 `Cache-Control` 和 `Expires` 缓存头。

```js
ctx.expires('1h'); //缓存一小时
```

#### ctx.config(name, value, m)

* `name` {Mixed} 配置名
* `value` {Mixed} 配置值
* `m` {String} 模块名，多模块项目下生效
* `return` {Mixed}

获取、设置配置项，内部调用 `think.config` 方法。

```js
ctx.config('name'); //获取配置
ctx.config('name', value); //设置配置值
ctx.config('name', undefined, 'admin'); //获取 admin 模块下配置值，多模块项目下生效
```

#### ctx.param(name, value)

* `name` {String} 参数名
* `value` {Mixed} 参数值
* `return` {Mixed}

获取、设置 URL 上的参数值。由于 `get`、`query` 等名称已经被 Koa 使用，所以这里只能使用 param。

```js
ctx.param('name'); //获取参数值，如果不存在则返回 undefined
ctx.param(); //获取所有的参数值，包含动态添加的参数
ctx.param('name1,name2'); //获取指定的多个参数值，中间用逗号隔开
ctx.param('name', value); //重新设置参数值
ctx.param({name: 'value', name2: 'value2'}); //重新设置多个参数值
```

#### ctx.post(name, value)

* `name` {String} 参数名
* `value` {Mixed} 参数值
* `return` {Mixed}

获取、设置 POST 数据。请求类型为 `POST`, `PUT`, `DELETE`, `PATCH`, `LINK`, `UNLINK` 时才可以通过 post 方法获取提交的数据。

```js
ctx.post('name'); //获取 POST 值，如果不存在则返回 undefined
ctx.post(); //获取所有的 POST 值，包含动态添加的数据
ctx.post('name1,name2'); //获取指定的多个 POST 值，中间用逗号隔开
ctx.post('name', value); //重新设置 POST 值
ctx.post({name: 'value', name2: 'value2'}); //重新设置多个 POST 值
```

有时候提交的数据是个复合的数据，这时候拿到的数据格式为下面的格式：

```
{ action: 'create',
  'data[0][username]': '',
  'data[0][nickname]': '',
  'data[0][password]': '' 
}
```

实际上我们希望 `data` 字段数据为数组，这时候可以使用 [think-qs](https://github.com/thinkjs/think-qs) 中间件来支持这种数据格式。

#### ctx.file(name, value)

* `name` {String} 参数名
* `value` {Mixed} 参数值
* `return` {Mixed}

获取、设置文件数据，文件会保存在临时目录下，为了安全，请求结束后会删除。如果需要使用对应的文件，可以通过 `fs.rename` 方法移动到其他地方。

```js
ctx.file('name'); //获取 FILE 值，如果不存在则返回 undefined
ctx.file(); //获取所有的 FILE 值，包含动态添加的数据
ctx.file('name', value); //重新设置 FILE 值
ctx.file({name: 'value', name2: 'value2'}); //重新设置多个 FILE 值
```

文件的数据格式为：

```js
{
  "size": 287313, //文件大小
  "path": "/var/folders/4j/g57qvmmd1lb_9h605w_d38_r0000gn/T/upload_fa6bf8c44179851f1cfec99544b4ef22", //临时存放的位置
  "name": "An Introduction to libuv.pdf", //文件名
  "type": "application/pdf", //类型
  "mtime": "2017-07-02T07:55:23.763Z" //最后修改时间
}
```

文件上传是通过 [think-payload](https://github.com/thinkjs/think-payload) 模块解析的，可以配置限制文件大小之类的参数。

```js
const fs = require('fs');
const path = require('path');
const rename = think.promisify(fs.rename, fs); // 通过 promisify 方法把 rename 方法包装成 Promise 接口
module.exports = class extends think.Controller {
  async indexAction(){
    const file = this.file('image');
    // 如果上传的是 png 格式的图片文件，则移动到其他目录
    if(file && file.type === 'image/png') {
      const filepath = path.join(think.ROOT_PATH, 'runtime/upload/a.png');
      think.mkdir(path.dirname(filepath));
      await rename(file.path, filepath)
    }
  }
}
```

如果上传多个同名的文件时（如：input 标签里设置了 multiple 属性），默认只会获取到一个。如果想获取多个的话，需要在 `src/config/middleware.js` 文件里 `payload` 中间件添加 `multiples` 属性，如：

```js
{
  handle: 'payload',
  options: {
    multiples: true
  }
}
```

此时通过 `this.file('name')` 获取的值为数组，里面包含了多个上传的文件。

#### ctx.cookie(name, value, options)

* `name` {String} Cookie 名
* `value` {mixed} Cookie 值
* `options` {Object} Cookie 配置项
* `return` {Mixed}

获取、设置 Cookie 值。

```js
ctx.cookie('name'); //获取 Cookie
ctx.cookie('name', value); //设置 Cookie
ctx.cookie(name, null); //删除 Cookie
ctx.cookie(name, null, {
  path: '/'
})
```

设置 Cookie 时，如果 value 的长度大于 4094，则触发 `cookieLimit` 事件，该事件可以通过 `think.app.on("cookieLimit")` 来捕获。

删除 Cookie 时，必须要设置 `domain`、`path` 等参数和设置的时候相同，否则因为浏览器的同源策略无法删除。

#### ctx.service(name, m, ...args)

* `name` {String} 要调用的 service 名称
* `m` {String} 模块名，多模块项目下生效
* `return` {Mixed}

获取 service，如果是类则实例化，否则直接返回。等同于 [think.service](/doc/3.0/think.html#toc-014)。

```js
// 获取 src/service/github.js 模块
const github = ctx.service('github');
```

#### ctx.download(filepath, filename)

* `filepath` {String} 下载文件的路径
* `filename` {String} 下载的文件名，如果没有则从 `filepath` 中获取。

下载文件，会通过 [content-disposition](https://github.com/jshttp/content-disposition) 模块设置 `Content-Disposition` 头信息。

```js
const filepath = path.join(think.ROOT_PATH, 'a.txt');
ctx.download(filepath);
```

如果文件名中含有中文导致乱码，那么可以自己手工指定 `Content-Disposition` 头信息，如：

```js
const userAgent = this.userAgent().toLowerCase();
let hfilename = '';
if (userAgent.indexOf('msie') >= 0 || userAgent.indexOf('chrome') >= 0) {
  hfilename = `=${encodeURIComponent(filename)}`;
} else if(userAgent.indexOf('firefox') >= 0) {
  hfilename = `*="utf8''${encodeURIComponent(filename)}"`;
} else {
  hfilename = `=${new Buffer(filename).toString('binary')}`;
}
ctx.set('Content-Disposition', `attachment; filename${hfilename}`)
ctx.download(filepath)
```
