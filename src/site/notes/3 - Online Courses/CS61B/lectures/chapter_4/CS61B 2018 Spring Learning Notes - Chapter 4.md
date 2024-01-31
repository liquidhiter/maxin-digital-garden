---
{"dg-publish":true,"permalink":"/3-online-courses/cs-61-b/lectures/chapter-4/cs-61-b-2018-spring-learning-notes-chapter-4/","noteIcon":"","created":"2024-01-31T22:49:21.370+01:00","updated":"2024-01-31T22:56:14.855+01:00"}
---

## Topic: Inheritance and Implements

### Extends
subclasses share the common methods with the parent class
### Constructors
- create the instance of parent class before the child class
- implicit and explicit call of `super` to first construct the parent objects
### `Object`
- All classes in Java extends `Object` class explicitly or implicitly by default
### Encapsulation
### Abstraction
### Type checking and casting
- compile-time type of functions
- compile-time type of expressions (`rhs` and `lhs`)
### High-order functions
### Generalize the behavior through inheritance
```java
public class Dog implements OurComparable {
    private String name;
    private int size;

    public Dog(String n, int s) {
        name = n;
        size = s;
    }

    public void bark() {
        System.out.println(name + " says bark");
    }

    public int compareTo(Object o) {
        Dog dog = (Dog) o;
        if (this.size < dog.size) {
            return -1;
        } else if (this.size == dog.size) {
            return 0;
        }

        return 1;
    }

    public static OurComparable max(OurComparable[] items) {
        int maxIndex = 0;
        for (int i = 0; i < items.length; i++) {
            int cmp = items[i].compareTo(items[maxIndex]);
            if (cmp > 0) {
                maxIndex = i;
            }
        }

        return items[maxIndex];
    }
}
```
- all classes that are supposed to be compared by calling `max` function need to implement the `OurComparable` interface
- note: static type of the input parameters of `max` function is `OurComparable`, which generalize the function

### Comparable interface
```java
public class Dog implements Comparable<Dog> {
    private int size;
    public int compareTo(Dog dog) {
        return this.size - dog.size;
    }
}
```
### ADT
- no implementation but only operations
### Collections
- list
- map
- set
### Qualities of interfaces
- All methods must be `public`.
- All variables must be `public static final`.
- Cannot be instantiated
- All methods are by default abstract unless specified to be `default`
- Can implement more than one interface per class
### Qualities of abstract class
- Methods can be public or private
- Can have any types of variables
- Cannot be instantiated
- Methods are by default concrete unless specified to be `abstract`
- Can only implement one per class
### Packages
- namespace organizes classes and interfaces
- package name starts with website address
- import classes
    - `import java.util.Comparator;`
    - `import java.util.*;`
    - `import static org.junit.Assert.*`
### Custom libraries
- Add path in the `CLASSPATH` environment variable