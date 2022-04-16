# \[极客大挑战 2019]Upload

## \[极客大挑战 2019]Upload

## 考点

* 文件上传绕过`<?`
* 文件上传解析漏洞

## wp

提交PHP文件

![](<../.gitbook/assets/image (4).png>)

提交图片马

![](<../.gitbook/assets/image (6).png>)

![](<../.gitbook/assets/image (23).png>)

过滤后缀和`<?`，重新构造图片马

![](<../.gitbook/assets/image (35).png>)

![](<../.gitbook/assets/image (7).png>)

扫描出来upload目录，那文件保存在`upload/123.jpg`，接下来是如何让服务器解析，先试试各种后缀。上传`.phtml`发现成功，访问就可以getshell

![](<../.gitbook/assets/image (26) (1).png>)

## 小结

1. 上传图片马无法解析，除了考虑`.htaccess`和`.user.ini`，还要考虑是否解析各种后缀
