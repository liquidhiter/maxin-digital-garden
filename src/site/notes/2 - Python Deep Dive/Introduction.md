---
{"dg-publish":true,"permalink":"/2-python-deep-dive/introduction/","noteIcon":"","created":"2024-01-28T21:23:05.020+01:00","updated":"2024-01-28T22:01:25.137+01:00"}
---

## source
- https://fasionchan.com/python-source/preface/story/
## procfs
- reference: https://en.wikipedia.org/wiki/Procfs
- what
	- `proc filesystem` 
		- presents information about `processes` and other `system information` in a `hierachical file-like structure`, providing a standard method for  dynamically accessing process data held in the `kernel`
		- typically mapped to a `a mount point` named `/proc` at `boot` time
		- ***The proc file system acts as an interface to internal data structures about running processes in the kernel.***
- demo
	- env:
		- `Ubuntu 22.04.3 LTS on Windows 10 x86_6`
		- `5.15.133.1-microsoft-standard-WSL2`
		```bash
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 1
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 136
		dr-xr-xr-x   9 messagebus      messagebus                    0 Jan 26 23:38 138
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 142
		dr-xr-xr-x   9 syslog          syslog                        0 Jan 26 23:38 143
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 144
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 145
		dr-xr-xr-x   9 root            root                          0 Jan 28 20:08 193735
		dr-xr-xr-x   9 root            root                          0 Jan 28 20:08 193736
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 28 19:55 193741
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 28 19:58 193933
		dr-xr-xr-x   9 root            root                          0 Jan 28 20:40 193935
		dr-xr-xr-x   9 root            root                          0 Jan 28 20:40 193937
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 28 20:09 193939
		dr-xr-xr-x   9 root            root                          0 Jan 28 20:45 195285
		dr-xr-xr-x   9 root            root                          0 Jan 28 20:45 195286
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 28 20:45 195287
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 28 20:45 195288
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 28 20:45 195293
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 28 20:45 195297
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 28 20:45 195480
		dr-xr-xr-x   9 root            root                          0 Jan 28 20:45 195882
		dr-xr-xr-x   9 root            root                          0 Jan 28 20:45 195883
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 28 20:45 195884
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 28 20:45 195891
		dr-xr-xr-x   9 root            root                          0 Jan 28 20:45 195897
		dr-xr-xr-x   9 root            root                          0 Jan 28 20:45 195898
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 28 20:45 195899
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 28 20:45 195913
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 28 20:45 195982
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 28 20:45 195991
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 28 20:45 196009
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 28 20:45 196019
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 28 20:46 196150
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 28 20:46 196256
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 2
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 28 21:27 203884
		dr-xr-xr-x   9 root            root                          0 Jan 27 09:11 20407
		dr-xr-xr-x   9 root            root                          0 Jan 27 09:11 20411
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 206
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 208
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 212
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 214
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 323
		dr-xr-xr-x   9 root            maxin                         0 Jan 26 23:38 335
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 35
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 26 23:38 370
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 26 23:38 371
		dr-xr-xr-x   9 maxin           maxin                         0 Jan 26 23:38 376
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 394
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 55
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 6
		dr-xr-xr-x   9 root            root                          0 Jan 27 00:17 7858
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 79
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 80
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 81
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 82
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 83
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 84
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 86
		dr-xr-xr-x   9 root            root                          0 Jan 26 23:38 88
		dr-xr-xr-x   9 systemd-resolve systemd-resolve               0 Jan 26 23:38 95

		# NOTE:
		# /proc not only contains the kernel related information
		# for example, cpu_info
		
		dr-xr-xr-x   2 root            root                          0 Jan 26 23:38 acpi
		-r--r--r--   1 root            root                          0 Jan 26 23:38 buddyinfo
		dr-xr-xr-x   4 root            root                          0 Jan 26 23:38 bus
		-r--r--r--   1 root            root                          0 Jan 26 23:38 cgroups
		-r--r--r--   1 root            root                          0 Jan 26 23:38 cmdline
		-r--r--r--   1 root            root                      25043 Jan 26 23:38 config.gz
		-r--r--r--   1 root            root                          0 Jan 26 23:38 consoles
		-r--r--r--   1 root            root                          0 Jan 26 23:38 cpuinfo
		-r--r--r--   1 root            root                          0 Jan 26 23:38 crypto
		-r--r--r--   1 root            root                          0 Jan 26 23:38 devices
		-r--r--r--   1 root            root                          0 Jan 26 23:38 diskstats
		-r--r--r--   1 root            root                          0 Jan 26 23:38 dma
		dr-xr-xr-x   4 root            root                          0 Jan 26 23:38 driver
		-r--r--r--   1 root            root                          0 Jan 26 23:38 execdomains
		-r--r--r--   1 root            root                          0 Jan 26 23:38 filesystems
		dr-xr-xr-x  11 root            root                          0 Jan 26 23:38 fs
		-r--r--r--   1 root            root                          0 Jan 26 23:38 interrupts
		-r--r--r--   1 root            root                          0 Jan 26 23:38 iomem
		-r--r--r--   1 root            root                          0 Jan 26 23:38 ioports
		dr-xr-xr-x  24 root            root                          0 Jan 26 23:38 irq
		-r--r--r--   1 root            root                          0 Jan 26 23:38 kallsyms
		-r--------   1 root            root            140737471590400 Jan 26 23:38 kcore
		-r--r--r--   1 root            root                          0 Jan 26 23:38 key-users
		-r--r--r--   1 root            root                          0 Jan 26 23:38 keys
		-r--------   1 root            root                          0 Jan 26 23:38 kmsg
		-r--------   1 root            root                          0 Jan 26 23:38 kpagecgroup
		-r--------   1 root            root                          0 Jan 26 23:38 kpagecount
		-r--------   1 root            root                          0 Jan 26 23:38 kpageflags
		-r--r--r--   1 root            root                          0 Jan 26 23:38 loadavg
		-r--r--r--   1 root            root                          0 Jan 26 23:38 locks
		-r--r--r--   1 root            root                          0 Jan 26 23:38 mdstat
		-r--r--r--   1 root            root                          0 Jan 26 23:38 meminfo
		-r--r--r--   1 root            root                          0 Jan 26 23:38 misc
		-r--r--r--   1 root            root                          0 Jan 26 23:38 modules
		lrwxrwxrwx   1 root            root                         11 Jan 26 23:38 mounts -> self/mounts
		-rw-r--r--   1 root            root                          0 Jan 26 23:38 mtrr
		lrwxrwxrwx   1 root            root                          8 Jan 26 23:38 net -> self/net
		-r--------   1 root            root                          0 Jan 26 23:38 pagetypeinfo
		-r--r--r--   1 root            root                          0 Jan 26 23:38 partitions
		-r--r--r--   1 root            root                          0 Jan 26 23:38 schedstat
		lrwxrwxrwx   1 root            root                          0 Jan 26 23:38 self -> 203884
		-r--r--r--   1 root            root                          0 Jan 26 23:38 softirqs
		-r--r--r--   1 root            root                          0 Jan 26 23:38 stat
		-r--r--r--   1 root            root                          0 Jan 26 23:38 swaps
		dr-xr-xr-x   1 root            root                          0 Jan 26 23:38 sys
		dr-xr-xr-x   5 root            root                          0 Jan 26 23:38 sysvipc
		lrwxrwxrwx   1 root            root                          0 Jan 26 23:38 thread-self -> 203884/task/203884
		-r--------   1 root            root                          0 Jan 26 23:38 timer_list
		dr-xr-xr-x   6 root            root                          0 Jan 26 23:38 tty
		-r--r--r--   1 root            root                          0 Jan 26 23:38 uptime
		-r--r--r--   1 root            root                          0 Jan 26 23:38 version
		-r--------   1 root            root                          0 Jan 26 23:38 vmallocinfo
		-r--r--r--   1 root            root                          0 Jan 26 23:38 vmstat
		-r--r--r--   1 root            root                          0 Jan 26 23:38 zoneinfo
		```
		> NOTE: /proc not only contains the kernel related information


