# Nodejs 学习笔记


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Nodejs 学习笔记](#nodejs-%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0)
  - [阻塞与非阻塞](#%E9%98%BB%E5%A1%9E%E4%B8%8E%E9%9D%9E%E9%98%BB%E5%A1%9E)
  - [处理事件](#%E5%A4%84%E7%90%86%E4%BA%8B%E4%BB%B6)
  - [流处理](#%E6%B5%81%E5%A4%84%E7%90%86)
  - [模块引用](#%E6%A8%A1%E5%9D%97%E5%BC%95%E7%94%A8)
  - [继承](#%E7%BB%A7%E6%89%BF)
  - [启动一个 html 服务](#%E5%90%AF%E5%8A%A8%E4%B8%80%E4%B8%AA-html-%E6%9C%8D%E5%8A%A1)
  - [Express](#express)
    - [GET](#get)
    - [POST](#post)
  - [多进程](#%E5%A4%9A%E8%BF%9B%E7%A8%8B)

<!-- /code_chunk_output -->



## 阻塞与非阻塞

```javascript
//Nodejs的阻塞与非阻塞

//Read files
var fs = require("fs");
var data = fs.readFileSync("./input.txt");
console.log(data.toString());
console.log("=========");
console.log("开始非阻塞");
fs.readFile("./input.txt", (err, res) => {
  console.log(res.toString());
});
console.log("非阻塞的程序");
```

<mark>fs 模块</mark>中的方法均有异步和同步版本，例如读取文件内容的函数**有异步的 fs.readFile()** 和**同步的 fs.readFileSync()**

```
> node main.js
node js 测试
=========
开始非阻塞
非阻塞的程序
node js 测试
```

## 处理事件

`on`为指定事件注册一个==监听器==，接受一个==字符串 event== 和一个回调函数。
`removeListener(event, listener)`移除指定事件的某个监听器，监听器必须是该事件已经注册过的监听器。

它接受两个参数，第一个是==事件名称==，第二个是==回调函数名称==。

```javascript
//处理事件
var event = require("events");
var eventEmitter = new event.EventEmitter();
var eventfunc = () => {
  console.log("执行处理事件");
};
eventEmitter.on("eventName", eventfunc);
eventEmitter.emit("eventName");
console.log("执行完毕");
```

## 流处理

```javascript
var zlib = require("zlib");

fs.createReadStream("./input.txt")
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream("./input.txt.gz"));

console.log("文件压缩完成");
```

## 模块引用

- 创建一个 js 文件./func.js, 使用`module.exports`来导出

```javascript
function Func() {
  this.func1 = () => {
    console.log("This is Function1");
  };

  this.func2 = () => {
    console.log("This is Function2");
  };
}

module.exports = Func;
```

- 导入到 main.js 中

```javascript
//引入模块
var functionImport = require("./func");
fi = new functionImport();
fi.func1();
```

<font color=#FF0000>不建议同时使用 exports 和 module.exports</font>

## 继承

`util.inherits(Sub, Base);`
Sub 为子对象， Base 为初始父对象

`util.inspect`转换为字符串

```javascript
var util = require("util");
console.log(util.inspect(fi));
```

`util.isArray(object)` 是否为数组

## 启动一个 html 服务

```javascript
var http = require("http");
var fs = require("fs");
var url = require("url");

http
  .createServer((req, res) => {
    var pathname = url.parse(req.url).pathname;

    fs.readFile(pathname.substr(1), (err, data) => {
      if (!err) {
        res.writeHead(200, { "Content-Type": "text/html" });
        res.write(data.toString());
      } else {
        res.writeHead(404, { "Content-Type": "text/html" });
      }
      res.end();
    });
  })
  .listen(9002);

console.log("Server running at http://127.0.0.1:9002");
```

## Express 

### GET

```javascript
var express = require("express");
var app = new express();

app.get("/", function(req, res) {
  console.log("主页 GET 请求");
  res.send("Hello GET");
});

app.listen(9002, () => {
  console.log("Server running at http://127.0.0.1:9002");
});
```

**通过 json 文件读取**

```javascript
var express = require("express");
var fs = require("fs");
var app = new express();

//listAll users
app.get("/listAll", function(req, res) {
  fs.readFile("./user.json", "utf8", (err, data) => {
    console.log("Get services right");
    res.send(data);
  });
});

app.listen(9002, () => {
  console.log("Service starting at http://127.0.0.1:9002");
});
```

user.json

```json
{
  "user1": {
    "id": 1,
    "name": "a",
    "pass": true
  },
  "user2": {
    "id": 2,
    "name": "b",
    "pass": false
  },
  "user3": {
    "id": 3,
    "name": "c",
    "pass": true
  }
}
```

### POST

- application/json
  可通过==Restlet Client==测试

```javascript
var express = require("express");
var fs = require("fs");
var app = new express();
var bodyParser = require("body-parser");

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));

//post users
app.post("/auth", (req, res, next) => {
  if (req.body.id == 1) {
    res.send("User has been locked");
  } else {
    res.send("OK");
  }
});

app.listen(9002, () => {
  console.log("Service starting at http://127.0.0.1:9002");
});
```

- form-data

```javascript
var multipart = require("connect-multiparty");
var multipartMiddleware = multipart();
app.post("/formdata", multipartMiddleware, function(req, res) {
  console.log(req.body);
  res.send("post successfully!");
});
```

## 多进程

`child_process.exec(command[, options], callback)`
使用==子进程执行命令==，缓存子进程的输出，并将子进程的输出以回调函数参数的形式返回。