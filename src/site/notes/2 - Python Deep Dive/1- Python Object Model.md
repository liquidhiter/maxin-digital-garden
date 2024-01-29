---
{"dg-publish":true,"permalink":"/2-python-deep-dive/1-python-object-model/","noteIcon":"","created":"2024-01-29T06:21:34.847+01:00","updated":"2024-01-29T07:06:14.654+01:00"}
---

## source
https://fasionchan.com/python-source/object-model/overview/

## everything is an object
- no concept of `primitive types`: they are all objects
```Python
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
```Python
>>> int
<class 'int'>
>>> type(int)
<class 'type'>
# int is an instance of the class type, which itself is a class
```
- **type** and **instances** are all **objects** 
![Pasted image 20240129063204.png](/img/user/Pasted%20image%2020240129063204.png)
- all built-in and custom classes inherit from `Object` by default
![Pasted image 20240129064059.png](/img/user/Pasted%20image%2020240129064059.png)

- Experiment
```Python
>>> class Dog(object):
...     def yelp(self):
...             print("woof")
...
>>> dog = Dog()
>>> dog.yelp()
woof
>>> type(dog.yelp)
<class 'method'>
>>> type(dog)
<class '__main__.Dog'>
>>> type(Dog)
<class 'type'>

>>> class Sleuth(Dog):
...     def yelp(self):
...             print("shit")
...
>>> sleuth = Sleuth()
>>> sleuth.yelp()
shit
>>> type(sleuth.yelp)
<class 'method'>
>>> type(Sleuth)
<class 'type'>
>>> issubclass(Sleuth, Dog)
True
>>> isinstance(sleuth, Dog)
True
>>> issubclass(Sleuth, object)
True
```
> <class 'method'>

- `object` itself is an instance of `type`
- `type` inherits from `object`
- `type` itself is an instance of `type`
```Python
>>> type(object)
<class 'type'>
>>> type(type)
<class 'type'>
>>> issubclass(type, object)
True
>>> type.__base__
<class 'object'>
```
![Pasted image 20240129070612.png](/img/user/Pasted%20image%2020240129070612.png)