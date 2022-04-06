# \[极客大挑战 2020]Roamphp1 Welcome

## \[极客大挑战 2020]Roamphp1-Welcome

## 考点

* 405是请求方式不对
* 数组绕过hash

## wp

状态码为405，代表请求的方式不对。随便POST一些东西，可以看到返回的源码

```php
 <?php
error_reporting(0);
if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
header("HTTP/1.1 405 Method Not Allowed");
exit();
} else {
    
    if (!isset($_POST['roam1']) || !isset($_POST['roam2'])){
        show_source(__FILE__);
    }
    else if ($_POST['roam1'] !== $_POST['roam2'] && sha1($_POST['roam1']) === sha1($_POST['roam2'])){
        phpinfo();  // collect information from phpinfo!
    }
}
```

是个hash碰撞，直接POST数据就可以绕过了`roam1[]=22&roam2[]=33`

然后看到phpinfo的界面，发现f1444aagggg.php，直接访问不存在，然后在后面找到了flag

![](../../.gitbook/assets/Cv54d\_9U5BoobLe0Ea7Aw0qenZQmdtM79O\_5OQz93dI.png)

![](../../.gitbook/assets/WHyibmT2v-aNCcSXhcanvcDQ6aVhg\_U59\_U8lKXlwRs.png)
