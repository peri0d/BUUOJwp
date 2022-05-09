# \[FBCTF2019]Products Manager

## \[FBCTF2019]Products Manager

## 考点

* 基于基于约束的SQL攻击的

## wp

给了源码，有三个功能，查看前五个产品、添加新产品和查看产品细节

大致看了一下源码，提示flag在facebook的description中

查看前五个产品就是index.php，执行db.php的`get_top_products()`函数，查询语句是写死的，获取前五个name

添加新产品是add.php，需要参数name、secret和description，其中name和description不为空，secret由大小写字母和数字组成，并且长度要大于等于10。然后使用`get_product($name)`判断是否已存在，然后调用`insert_product($name, hash('sha256', $secret), $description)`插入数据

```php
function get_product($name) {
  global $db;
  $statement = $db->prepare(
    "SELECT name, description FROM products WHERE name = ?"
  );
  check_errors($statement);
  $statement->bind_param("s", $name);
  check_errors($statement->execute());
  $res = $statement->get_result();
  check_errors($res);
  $product = $res->fetch_assoc();
  $statement->close();
  return $product;
}
function insert_product($name, $secret, $description) {
  global $db;
  $statement = $db->prepare(
    "INSERT INTO products (name, secret, description) VALUES
      (?, ?, ?)"
  );
  check_errors($statement);
  $statement->bind_param("sss", $name, $secret, $description);
  check_errors($statement->execute());
  $statement->close();
}
```

查看产品细节是view.php，需要参数name和secret。然后调用`check_name_secret($name, hash('sha256', $secret))`判断name和secret是否对应，最后调用`get_product($name)`获取数据

```php
function check_name_secret($name, $secret) {
  global $db;
  $valid = false;
  $statement = $db->prepare(
    "SELECT name FROM products WHERE name = ? AND secret = ?"
  );
  check_errors($statement);
  $statement->bind_param("ss", $name, $secret);
  check_errors($statement->execute());
  $res = $statement->get_result();
  check_errors($res);
  if ($res->fetch_assoc() !== null) {
    $valid = true;
  }
  $statement->close();
  return $valid;
}
```

这里是基于约束的SQL攻击，要求name和secret都是64位。

```sql
CREATE TABLE products (
  name char(64),
  secret char(64),
  description varchar(250)
);

INSERT INTO products VALUES('facebook', sha256(....), 'FLAG_HERE');
INSERT INTO products VALUES('messenger', sha256(....), ....);
INSERT INTO products VALUES('instagram', sha256(....), ....);
INSERT INTO products VALUES('whatsapp', sha256(....), ....);
INSERT INTO products VALUES('oculus-rift', sha256(....), ....);
```

如果输入的超过64位，分类讨论一下，查询(select)和插入(insert)

在查询中进行字符串比较时，若两字符串长度不一样，则会在较短的字符串末尾填充空格，使两个字符串长度一致。

在插入时若数据长度超过了预先设定的限制，数据库会对字符串进行截断，只保留限定的长度。

本题payload如下，在burp中操作好点

```
add.php
name:facebook+'%20'*64
secret:aaaaBBBB1234
description:1111

view.php
name:facebook
secret:aaaaBBBB1234
```

首先，插入的name大于规定的64位，会截断插入

| name                                                      | secret       | description        |
| --------------------------------------------------------- | ------------ | ------------------ |
| <mark style="color:red;">`facebook`</mark>                | \*\*\*\*\*\* | flag{\*\*\*\*\*\*} |
| <mark style="color:orange;">`facebook`</mark>             | aaaaBBBB1234 | 1111               |

然后执行check\_name\_secret时，传入<mark style="color:red;">`facebook`</mark>和`aaaaBBBB1234`，那么执行的语句就是`SELECT name FROM products WHERE name =`` `<mark style="color:red;">`facebook`</mark>` ``AND secret = aaaaBBBB1234`

它就会触发MySQL字符串比较的机制，在<mark style="color:red;">`facebook`</mark>末尾添加空格使它和<mark style="color:orange;">`facebook`</mark>            长度相同，然后再进行where比较，这样where比较是肯定通过的，check\_name\_secret返回TRUE

然后就会执行get\_product($name)获取<mark style="color:red;">`facebook`</mark>的信息，这样就造成了水平越权

```php
if (check_name_secret($name, hash('sha256', $secret)) === false) {
    return "Incorrect name or secret, please try again";
}
$product = get_product($name);
```
