# \[GXYCTF2019]StrongestMind

## \[GXYCTF2019]StrongestMind

## wp

![](<../.gitbook/assets/image (29) (1) (1) (1).png>)

写脚本计算1000次

```python
import requests
import re
import time

url = 'http://76561f35-2f9d-46ca-a58e-e4d944ebde9c.node4.buuoj.cn:81/index.php'

s = requests.session()
r = s.get(url)
while True:
    calc = re.search('<br>(\d*)(-|\+)(\d*)<br>', r.text.replace(' ',''))
    ans = eval(calc.group().replace('<br>',''))

    data = {
      'answer': ans
    }
    r = s.post(url, data=data)
    print(r.text)
    time.sleep(0.1)
```
