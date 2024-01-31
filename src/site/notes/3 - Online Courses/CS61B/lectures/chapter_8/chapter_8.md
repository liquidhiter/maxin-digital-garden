---
{"dg-publish":true,"permalink":"/3-online-courses/cs-61-b/lectures/chapter-8/chapter-8/","noteIcon":"","created":"2024-01-31T22:49:21.435+01:00","updated":"2024-01-31T22:51:13.055+01:00"}
---


# CS61B 2018 Spring Learning Notes - Chapter 8
## Topic: Encapsulation, Asymptotic and Amortized Analysis

- `“An engineer will do for a dime what any fool will do for a dollar” -- Paul Hilfinger`

### Efficient Programming
- programmer's perspective
    - cost of development and maintenance
- code's perspective
    - performance of the software (picee-wise and as a whole)
- encapsulation
- API's
    - API design and data structure selection can be an iterative proces
        - **Dont stick to getting the right answer before making it working**
        - refactor can help to get the right answer later
        - WORKING is the king in the early stage!
```java
Map<String, Map<Integer, Double>> someMap;
```
> Hide more information to avoid the complex data type specifications

```java
public class TimeSeries<T> {
    /*Underlying implementation utilize any Map*/
    public TimeSeries() {
        /*Do something here*/
    }
}

Map<String, TimeSeries<Double>> someMap;
```
- ADT's
    - `Is-A` vs `Has-A`
    - `java.util.Stack` is a bad design
    - `Circle-ellipse problem`: https://en.wikipedia.org/wiki/Circle%E2%80%93ellipse_problem
        - considering `circle` as the sub-class of `ellipse`
            - overriden methods are not meaningful for a `circle` object
    - more interesting example:
        - implementation inheritance breaks encapsulation
        ```java
        /* Defined in the father class */
        public void bark() {
            barkMany(1);
        }
        public void barkMany(int N) {
            for (int i = 0; i < N; i += 1) {
                System.out.println("bark");  
            }
        }

        /* Defined in the sub-class */
        @Override
        public void barkMany(int N) {
            System.out.println("As a dog, I say: ");
            for (int i = 0; i < N; i += 1) {
                bark();
            }
        }
        ```
        - subClassObj.barkMany(3) results in infinite loop
            - the implementation details are exposed to the client users
    - suggestions in `Effective Java`
        - It is safe to use inheritance within a package, where the subclass and the superclass are under the control   of the same programmers.
        - It is safe to extend classes specifically designed and documented for extension.
        - Inheriting from ordinary concrete classes across package boundaries is dangerous.


### Extension and Delegation
#### extension
```java
public class ExtensionStack<Item> extends LinkedList<Item> {
    public void push(Item x) {
        add(x);
    }
}
```
- inherit all methods from certain class and re-use all of them directly


#### delegation
```java
public class DelegationStack<Item> {
    private LinkedList<Item> L = new LinkedList<Item>();
    public void push(Item x) {
        L.add(x);
    }
}
```
- similar to extension, but the current class isn't considered as another version of the class from which the method is pulled 

### Views
**Don't think really understand this part**
![Alt text](image.png)
![Alt text](image-1.png)
![Alt text](image-2.png)
![Alt text](image-3.png)