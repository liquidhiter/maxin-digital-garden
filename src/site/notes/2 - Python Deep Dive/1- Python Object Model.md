---
{"dg-publish":true,"permalink":"/2-python-deep-dive/1-python-object-model/","noteIcon":"","created":"2024-01-29T06:21:34.847+01:00","updated":"2024-01-29T06:33:57.199+01:00"}
---

## source
https://fasionchan.com/python-source/object-model/overview/

## everything is an object
- no concept of `primitive types`: they are all objects
```Python3
>>> a = 1
>>> type(a)
<class 'int'>
>>> b = 1.1
>>> type(b)
<class 'float'>
>>> s = "python"
>>> type(s)
<class 'str'>
>>> isTrue = True
>>> type(isTrue)
<class 'bool'>
```
- `type` itself is also an `object`
```Python3
>>> int
<class 'int'>
>>> type(int)
<class 'type'>
```
- **type** and **instances** are all **objects** 
![Pasted image 20240129063204.png](/img/user/Pasted%20image%2020240129063204.png)