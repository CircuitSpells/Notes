# SOLID Principles

## Single Responsibility Principle

`A class should have one and only one reason to change, meaning that a class should have only one job.`

KISS. Pull complex behavior into separate classes.

## Open-Closed Principle

`Objects or entities should be open for extension but closed for modification.`

This means that a parent class should be built to be easily extendable by child classes. A parent class shouldn't have to be modified to implement custom behavior for a child class.

## Liskov Substitution Principle

`Let q(x) be a property provable about objects of x of type T. Then q(y) should be provable for objects y of type S where S is a subtype of T.`

This is a really roundabout way of saying that a child class should be able to be replaced by its parent class without breaking behavior (with the exception of properties/methods custom to the child class).

## Interface Segregation Principle

`A client should never be forced to implement an interface that it doesn’t use, or clients shouldn’t be forced to depend on methods they do not use.`

This suggests that interfaces should be split up into their smallest logical parts such that the classes that implement them do not have to implement methods that they will not use.

## Dependency Inversion Principle

`Entities must depend on abstractions, not on concretions. It states that the high-level module must not depend on the low-level module, but they should depend on abstractions.`

Write abstractions to class dependencies using interfaces such that those classes could be replaced without affecting the implementation behavior.
