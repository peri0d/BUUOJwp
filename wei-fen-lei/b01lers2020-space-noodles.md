# \[b01lers2020]Space Noodles

## \[b01lers2020]Space Noodles

## 考点

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

![](<../.gitbook/assets/image (2).png>)

```
curl -X OPTIONS "http://ac3d3754-ce97-47cb-a140-de19ded4b2e1.node4.buuoj.cn:81/circle/one/" --output 1.pdf
```

`/two/`使用~~PUT~~请求返回如下

```
curl -X PUT "http://ac3d3754-ce97-47cb-a140-de19ded4b2e1.node4.buuoj.cn:81/two/" 
Put the dots???
```

使用CONNECT请求会返回png图片

![](<../.gitbook/assets/image (14).png>)

