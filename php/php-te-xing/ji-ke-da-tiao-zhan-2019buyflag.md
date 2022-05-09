# \[极客大挑战 2019]BuyFlag

## \[极客大挑战 2019]BuyFlag

## wp

访问pay.php提示

```
If you want to buy the FLAG:
You must be a student from CUIT!!!
You must be answer the correct password!!! 
```

抓包发现cookie有个user字段是0，改成1，然后提示要password

![](<../../.gitbook/assets/image (33) (1) (1) (1) (1) (1).png>)

在注释找到了password的代码

![](<../../.gitbook/assets/image (24) (1) (1) (1) (1).png>)

提示要POST数据password和money，password不能是数字，但是又要`==404`，利用PHP的弱比较，password是`404a`

根据前面提示的`Flag need your 100000000 money`，money要比100000000大，直接使用100000000会提示Nember lenth is too long，那就使用科学计数法`10e10`

最后POST的数据是`password=404a&money=10e10`
