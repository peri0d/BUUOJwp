# \[RCTF 2019]Nextphp

## \[RCTF 2019]Nextphp

## 考点



## wp

给了个shell的源码

```php
<?php
if (isset($_GET['a'])) {
    eval($_GET['a']);
} else {
    show_source(__FILE__);
}
```

`?a=phpinfo();`看到phpinfo中设置了disable\_functions

![](<../../.gitbook/assets/image (8).png>)

`?a=var_dump(scandir('/'));`提示没有权限，`php`看到两个文件，index.php和preload.php。再读取preload.php

```php
<?php
final class A implements Serializable {
    protected $data = [
        'ret' => null,
        'func' => 'print_r',
        'arg' => '1'
    ];

    private function run () {
        $this->data['ret'] = $this->data['func']($this->data['arg']);
    }

    public function __serialize(): array {
        return $this->data;
    }

    public function __unserialize(array $data) {
        array_merge($this->data, $data);
        $this->run();
    }

    public function serialize (): string {
        return serialize($this->data);
    }

    public function unserialize($payload) {
        $this->data = unserialize($payload);
        $this->run();
    }

    public function __get ($key) {
        return $this->data[$key];
    }

    public function __set ($key, $value) {
        throw new \Exception('No implemented');
    }

    public function __construct () {
        throw new \Exception('No implemented');
    }
}
```

这里就是利用反序列化绕过disable\_functions，有个FFI的方式
