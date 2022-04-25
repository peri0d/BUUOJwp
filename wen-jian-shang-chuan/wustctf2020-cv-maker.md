# \[WUSTCTF2020]CV Maker

## \[WUSTCTF2020]CV Maker

## 考点

* PHP`exif_imagetype`函数
* burp改文件名绕过后缀限制
* 图片马制作

## wp

打开靶机，没有发现敏感目录，有登陆注册的功能，随便注册登陆，进入到更换头像的页面，不选择头像直接点上传会提示`exif_imagetype not image!`。

![](<../.gitbook/assets/image (8) (1) (1) (1).png>)

`exif_imagetype`函数会读取图片的第一个字节并检查。到这里初步判断是文件上传，然后在返回包看到PHP是5.5版本。

![](<../.gitbook/assets/image (27) (1) (1) (1) (1) (1).png>)

![](<../.gitbook/assets/image (17) (1) (1) (1) (1).png>)
