# \[b01lers2020]Welcome to Earth

## \[b01lers2020]Welcome to Earth

## wp

直接访问发现从`/`跳转到了`/die`，抓包发现主页存在路径`/chase`

![](<../.gitbook/assets/image (3) (1).png>)

访问后又有其他路径，接着访问有`/shoot`目录，然后是`/door`目录

![](<../.gitbook/assets/image (5) (1).png>)

到这可以猜到大概意思是要不断访问出现的路径，其格式大概是这样`window.location='/shoot/'`

访问`/door`目录是给了一个js文件

![](<../.gitbook/assets/image (13).png>)

再访问js文件才能得到下一个目录，`/open`目录也是同理

![](<../.gitbook/assets/image (17) (1).png>)

用如下脚本，得到了第一阶段的结果

```
http://44be1271-5bab-4fd1-bb62-6ddff63afe3a.node4.buuoj.cn:81/
http://44be1271-5bab-4fd1-bb62-6ddff63afe3a.node4.buuoj.cn:81/chase
http://44be1271-5bab-4fd1-bb62-6ddff63afe3a.node4.buuoj.cn:81/leftt
http://44be1271-5bab-4fd1-bb62-6ddff63afe3a.node4.buuoj.cn:81/shoot
http://44be1271-5bab-4fd1-bb62-6ddff63afe3a.node4.buuoj.cn:81/door
```

```python
import requests
import re

url = "http://44be1271-5bab-4fd1-bb62-6ddff63afe3a.node4.buuoj.cn:81/"
pad = ""
while True:
    url_t = url + pad
    print(url_t)
    r = requests.get(url_t)
    reg = "window\.location.?=.+/[a-z]+/."
    reg = re.compile(reg)
    x = reg.findall(r.text)
    # ['window.location = "/chase/"', 'window.location = "/die/"']
    # ['window.location = "/die/"', 'window.location = "/die/"', 'window.location = "/leftt/"', 'window.location = "/die/"']
    if len(x)==0:
        break
    for i in x:
        if "die" not in i:
            pad = i.split("/")[1]
        else:
            continue
```

脚本

```python
import requests
import re


url = "http://44be1271-5bab-4fd1-bb62-6ddff63afe3a.node4.buuoj.cn:81/"
pad = "door"
while True:
    url_t = url + pad
    print(url_t)
    r = requests.get(url_t)
    reg = "src=\"/static/js/[a-z_]+\.js\""
    reg = re.compile(reg)
    x = reg.findall(r.text)
    # ['window.location = "/chase/"', 'window.location = "/die/"']
    # ['window.location = "/die/"', 'window.location = "/die/"', 'window.location = "/leftt/"', 'window.location = "/die/"']
    if len(x)==0:
        break
    pad = (x.split("\"")[1])[1:]
    
```

fight.js

```javascript
function scramble(flag, key) {
  for (var i = 0; i < key.length; i++) {
    let n = key.charCodeAt(i) % flag.length;
    let temp = flag[i];
    flag[i] = flag[n];
    flag[n] = temp;
  }
  return flag;
}

function check_action() {
  var action = document.getElementById("action").value;
  var flag = ["{hey", "_boy", "aaaa", "s_im", "ck!}", "_baa", "aaaa", "pctf"];

  // TODO: unscramble function
}
```

给出了flag的数组，要重新组合

```python
from itertools import permutations

flag = ["{hey", "_boy", "aaaa", "s_im", "ck!}", "_baa", "aaaa", "pctf"]

item = permutations(flag)
for i in item:
  k = ''.join(list(i))
  if k.startswith('pctf{hey_boys') and k[-1] == '}':
    print(k)
```
