---
{"dg-publish":true,"permalink":"/2-python-deep-dive/z-python-source-code/","noteIcon":"","created":"2024-01-29T23:12:17.367+01:00","updated":"2024-01-29T23:15:35.654+01:00"}
---

> object.h

```c
#ifdef Py_TRACE_REFS
/* Define pointers to support a doubly-linked list of all live heap objects. */
#define _PyObject_HEAD_EXTRA            \
    struct _object *_ob_next;           \
    struct _object *_ob_prev;

#define _PyObject_EXTRA_INIT 0, 0,

#else
#define _PyObject_HEAD_EXTRA
#define _PyObject_EXTRA_INIT
#endif

/* PyObject_HEAD defines the initial segment of every PyObject. */
#define PyObject_HEAD                   PyObject ob_base;

#define PyObject_HEAD_INIT(type)        \
    { _PyObject_EXTRA_INIT              \
    1, type },

#define PyVarObject_HEAD_INIT(type, size)       \
    { PyObject_HEAD_INIT(type) size },
```
- `PyObject_HEAD_INIT` is used to initialize the `PyObject` header
	- `1` is the initial value of the `reference counter`
	- `type` is a `type object`
	- `_PyObject_EXTRA_INIT` is needed when `debug` is enabled, i.e. two extra pointers are created, and initialized to `NULL`


