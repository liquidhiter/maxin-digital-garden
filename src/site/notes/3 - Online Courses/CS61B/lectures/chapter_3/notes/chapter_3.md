---
{"dg-publish":true,"permalink":"/3-online-courses/cs-61-b/lectures/chapter-3/notes/chapter-3/","noteIcon":"","created":"2024-01-31T22:49:21.475+01:00","updated":"2024-01-31T22:50:35.159+01:00"}
---


# CS61B 2018 Spring Learning Notes - Chapter 3

## Topic: test

- example of `junit`
```java
import org.junit.Test;
import static org.junit.Assert.*;

public class TestSort {

    @Test
    public void testSort() {
        String[] input = {"i", "have", "an", "egg"};
        String[] expected = {"an", "egg", "have", "i"};
        Sort.sort(input);
        assertArrayEquals(expected, input);
    }

    @Test
    public void testFindSmallest() {
        String[] input = {"i", "have", "an", "egg"};
        int expected = 2;
        int actual = Sort.findSmallest(input, 0, input.length);
        assertEquals(expected, actual);
    }

    @Test
    public void testSwap() {
        String[] input = {"i", "have", "an", "egg"};
        String[] expected = {"an", "have", "i", "egg"};
        Sort.swap(input, 0, 2);
        assertArrayEquals(expected, input);
    }
}
```

### interface
```java
public interface List61B<T> {
    public void addFirst(T x);
    public void addLast(T x);
    public T getFirst();
    public T getLast();
    public T removeLast();
    public T get(int i);
    public void insert(T x, int position);
    public int size();

    /* implementation of method is supported in Java interface 
     * as `default` meant: all sub-classes use this implementation if no override is provided
    */
    default public void print() {
        for (int i = 0; i < size(); i += 1) {
            System.out.print(get(i) + " ");
        }
        System.out.println();
    }
}
```

### dynamic type and override and polymorphism
```java
List61B<String> list = new SLList61B<String>();
list.addLast();
```
- `list` is an object of type `List61B<String>`
- `list` points to an object of type `SLList61B<String>`
- memory compatible as `SLList61B` follows the same `layout` (might not be a good word here): `methods`
- `addLast` method call refers to the pointee object

### static type and overload
```java
public static void peek(List61B<String> list)
public static void peek(SLList<String> list)

SLList<String> sLList = new SLList<String>("HI");
AList<String> aList = new AList<String>("IH");
SLList<String> SP = new SLList<String>();
List61B<String> LP = SP;
peek(SP); // ca;; peek(SLList<String> list)
peek(LP); // ca;; peek(SLList<String> list)
```