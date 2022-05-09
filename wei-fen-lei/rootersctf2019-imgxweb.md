# \[RootersCTF2019]ImgXweb

## \[RootersCTF2019]ImgXweb

## 考点

* `robots.txt`泄露
* jwt伪造session

## wp

先注册登录，然后是个文件上传的功能

![](<../.gitbook/assets/image (35).png>)

随便上传一个文件，可以得到路径`static/5aefd8409e75f852181d088da9e4ce1a/123.txt`

扫一下目录，发现`robots.txt`，提示访问`/static/secretkey.txt`，得到`you-will-never-guess`

然后去jwt.io验证一下，把user改成admin

![](<../.gitbook/assets/image (32) (1).png>)

然后得到flag图片

![](<../.gitbook/assets/image (23) (1).png>)

burp抓包看到flag

![](<../.gitbook/assets/image (28) (1).png>)
