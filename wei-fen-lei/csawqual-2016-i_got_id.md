# \[CSAWQual 2016]i\_got\_id

## \[CSAWQual 2016]i\_got\_id

## 考点

* Perl代码特性
* Perl特性
* Perl文件上传
* Perl特殊变量

## wp

三个页面，`cgi-bin/hello.pl`回显hello world，`cgi-bin/forms.pl`是表单提交，`cgi-bin/file.pl`是文件上传，都是用Perl写的页面。

第二个页面`cgi-bin/forms.pl`会回显表单提交的内容

&#x20;

![](<../.gitbook/assets/image (33).png>)

第三个页面会回显上传文件的内容

![](<../.gitbook/assets/image (31) (1).png>)

这里要对Perl很熟悉才能继续，回显上传文件内容，代码大致如下

```perl
use strict;
use warnings;
use CGI;
 
my $cgi= CGI->new;
if ( $cgi->upload( 'file' ) )
{
my $file= $cgi->param( 'file' );
while ( <$file> ) { print "$_"; } }
```

`upload()`函数是用于处理文件上传的标准函数，参数为在构造表单时 `<input type="file" name=" ... ” />`设置的`name`的值

`param()`函数用于获取传入的参数，可以是个普通字符串也可以是数组，如果是数组，则只返回第一个

这里也可以通过`/cgi-bin/forms.pl`判断是使用`param()`函数获取传入的参数

![](<../.gitbook/assets/image (5).png>)

然后通过`<$file>`读取文件句柄，使用`$_`获取文件每一行的数据

那么问题就来了，如果上传文件时，再增加一个file字段，让filename为空，那`$file`就会变成上传的文件内容，而输出的文件内容为空，如果把文件内容填`ARGV`，最后一行就是`while (<ARGV>) { print "$_"; }`。

payload

![](<../.gitbook/assets/image (7) (1).png>)

修改`/etc/passwd`为`/flag`就可以看到结果

还有另一种方式，进行getshell

既然在Perl中，直接在url给出参数就相当于命令行传参，也就是说可以进行命令执行

使用`ls`可以执行命令，但它是bash运行环境，整个语句的输出结果在shell的缓冲区里，也就是后台服务器才能看到，并不会被`$_`读取输出到html标签中。在linux里我们只需要管道操作就可以指定结果的存放位置了。

而Perl `open()`函数会默认打开一个管道，可以利用Perl `open()`函数打开的管道，进行劫持，通过`|`操作符，把内容引入`open()`函数已经打开的管道中，就可以输出到html标签中。

> Let me talk about Perl’s open() function. This function can also execute commands, because it is used to open pipes. In this case, you can use | as a delimiter, because Perl looks for | to indicate that open() is opening a pipe. An attacker can hijack an open() call which otherwise would not even execute a command by adding a | to his query.

payload：`POST /cgi-bin/file.pl?cat%20%2fflag%20|`，文件上传和上面相同

## 小结

1. Perl中的`<>`运算
   1. 如果尖括号中间是文件句柄，尖括号运算符允许你读取文件句柄，比如。
   2. 如果尖括号中间是搜索模式，尖括号运算符能返回与该模式匹配的文件列表，这称为一个glob，比如`< *.bat>`
   3. 尖括号运算符如果中间没有任何东西，那么它可以读取命令行上所有文件的内容。命令行执行`perl –w Example.pl file1 file2 file3`就会读取`file1 file2 file3`
2. `$_` 是默认参数的意思，指的是在不指定的情况下，程序处理的上一个变量
3. `$0`表示当前正在运行的Perl脚本名
4. perl将perl命令行的参数列表放进数组`ARGV`(`@ARGV`)中
   * `$ARGV`表示命令行参数代表的文件列表中，当前被处理的文件名
   * `@ARGV`表示命令行参数数组
   * `$ARGV[n]`：表示命令行参数数组的元素
   * `ARGV`：遍历数组变量`@ARGV`中所有文件名的特殊文件句柄
5. 在Perl中，直接在url给出参数就相当于命令行传参
6. [https://tsublogs.wordpress.com/2016/09/18/606/](https://tsublogs.wordpress.com/2016/09/18/606/)
7. [https://blog.csdn.net/wssmiss/article/details/105620355](https://blog.csdn.net/wssmiss/article/details/105620355)

