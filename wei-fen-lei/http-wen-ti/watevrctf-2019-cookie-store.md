# \[watevrCTF-2019]Cookie Store

## \[watevrCTF-2019]Cookie Store

## wp

直接选择buy Flag Cookie，然后抓包，发现cookie中有一串base64

![](<../../.gitbook/assets/image (28) (1) (1) (1) (1).png>)



解码后得到

```
{"money": 50, "history": []}
```

把50改成500，编码后修改session重新访问可以得到flag，直接在浏览器修改就可以了。

```
eyJtb25leSI6IDUwMCwgImhpc3RvcnkiOiBbXX0=
```

![](<../../.gitbook/assets/image (9) (1) (1) (1).png>)
