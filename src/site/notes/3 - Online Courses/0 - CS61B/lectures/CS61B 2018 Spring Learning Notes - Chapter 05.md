---
{"dg-publish":true,"permalink":"/3-online-courses/0-cs-61-b/lectures/cs-61-b-2018-spring-learning-notes-chapter-05/","noteIcon":"","created":"2024-01-31T22:49:21.460+01:00","updated":"2024-01-31T22:56:29.981+01:00"}
---

## Topic: More Java launguage stuff
### Autoboxing and unboxing
- primitive types and generic type arguments
- implicit convertion between the primitive types and wrapper

dis-advantages of using wrapper type for primitive ones
- auto-boxing results in worse performance
- arrays are never auto-boxed or auto-unboxed
- wrapper types require much more memory than primitive ones

### Immutabulity
- similar concept as in Python: be immutable as it should be, e.x. tuple rather than list
- final reference only guarantees that the reference itself is not changable, not the object


### Generic methods
```java
public static <K, V> V get(Map61B<K, V> map, K key) {
    /* */
}
```

```java
public static <K extends Comparable<K>, V> K maxKeys(Map61B<K, V> map, K key) {
    /* */
}
```
- `<K extends Comparable<K>, V>` all classes of K needs to be comparable