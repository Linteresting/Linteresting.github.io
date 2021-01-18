# node.js学习笔记

Created: 2021/01/06
Created by: Max Lin
Tags: note

### **事件驱动**

```jsx
var events = require('events')
var eventsEmitter = new events.EventEmitter();

eventsEmitter.on('eventName', eventHandler)
eventEmitter.emit('eventName')
```

![https://www.runoob.com/wp-content/uploads/2015/09/event_loop.jpg](https://www.runoob.com/wp-content/uploads/2015/09/event_loop.jpg)

**回调**：`function foo1(name, age, callback) { }`

```jsx
var foo = function(name, age) {
    console.log(name)
    console.log(age)
}

var callback = function() {
    console.log('callback')
}

foo('lin', 9, callback()) // 参数是函数 回调函数
```

**Node应用程序如何工作？**

```jsx
var fs = require('fs')

fs.readFile('fileName', function(err, data) {
	if (err) {
		console.log(err.stack)
		return;
	}
	console.log(data.toString())
})
console.log('end')
```

**Node.js EventsEmitter**

`events` 模块只提供了一个对象： `events.EventEmitter`。`EventEmitter` 的核心就是事件触发与事件监听器功能的封装。

```jsx
var EventEmitter = require('event').EventEmitter
var event = new EventEmitter()

event.on('some_event', function() {
	// 事件触发
	console.log('some_event 触发')
})

// setTimeout(function () {
//     event.emit('some_event')
// }, 1000)

setInterval(function () {
    event.emit('some_event')
}, 1000)
```

当事件触发时，注册到这个事件的事件监听器被依次调用，事件参数作为回调函数参数传递。

```jsx
var EventEmitter = require('events'),eventEmitter
var emitter = new EventEmitter()

emitter.on('some_event', function (arg1, arg2) {
    console.log('listener1', arg1, arg2)
})

emitter.on('some_event', function (arg1, arg2) {
    console.log('listener2', arg1, arg2)
})

emitter.emit('some_event', 'arg1 参数1', 'arg2 参数二')
```

**EventEmitter 的属性**

[方法](https://www.notion.so/1b7a271693474a7a8d271e42cc484974)

[类方法](https://www.notion.so/96f16505f19b491788d3b9ce0b4466f8)

```jsx
var EventEmitter = require('events').EventEmitter
var emitter = new EventEmitter()

var listener1 = function() {
	console.log('listener1 监听器1执行')
}

var listener2 = function () {
	console.log('listener2 监听器2执行')
}

emitter.on('connection', listener1)
emitter.addListener('connection', listener2)

var eventListenerCount = emitter.listenerCount('connection')
console.log(eventListenerCount + '个监听器监听连接事件')

emitter.emit('connection')

emitter.removeListener('connection', listener1)
console.log('监听器1不再监听')

emitter.emit('connection')

eventListenerCount = emitter.listenerCount('connection')
console.log(eventListeners + " 个监听器监听连接事件。")

console.log('end')
```

**error 事件**

`EventEmitter` 定义了一个特殊的事件 error，它包含了错误的语义，我们在遇到 异常的时候通常会触发 error 事件。

当 error 被触发时，`EventEmitter` 规定如果没有响应的监听器，Node.js 会把它当作异常，退出程序并输出错误信息。

```jsx
var EventEmitter = require('events').EventEmitter
var emitter = new EventEmitter()

emitter.on('error', function () {
	console.log('错误！')
})

emitter.emit('error')
```

**继承 EventEmitter**

大多数时候我们不会直接使用 `EventEmitter`，而是在对象中继承它。包括 fs、net、 http 在内的，只要是支持事件响应的核心模块都是 EventEmitter 的子类。

首先，具有某个实体功能的对象实现事件符合语义，事件的监听和发生应该是一个对象的方法。

其次 JavaScript 的对象机制是基于原型的，支持部分多重继承，继承 `EventEmitter` 不会打乱对象原有的继承关系。

### Node.js Buffer

The `Buffer` class is within the global scope, making it unlikely that one would need to ever use `require('buffer').Buffer`.

但在处理像TCP流或文件流时，必须使用到二进制数据。因此在 Node.js中，定义了一个 Buffer 类，该类用来创建一个专门存放二进制数据的缓存区。

```jsx
var buf = Buffer.from('test', ascii)

console.log(buf.toString('hex'))
console.log(buf.toString('base64'))
console.log(buf.toString())
```

**创建 Buffer 类**

```jsx
// Creates a zero-filled Buffer of length 10.
const buf1 = Buffer.alloc(10)
console.log(buf1.toString('hex'))

// Creates a Buffer of length 10, filled with bytes which all have the value `1`.
const buf2 = Buffer.alloc(10, 1)

// 未初始化长度为10的buffer，比Buffer.alloc()快，但是返回的Buffer实例可能会包含老数据
// 所以需要用fill(), write(),或者其他方法覆盖Buffer的内容
const buf3 = Buffer.allocUnsafe(10)

const buf4 = Buffer.from([1, 2, 3])

// Creates a Buffer containing the bytes [1, 1, 1, 1] – the entries
// are all truncated using `(value & 255)` to fit into the range 0–255.
const buf5 = Buffer.from([257, 257.5, -255, '1']);

// Creates a Buffer containing the UTF-8-encoded bytes for the string 'tést':
// [0x74, 0xc3, 0xa9, 0x73, 0x74] (in hexadecimal notation)
// [116, 195, 169, 115, 116] (in decimal notation)
const buf6 = Buffer.from('tést');

// Creates a Buffer containing the Latin-1 bytes [0x74, 0xe9, 0x73, 0x74].
const buf7 = Buffer.from('tést', 'latin1');
```

**写入缓冲区**

```jsx
buf.write(string[, offset[, length]][, encoding])
```

**从缓冲区读取数据**

```jsx
buf.toString([encoding[, start[, end]]])
```

```jsx
buf = Buffer.alloc(26)
for (var i = 0; i < 26; i++) {
	buf[i] = i + 97
}

console.log( buf.toString('ascii'));       // 输出: abcdefghijklmnopqrstuvwxyz
console.log( buf.toString('ascii',0,5));   //使用 'ascii' 编码, 并输出: abcde
console.log( buf.toString('utf8',0,5));    // 使用 'utf8' 编码, 并输出: abcde
console.log( buf.toString(undefined,0,5)); // 使用默认的 'utf8' 编码, 并输出: abcde
```

**将 Buffer 转换为 JSON 对象**

```jsx
const buf = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5]);
const json = JSON.stringify(buf);
```

**缓冲区合并**

```jsx
Buffer.concat(list[, totalLength])

var buffer1 = Buffer.from(('菜鸟教程'));
var buffer2 = Buffer.from(('www.runoob.com'));
var buffer3 = Buffer.concat([buffer1,buffer2]);
console.log("buffer3 内容: " + buffer3.toString());
```

**缓冲区比较**

```jsx
buf.compare(otherBuffer);

var buffer1 = Buffer.from('ABC');
var buffer2 = Buffer.from('ABCD');
var result = buffer1.compare(buffer2);

if(result < 0) {
   console.log(buffer1 + " 在 " + buffer2 + "之前");
}else if(result == 0){
   console.log(buffer1 + " 与 " + buffer2 + "相同");
}else {
   console.log(buffer1 + " 在 " + buffer2 + "之后");
}
```

**拷贝缓冲区**

```jsx
buf.copy(targetBuffer[, targetStart[, sourceStart[, sourceEnd]]])

var buf1 = Buffer.from('abcdefghijkl');
var buf2 = Buffer.from('RUNOOB');

//将 buf2 插入到 buf1 指定位置上
buf2.copy(buf1, 2);

console.log(buf1.toString());
```

**缓冲区裁剪**

```jsx
buf.slice([start[, end]])

var buffer1 = Buffer.from('runoob');
// 剪切缓冲区
var buffer2 = buffer1.slice(0,2);
console.log("buffer2 content: " + buffer2.toString());
```

**缓冲区长度**

```jsx
var buffer = Buffer.from('www.runoob.com');
//  缓冲区长度
console.log("buffer length: " + buffer.length);
```

### **Node.js Stream**

Node.js，`Stream` 有四种流类型：

- **Readable** - 可读操作。
- **Writable** - 可写操作。
- **Duplex** - 可读可写操作.
- **Transform** - 操作被写入数据，然后读出结果。

所有的 `Stream` 对象都是 `EventEmitter` 的实例。常用的事件有：

- **data** - 当有数据可读时触发。
- **end** - 没有更多的数据可读时触发。
- **error** - 在接收和写入过程中发生错误时触发。
- **finish** - 所有数据已被写入到底层系统时触发。

**从流中读取数据**

```jsx
var fs = require('fs')
var data = ''

var readerStream = fs.createReadStream('input.txt');
readerStream.setEncoding('UTF8')

readerStream.on('data', (chunk) => {
	data += chunk
})

readerStream.on('end', () => {
	console.log(data)
})

readerStream.on('error', (err) => {
	console.log(err.stack)
})

console.log('end')
```

**写入流**

```jsx
var fs = require('fs')
var data = 'abcdefghijklmnopqrstuvwxyz'

var writeStream = fs.createWriteStream('output.txt')
writeStream.write(data, 'UTF-8')
writeStream.end()

writeStream.on('finish', () => {
	console.log('写入完成！')
})

writeStream.on('error', (err) => {
	console.log(err.stack)
})

console.log('程序执行完成')
```

**管道流**

```jsx
var fs = require('fs')

var readerStream = fs.createReadStream('input.txt')

var writeStream = fs.createWriteStream('output.txt')

readerStream.pipe(writeStream)

console.log('end')
```

**链式流**

压缩

```jsx
var fs = require('fs')
var zlib = require('zlib')

fs.createReadStream('input.txt')
	.pipe(zlib.createGzip())
	.pipe(fs.createWriteStream('input.txt.gz'))

console.log('end')
```

解压

```jsx
var fs = require('fs')
var zlib = require('zlib')

fs.createReadStream('input.txt.gz')
	.pipe(zlib.createGunzip())
	.pipe(fs.createWriteStream('input.txt'))

console.log('end')
```

### Node.js 模块系统

一个 Node.js 文件就是一个模块，这个文件可能是JavaScript 代码、JSON 或者编译过的C/C++ 扩展。

`Hello`模块：

```jsx
function Hello() {
	var name;
	this.setName = function(thyName) {
		name = thyName
	}
	this.sayHey = function() {
		console.log('hello ' + name)
	}
}

module.exports = Hello
```

引用：

```jsx
var Hello = require('./hello')

hello = new Hello()

hello.setName = 'test'

hello.sayHey()
```

`require` 加载过程：

![https://www.runoob.com/wp-content/uploads/2014/03/nodejs-require.jpg](https://www.runoob.com/wp-content/uploads/2014/03/nodejs-require.jpg)

**从文件模块缓存中加载**

尽管原生模块与文件模块的优先级不同，但是都会优先从文件模块的缓存中加载已经存在的模块。

**从原生模块加载**

原生模块的优先级仅次于文件模块缓存的优先级。require 方法在解析文件名之后，优先检查模块是否在原生模块列表中。以http模块为例，尽管在目录下存在一个 http/http.js/http.node/http.json 文件，require("http") 都不会从这些文件中加载，而是从原生模块中加载。

原生模块也有一个缓存区，同样也是优先从缓存区加载。如果缓存区没有被加载过，则调用原生模块的加载方式进行加载和执行。

**从文件加载**

当文件模块缓存中不存在，而且不是原生模块的时候，Node.js 会解析 require 方法传入的参数，并从文件系统中加载实际的文件，

require方法接受以下几种参数的传递：

- http、fs、path等，原生模块。
- ./mod或../mod，相对路径的文件模块。
- /pathtomodule/mod，绝对路径的文件模块。
- mod，非原生模块的文件模块。

### Node.js 函数

在 JavaScript中，一个函数可以作为另一个函数的参数。我们可以先定义一个函数，然后传递，也可以在传递参数的地方直接定义函数。

```jsx
var http = require('http')

http.createServer(function(request, response) {
	response.writeHead(200, {"Conte-Type": "text/plain"})
	response.write("hello world")
	response.end()
}).listen(8888)
```

向 createServer 函数传递了一个匿名函数。

### Node.js 路由

router.js

```jsx
function route(pathname) {
    console.log('About to route a request for ' + pathname)
    if (pathname == '/test') {
        return 'test get'
    }
}

exports.route = route
```

server.js

```jsx
var http = require('http')
var url = require('url')

function start(route) {
    http.createServer((request, response) => {
        var pathName = url.parse(request.url).pathname
        var res = route(pathName)
        response.writeHead(200, {"Content-Type": "text/plain"});
        response.write("Hello World " + (res === undefined ? '' : res));
        response.end();
    }).listen(8888)
}

exports.start = start
```

index.js

```jsx
var server = require('./server')
var route = require('./router')

server.start(route.route)
```

### Node.js GET/POST请求

**GET**

```jsx
var http = require('http');
var url = require('url');
var util = require('util');
 
http.createServer(function(req, res){
    res.writeHead(200, {'Content-Type': 'text/plain'});
 
    // 解析 url 参数
    var params = url.parse(req.url, true).query;
    res.write("网站名：" + params.name);
    res.write("\n");
    res.write("网站 URL：" + params.url);
    res.end();
 
}).listen(3000);
```

**POST**

```jsx
var http = require('http');
const { builtinModules } = require('module');
var querystring = require('querystring')

var postHTML = '<html><head><meta charset="utf-8"><title>菜鸟教程 Node.js 实例</title></head>' +
        '<body>' +
        '<form method="post">' +
        '网站名： <input name="name"><br>' +
        '网站 URL： <input name="url"><br>' +
        '<input type="submit">' +
        '</form>' +
        '</body></html>';
http.createServer((req, res) => {
    var body = '';
    req.on('data', (chunk) => {
        body += chunk
    })
    req.on('end', () => {
        body = querystring.parse(body)
        res.writeHead(200, {'Content-Type': 'text/html; charset=utf8'});
        if (body.name && body.url) {
            res.write("网站名：" + body.name);
            res.write("<br>");
            res.write("网站 URL：" + body.url);
        } else {
            res.write(postHTML)
        }
        res.end()
    })
}).listen(3000)
```

### Node.js Express 框架

**Express 简介**

Express 是一个简洁而灵活的 node.js Web应用框架, 提供了一系列强大特性帮助你创建各种 Web 应用，和丰富的 HTTP 工具。

Express 框架核心特性：

- 可以设置中间件来响应 HTTP 请求。
- 定义了路由表用于执行不同的 HTTP 请求动作。
- 可以通过向模板传递参数来动态渲染 HTML 页面。

**安装 Express**

```bash
npm install express --save

# body-parser - node.js 中间件，用于处理 JSON, Raw, Text 和 URL 编码的数据。
npm install body-parser --save
# cookie-parser - 这就是一个解析Cookie的工具。通过req.cookies可以取到传过来的cookie，并把它们转成对象。
npm install cookie-parser --save
# multer - node.js 中间件，用于处理 enctype="multipart/form-data"（设置表单的MIME编码）的表单数据。
npm install multer --save
```

第一个Express框架实例

```jsx
//express_demo.js 文件
var express = require('express');
var app = express();
 
app.get('/', function (req, res) {
   res.send('Hello World');
})
 
var server = app.listen(8081, function () {
 
  var host = server.address().address
  var port = server.address().port
 
  console.log("应用实例，访问地址为 http://%s:%s", host, port)
})
```

**请求和响应**

```jsx
app.get('/', function (req, res) {
   // --
})
```

**Request 对象** - request 对象表示 HTTP 请求，包含了请求查询字符串，参数，内容，HTTP 头部等属性。常见属性有：

1. `req.app`：当callback为外部文件时，用`req.app`访问express的实例
2. `req.baseUrl`：获取路由当前安装的URL路径
3. `req.body / req.cookies`：获得「请求主体」/ Cookies
4. `req.fresh / req.stale`：判断请求是否还「新鲜」
5. `req.hostname / req.ip`：获取主机名和IP地址
6. `req.originalUrl`：获取原始请求URL
7. `req.params`：获取路由的parameters
8. `req.path`：获取请求路径
9. `req.protocol`：获取协议类型
10. `req.query`：获取URL的查询参数串
11. `req.route`：获取当前匹配的路由
12. `req.subdomains`：获取子域名
13. `req.accepts()`：检查可接受的请求的文档类型
14. `req.acceptsCharsets / req.acceptsEncodings / req.acceptsLanguages`：返回指定字符集的第一个可接受字符编码
15. `req.get()`：获取指定的HTTP请求头
16. `req.is()`：判断请求头Content-Type的MIME类型

**Response 对象** - response 对象表示 HTTP 响应，即在接收到请求时向客户端发送的 HTTP 响应数据。常见属性有：

1. `res.app`：同req.app一样
2. `res.append()`：追加指定HTTP头
3. `res.set()`在`res.append()`后将重置之前设置的头
4. `res.cookie(name，value [，option])`：设置Cookie
5. `opition: domain / expires / httpOnly / maxAge / path / secure / signed`
6. `res.clearCookie()`：清除Cookie
7. `res.download()`：传送指定路径的文件
8. `res.get()`：返回指定的HTTP头
9. `res.json()`：传送JSON响应
10. `res.jsonp()`：传送JSONP响应
11. `res.location()`：只设置响应的Location HTTP头，不设置状态码或者close response
12. `res.redirect()`：设置响应的Location HTTP头，并且设置状态码302
13. `res.render(view,[locals],callback)`：渲染一个view，同时向callback传递渲染后的字符串，如果在渲染过程中有错误发生next(err)将会被自动调用。callback将会被传入一个可能发生的错误以及渲染后的页面，这样就不会自动输出了。
14. `res.send()`：传送HTTP响应
15. `res.sendFile(path [，options] [，fn])`：传送指定路径的文件 -会自动根据文件extension设定Content-Type
16. `res.set()`：设置HTTP头，传入object可以一次设置多个头
17. `res.status()`：设置HTTP状态码
18. `res.type()`：设置Content-Type的MIME类型

**路由**

```jsx
var express = require('express')
var app = express()

app.get('/', (req, res) => {
    console.log('主页GET请求')
    res.send('Hello GET')
})

app.post('/', (req, res) => {
    console.log('主页POST请求')
    res.send('Hello POST')
})

app.get('/del_user', (req, res) => {
    console.log('del user')
    res.send('删除页面')
})

app.get('/list_user', (req, res) => {
    console.log('list_user')
    res.send('用户列表')
})

app.get('/ab*cd', (req, res) => {
    console.log("/ab*cd GET 请求");
    console.log('正则匹配')
    res.send('正则匹配')
})

var server = app.listen(8081, () => {
    var host = server.address().address
    var port = server.address().port

    console.log("应用实例，访问地址为 http://%s:%s", host, port)
})
```

**静态文件**

可以使用 `express.static` 中间件来设置静态文件路径。

```jsx
app.use('/public', express.static('public'));
```

```jsx
var express = require('express')
var app = new express()

app.use('/public', express.static('public'))

app.get('/', (req, res) => {
	console.log('主页get')
	res.send('主页')
})

var server = app.listen(8081, () => {
	var host = server.address().address
	var port = server.address().port

	console.log('应用实例，访问地址为http://%s:%s', host, port)
})
```

**GET 方法**

index.html

```html
<html>
<body>
<form action="http://127.0.0.1:8081/process_get" method="GET">
First Name: <input type="text" name="first_name">  <br>
Last Name: <input type="text" name="last_name">
<input type="submit" value="Submit">
</form>
</body>
</html>
```

```jsx
var express = require('express')
var app = new express()

app.use('/public', express.static('public'))

app.get('/', (req, res) => {
    res.sendFile(__dirname + '/public/' + 'index.html')
})

app.get('/process_get', (req, res) => {
    var response = {
        'first_name': req.query.first_name,
        'last_name': req.query.last_name
    }
    console.log(response)
    res.send(JSON.stringify(response))
})

var server = app.listen(8081, () => {
    var host = server.address().address
    var port = server.address().port

    console.log('应用实例，访问地址为 http://%s:%s', host, port)
})
```

**POST 方法**

```jsx
var express = require('express')
var app = new express()
var bodyParser = require('body-parser')

var urlencodeParse = bodyParser.urlencoded({extended: false})

app.use('/public', express.static('public'))

app.get('/index.html', (req, res) => {
    res.sendFile(__dirname + '/public/' + 'index.html')
})

app.post('/process_post', urlencodeParse, (req, res) => {
    var response = {
        'first_name': req.body.first_name,
        'last_name': req.body.last_name
    }

    console.log(response)
    res.end(JSON.stringify(response))
})

var server = app.listen(8081, () => {
    var host = server.address().address
    var port = server.address().port

    console.log('访问地址为:http://%s:%s', host, port)
})
```

```java
// 反转链表
ListNode reverse(ListNode head) {
	if (head.next == null) return head;
	ListNode last = reverse(head.next);
	head.next.next = head;
	head.next = null;
	return last;
}
```
