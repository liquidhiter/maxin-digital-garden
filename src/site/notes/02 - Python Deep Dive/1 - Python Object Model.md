---
{"dg-publish":true,"permalink":"/02-python-deep-dive/1-python-object-model/","noteIcon":"","created":"2024-01-29T06:21:34.847+01:00","updated":"2024-01-29T21:50:17.826+01:00"}
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
![Z - assets/images/Pasted image 20240129063204.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240129063204.png)
- all built-in and custom classes inherit from `Object` by default
![Z - assets/images/Pasted image 20240129064059.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240129064059.png)

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
![Z - assets/images/Pasted image 20240129070612.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240129070612.png)

## variable is just a name
```Python
a = 1
id(a)
>>> 139725558252048
b = a
id(b)
>>> 139725558252048
```
- `a` and `b` both stores the address of the integer literal
	- namely, `pointers` 

## mutable and immutable objects
```Python
>>> a = 1
>>> id(a)
139725558252048
>>> a += 1
>>> id(a)
139725558252080
```
- `integer` is immutable in `Python`
	- `a += 1` actually does the following steps
		- fetch value of a and add it by 1, and then create a new object with the new value
		- points the original variable `a` to the new object

> Old object(s) are garbage collected (GC), the efficiency can be decreased a lot.

- [ ] #retrospect constant pool - the way to improve the efficiency of integer manipulation used in `Python` (actually similar technique is used in other modern languages such as `Java`)

As for the mutable objects
```Python
>>> l = [1,2,3]
>>> id(l)
139725535787648
>>> l += [4]
>>> id(l)
139725535787648
```
- [ ] #retrospect how the list is represented and maintained in `Python`?
![Z - assets/images/Pasted image 20240129212153.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240129212153.png)

## objects with fixed size and variable size

### variable size
#### integers
```Python
>>> import sys
>>> sys.getsizeof(1)
28
>>> sys.getsizeof(1000000)
28
>>> sys.getsizeof(100000000000000000000000000000000000000000000)
44
```
- big integers are supported in `Python` by concatenating  multiple `32-bit` integers together
- [ ] #retrospect understand how integer is represented in `Python`: header in the object used to record how many `32-bit` integers  are used ?
#### strings
```Python
>>> import sys
>>> sys.getsizeof("a")
50
>>> sys.getsizeof("ab")
51
>>> sys.getsizeof("abcdefghijklmnopqrstuvwxyz")
75
```
- similar to the integer representation
	- NOTE: `sizeof` a character `a` is much larger than what it is needed to represent itself (usually one byte is enough)
- [ ] #retrospect understand how string is represented in `Python`

### fixed size
#### floating numbers
```Python
>>> import sys
>>> sys.getsizeof(1.0)
24
>>> sys.getsizeof(1.1111)
24
>>> sys.getsizeof(1.1111111111111111111111111111111111111111111111)
24
```
- `float` is implemented as a `double` , its size is fixed
- larger range of values are supported by the `float` (`double`) at the cost of **precision**
```Python
>>> import sys
>>> sys.getsizeof(1000000000000000000000000000000000.)
24
>>> int(1000000000000000000000000000000000.)
999999999999999945575230987042816
```
- range is limited
```Python
>>> 10. ** 1000
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
OverflowError: (34, 'Numerical result out of range')
```