## `namedtuple` instead of custom class
- see: https://github.com/giampaolo/psutil/blob/2f1cd0cafa895786183cf43bb5f02fceeb68de55/psutil/_common.py#L202
```python3
# psutil.users()
suser = namedtuple('suser', ['name', 'terminal', 'host', 'started', 'pid'])
# psutil.net_connections()
sconn = namedtuple('sconn', ['fd', 'family', 'type', 'laddr', 'raddr',
                             'status', 'pid'])
```
- `__dict__` saves all the attributes of a class instance
- using custom class here results in the huge amount of memory requirement as each connection needs an instance
- `namedtuple` is a better option
	- application scenery: there is **a large amount of fixed / static attributes**

## python IO and kernel IO
- physical files on the disk
	- e.x. `$HOME/.zshrc`
- mapped files on the disk
	- e.x. `/proc/cpu_info`
- buffer sizes are different when Python processes these two different types of files
	- [ ] #task although here I categorized them as `physical files and mapped files`, what are they called in the community, and what's the actual difference?
- Experiment
	- env: `Python 3.10.12 (main, Nov 20 2023, 15:14:05) [GCC 11.4.0] on linux`
```python3
f = open("test.txt)
f.buffer.__sizeof__()
>>> 4248

f = open("/proc/cpu_info")
f.buffer.__sizeof__()
>>> 1176
```
> 4248 = 4096 + 152, where `152` is the size of the header of reading buffer object
> 1176   = 1024 + 152=
- [ ] #task I actually don't know the name of this object, i.e. reading buffer object, might be wrong here. Figure out what is the name, and what it is and how it is used
- ROOT cause:
	- reading buffer of the virtual files is **4 times** less than the real files, which results in more frequent context switching

## kernel
- [ ] #task understand the section related to kernel patch later