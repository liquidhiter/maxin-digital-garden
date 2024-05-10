---
{"dg-publish":true,"permalink":"/05-online-courses/c00-cs-61-b/lectures/cs-61-b-2018-spring-learning-notes-chapter-07/","noteIcon":"","created":"2024-01-31T22:49:21.403+01:00","updated":"2024-01-31T22:56:46.755+01:00"}
---

## Topic: Package and Access Control

### Package
- namescope used to organize classes and interfaces to avoid ambiguity
- package name starts with the website address by convention
    - e.x. `ug.josh.animal; // where josh.ug is the website`
- access class from whithin the same package
    - `Animal a = new Animal(...);`
- access class from outside the package
    - `ug.josh.animal.Animal a = new ug.josh.animal.Animal(...);`
    - alternative
        - `import ug.josh.animal;`

#### Default Package
- Java class without explicit package is condsidered to be part of the `default package`
- code from the default package can't be imported
- risk of having class name collisions

### Access
- private: only visible in the same class
- package private (default): classes belonging in the same package can access
- protected: classes within the same package and subclasses can access
- public: all classes can access

![access table](image.png)

- none access modifier decorated memebers in classes part of the default package are accessible between all `default-package` classes
- **default access for methods in the interface are public**
- access depends only on the static types (???)
