# \[网鼎杯 2020 玄武组]SSRFMe

## \[网鼎杯 2020 玄武组]SSRFMe

## 考点



## wp

打开靶机给了源码

第一个函数`check_inner_ip`，传入的URL必须以`http https gopher dict`开头，然后用parse\_url解析获取hostname

`gethostbyname()`函数用于获取hostname对应的IPv4地址，失败则返回hostname

`ip2long()`函数用于把IPv4地址转换为长整数

传入的URL对应的IP格式必须是`127.*   10.*  172.16.*  192.168.*`

否则返回False

```php
function check_inner_ip($url)
{
    $match_result=preg_match('/^(http|https|gopher|dict)?:\/\/.*(\/)?.*$/',$url);
    if (!$match_result)
    {
        die('url fomat error');
    }
    try
    {
        $url_parse=parse_url($url);
    }
    catch(Exception $e)
    {
        die('url fomat error');
        return false;
    }
    $hostname=$url_parse['host'];
    $ip=gethostbyname($hostname);
    $int_ip=ip2long($ip);
    return ip2long('127.0.0.0')>>24 == $int_ip>>24 || ip2long('10.0.0.0')>>24 == $int_ip>>24 || ip2long('172.16.0.0')>>20 == $int_ip>>20 || ip2long('192.168.0.0')>>16 == $int_ip>>16;
}
```

第二个函数`safe_request_url`，传入的url如果符合`check_inner_ip`函数的要求，就直接输出url

如果传入的不是私有地址，就使用PHP curl获取页面。如果私有地址中间有302跳转，就把跳转的目标url再用`safe_request_url`函数访问，这样就避免了用302跳转绕过本地限制

```php
function safe_request_url($url)
{

    if (check_inner_ip($url))
    {
        echo $url.' is inner ip';
    }
    else
    {
        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($ch, CURLOPT_HEADER, 0);
        $output = curl_exec($ch);
        $result_info = curl_getinfo($ch);
        if ($result_info['redirect_url'])
        {
            safe_request_url($result_info['redirect_url']);
        }
        curl_close($ch);
        var_dump($output);
    }

}
```

最后一段，提示先访问本地的`hint.php`

```php
if(isset($_GET['url'])){
    $url = $_GET['url'];
    if(!empty($url)){
        safe_request_url($url);
    }
} 
// Please visit hint.php locally. 
```

这些操作相当于限制本地访问，同时要求访问本地`hint.php`

常用策略就是八进制 十六进制 全0 等方法

```
http://0.0.0.0/hint.php    只能Linux下用
http://[0:0:0:0:0:ffff:127.0.0.1]/hint.php
```

得到新代码，向POST传入的file参数写入`"<?php echo 'redispass is root';exit();".$_POST['file']`

```php
<?php
if($_SERVER['REMOTE_ADDR']==="127.0.0.1"){
  highlight_file(__FILE__);
}
if(isset($_POST['file'])){
  file_put_contents($_POST['file'],"<?php echo 'redispass is root';exit();".$_POST['file']);
}
```

这里给了提示redis密码为`root`，这里没有写权限所以写不了shell

预期解法是redis主从复制rce BUU环境有问题复现不了

下载[redis-ssrf](https://github.com/xmsec/redis-ssrf)，修改[ssrf-redis.py](https://github.com/xmsec/redis-ssrf/blob/master/ssrf-redis.py)

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

下载[redis-rogue-server](https://github.com/Dliv3/redis-rogue-server) &#x20;

把exp.so ssrf-redis.py rogue-server.py上传到VPS

运行ssrf-redis.py得到payload

再运行rogue-server.py

把payload再URL编码一下，在靶机用编码后的payload打过去

## 小结

1. [\[网鼎杯 2020 玄武组\]SSRFMe](https://liotree.github.io/2020/07/10/%E7%BD%91%E9%BC%8E%E6%9D%AF-2020-%E7%8E%84%E6%AD%A6%E7%BB%84-SSRFMe/)1
2. [\[网鼎杯 2020 玄武组\]SSRFMe](https://www.cnblogs.com/karsa/p/14123995.htm)2

