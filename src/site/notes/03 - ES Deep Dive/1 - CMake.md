---
{"dg-publish":true,"permalink":"/03-es-deep-dive/1-c-make/","noteIcon":"","created":"2024-03-07T18:45:55.887+01:00","updated":"2024-05-25T20:15:58.254+02:00"}
---

## 零碎知识点

### register packages
	- two different kinds of packages
		- user packages registry
		- system packages registry
			- what does it mean? different scope?
	- how to register user packages?
		- `.cmake/packages`
		- example![Z - assets/images/Pasted image 20240307184804.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240307184804.png)
```bash
/home/maxin/zephyr-sdk-0.16.5/cmake
```

### clean
```c
cmake --build ./blinky/.build --target clean
```