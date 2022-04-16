# \[ACTF2020 新生赛]Upload

## \[ACTF2020 新生赛]Upload

## 考点

* 文件上传phtml解析

## wp

传一个php测试一下

![](<../.gitbook/assets/image (29).png>)

![](<../.gitbook/assets/image (27) (1).png>)

删除check再上传

![](<../.gitbook/assets/image (18) (1).png>)

说明还存在过滤，可能是对文件内容或是文件头的过滤。继续上传PHP文件

```
GIF89a
<script language='php'>phpinfo();</script>
```

![](<../.gitbook/assets/image (28).png>)

换一下后缀试一下，换成phtml

![](<../.gitbook/assets/image (8).png>)

访问成功解析，再传shell即可。
