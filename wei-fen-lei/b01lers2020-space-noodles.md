# \[b01lers2020]Space Noodles

## \[b01lers2020]Space Noodles

## 考点

* 脑洞题

## wp

提示`Cant GET /`，使用POST，提示一些文本，大意是要从五个元素中发现信息

![](<../.gitbook/assets/image (29).png>)

没看出啥，在右键查看页面源代码

![](<../.gitbook/assets/image (19).png>)

五个提示如下

```
/circle/one/
/two/
/square/
/com/seaerch/
/vim/quit/
```

`/circle/one/`使用`OPTIONS`请求返回pdf

![](<../.gitbook/assets/image (3).png>)

```
curl -X OPTIONS "http://ac3d3754-ce97-47cb-a140-de19ded4b2e1.node4.buuoj.cn:81/circle/one/" --output 1.pdf
```

``

`/two/`使用`PUT`请求

```
curl -X PUT "http://ac3d3754-ce97-47cb-a140-de19ded4b2e1.node4.buuoj.cn:81/two/" 
Put the dots???
```

使用`CONNECT`请求会返回png图片，`up_on_noodles_`

![](<../.gitbook/assets/image (14).png>)

`/square/`用`DELETE`请求返回图片

```
curl -X DELETE "http://ac3d3754-ce97-47cb-a140-de19ded4b2e1.node4.buuoj.cn:81/square/" --output 2.png
```

![](<../.gitbook/assets/image (2).png>)

结果

```
    E
    S
    I
    R
    P
  E R
  C E
  A T
E P N
TASTES
 L A U
 D U L
 E   A
 R   C
 A   O
 A
 N
```

中间一行就是`tastes`

`/com/seaerch/`用`GET`请求

```
curl -X GET "http://ac3d3754-ce97-47cb-a140-de19ded4b2e1.node4.buuoj.cn:81/com/seaerch/"
<htlm>

,,,,,,,,,<search> <-- comment for search --!>:

  ERROR </> search=null</end>

</html>
```

`GET`请求`?search=1`没有结果，再把请求内容放在POST请求体中看一下

![](<../.gitbook/assets/image (33).png>)

改成`search=flag`可以得到结果`_good_in_s`

```
<htlm>
,,,,,,,,,<search> <-- comment for search --!>:
  <query> good search</query>
  results: <p>_good_in_s</p>:w
</html>
```

/vim/quit/用TRACE请求

```
TRACE /vim/quit/?exit=:wq
```

得到

```
   <hteeemel<body>>

      <flag> well done wait </flag>
<text> this one/> <flag>pace_too}</flag>

</>
```

全部拼接

`flag{ketchup_on_noodles_tastes_good_in_space_too}`

## 小结

1. [https://ctftime.org/task/10706](https://ctftime.org/task/10706)
2. [https://syunaht.com/p/725288439.html](https://syunaht.com/p/725288439.html#!)
