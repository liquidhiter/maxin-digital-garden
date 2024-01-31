---
{"dg-publish":true,"permalink":"/3-online-courses/cs-61-b/lectures/chapter-6/cs-61-b-2018-spring-learning-notes-chapter-6/","noteIcon":"","created":"2024-01-31T22:49:21.451+01:00","updated":"2024-01-31T22:56:37.996+01:00"}
---

## Topic: Exceptions, Iterators, Iterables and Object Methods
### Iterable and iterator
```java
import java.util.Iterator;

public class ArraySet<T> implements Iterable<T> {

    private T[] items;
    private int size;

    /*======= Omit some code here =======*/

    /** returns an iterator (a.k.a. seer) */
    public Iterator<T> iterator() {
        return new ArraySetIterator();
    }

    private class ArraySetIterator implements Iterator<T> {
        private int nextIdx;

        public ArraySetIterator() {
            nextIdx = 0;
        }

        @Override
        public boolean hasNext() {
            return nextIdx < size;
        }

        @Override
        public T next() {
            return items[nextIdx++];
        }
    }

    public static void main(String[] args) {
        ArraySet<String> s = new ArraySet<>();
        s.add(null);
        s.add("horse");
        s.add("fish");
        s.add("house");
        s.add("fish");
        System.out.println(s.contains("horse"));
        System.out.println(s.size());

        Iterator<String> iter = s.iterator();
        while (iter.hasNext()) {
            System.out.println(iter.next());
        }

        for (String item : s) {
            System.out.println(item);
        }
    }
}
```
- `Iterable` requires the class to define the method of `iterator()` which returns an iterator
- `Iterator` implements how the iteration is working, specifically defining methods of `hasNext()` and `next()`
  - `next()` is called when the `hasNext()` returns false: common convention is to throw the `NoSuchElementException`
  - `next()` can be called without calling the `hasNext()` method


### `toString()` and `equals()`
```java
@Override
public boolean equals(Object obj) {
    /* self-equals */
    if (this == obj) {
        return true;
    }

    if (obj == null) {
        return false;
    }

    /* comparable for same class */
    if (obj.getClass() != this.getClass()) {
        return false;
    }

    ArraySet<T> other = (ArraySet<T>) obj;
    if (other.size() != this.size()) {
        return false;
    }

    for (T item : this) {
        if (!other.contains(item)) {
            return false;
        }
    }
    
    return true;
}
```
- understand the difference between `==` and `equals`


### Generic static methods
```java
public static <type> ArraySet<type> of(type... args) {
    ArraySet<type> result = new ArraySet<>();
    for (type t : args) {
        result.add(t);
    }

    return result;
}
```