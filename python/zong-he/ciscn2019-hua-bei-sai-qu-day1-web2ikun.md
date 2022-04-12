# \[CISCN2019 华北赛区 Day1 Web2]ikun

## \[CISCN2019 华北赛区 Day1 Web2]ikun

## 考点

* JWT
* tornado审计
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

![](<../../.gitbook/assets/image (12).png>)

```
>>> s='\\u8fd9\\u7f51\\u7ad9\\u4e0d\\u4ec5\\u53ef\\u4ee5\\u4ee5\\u8585\\u7f8a\\u6bdb\\uff0c\\u6211\\u8fd8\\u7559\\u4e86\\u4e2a\\u540e\\u95e8\\uff0c\\u5c31\\u85cf\\u5728\\u006c\\u0076\\u0036\\u91cc'
>>> s.encode('utf8').decode('unicode_escape')
'这网站不仅可以以薅羊毛，我还留了个后门，就藏在lv6里'
```

然后找lv6在哪儿，写爬虫爆破



