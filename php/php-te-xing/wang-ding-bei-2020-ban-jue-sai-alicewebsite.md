# \[网鼎杯 2020 半决赛]AliceWebsite

给了源码，index.php有个明显的文件包含，还没做任何过滤

因为有file\_exists，所以不能用`php://filter/read=convert.base64-encode/resource=index.php`

直接传?action=/etc/passwd，可以看到结果，再传?action=/flag就有了
