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

如果GET传入`flag=123`，在第9行就是`$flag=$123`，第12行的判断就是`("123" === "flag" && "flag" !== 'flag')`返回FALSE，第16行返回FALSE，第19行返回FALSE，最后相当于执行<mark style="color:orange;">`echo "the flag is: ".$123`</mark>

如果POST传入`flag=123`，在第6行就是`$flag="123"`，第12行的判断返回FALSE，第16行返回FALSE，第19行返回FALSE，最后相当于执行<mark style="color:orange;">`echo "the flag is: 123"`</mark>

所以要进行变量覆盖，flag必须是作为参数值传入，如果GET传入`abc=flag`，在第9行就是`$abc=$flag`，第12行的判断返回FALSE，第16行返回TRUE，输出$yds，所以直接让`abc`是`yds`就可以了

payload：`?yds=flag`

## 小结

1. 结合输入搞明白代码
