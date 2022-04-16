# \[BJDCTF2020]Mark loves cat

## \[BJDCTF2020]Mark loves cat

## 考点

* .git泄露
* 变量覆盖
* 代码审计

## wp

目录扫描发现`.git`泄露，[GitHack](https://github.com/lijiejie/GitHack)下载源码，或者用[Git\_Extract](https://github.com/gakki429/Git\_Extract)（反正我没成功）

flag.php定义了`$flag=...`

{% code title="index.php" %}
```php
include 'flag.php';
$yds = "dog";
$is = "cat";
$handsome = 'yds';
foreach($_POST as $x => $y){
    $$x = $y;
}
foreach($_GET as $x => $y){
    $$x = $$y;
}
foreach($_GET as $x => $y){
    if($_GET['flag'] === $x && $x !== 'flag'){
        exit($handsome);
    }
}
if(!isset($_GET['flag']) && !isset($_POST['flag'])){
    exit($yds);
}
if($_POST['flag'] === 'flag'  || $_GET['flag'] === 'flag'){
    exit($is);
}
echo "the flag is: ".$flag;
```
{% endcode %}

很明显的变量覆盖，同样很明显不能直接传入`flag=***`，否则会被覆盖，也就是考虑把其他值覆盖为flag。

在第一个exit处，GET不能传入flag参数，若传入其值只能为flag，否则返回handsome的值

第二个exit处，不POST或者GET传入flag参数就退出程序，否则返回yds的值

第三个exit处，POST或者GET传入flag参数的值如果为flag，否则返回is的值
