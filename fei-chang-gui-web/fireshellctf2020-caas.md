# \[FireshellCTF2020]Caas

## \[FireshellCTF2020]Caas

## 考点

* c语言在include特点

## wp

题目的功能是编译代码，输入一个print(1)提示的有信息

```xml
Sorry, we could not compile this code.
b'/tmp/caas_g_cr0hmg.c:1:7: error: expected declaration specifiers or \xe2\x80\x98...\xe2\x80\x99 before numeric constant\n print(1)\n ^\n'
```

看提示，是c语言，那就用c语言的代码试试

```cpp
#include <stdio.h>
 
int main() {
    printf("Hello, World! \n");
    return 0;
}
```

点击编译，给了一个二进制文件，是c语言编译的文件，可以直接在Linux中执行

c语言在include引入一个无法解析的头文件时，会报错并返回文件内容，所以本题的payload就是`#include "/flag"`

## 小结

1. c语言在include引入一个无法解析的头文件时，会报错并返回文件内容
