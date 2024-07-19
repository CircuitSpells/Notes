# Design Patterns

Each pattern introduces their own *design principles*, however these principles apply to all other patterns and are intended to be a method of thinking about design patterns as a whole.



## Strategy Pattern

The Strategy Pattern defines a family of algorithms, encapsulates each one, and makes them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

### Example: DuckSim game

We have a game that simulates ducks such as mallards and redhead ducks, but also rubber ducks and decoy ducks. We want to design this game to allow for it to be updated forever, so the implementation shouldn't require the entire code to change when a duck needs to adopt new behavior, or a new type of duck is added.

A naive approach might be to implement a Duck abstract base class for shared behavior, interfaces for non-shared behavior, and then have the child duck classes implement the interfaces directly. One issue with this is if two ducks implement their interfaces exactly the same, thus duplicating code. Another issue is that the behavior defined by the interfaces cannot change at runtime, which prevents feature updates such as allowing a decoy duck to pick up a rocket and begin flying mid-game.

Implementation:
- Duck abstract base class for shared behavior (swim(), display())
- *Behavior* interfaces for non-shared behavior:
    - IFlyBehavior (requires fly())
        - Fly classes that implement IFlyBehavior: FlyWithWings, NoFly
    - IQuackBehavior (requires quack())
        - Fly classes that implement IQuackBehavior: Quack, Squeak, SilentQuack
- Now, the Duck abstract base class can have IFlyBehavior and IQuackBehavior properties (with setters) and methods to encapsulate the calling of the interface methods. Then, the child classes can each set their own behavior. This also means that the behavior can change at runtime, so the decoy duck could suddenly set its FlyBehavior from NoFly to RocketFly if we later wanted to implement that into the game.

### Design Principles

- Identify the aspects of your application that vary and separate them from what stays the same.
    - Don't stick everything into a single base class; abstract behavior out.
- Program to an interface, not an implementation.
    - Prefer Has-A relationships over Is-A relationships.
- Favor composition over inheritance.
    - Has-A relationships allow for the composition of more complex behavior.



## Observer Pattern

The Observer Pattern defines a one-to-many dependency between objects so that when one object changes state, all of its dependents are notified and updated automatically.

Typically, the "subject" is the "one", and the "observer" is the "many".

### Example: Weather App

We need a weather app that pulls latest temperature, humidity, and pressure readings from a weather station, and conveys that info to three different weather displays. We'd also like the option to add displays down the line.

(one such) implementation:
- ISubject
    - addObserver(IObserver o), removeObserver(IObserver o), notifyObservers()
- WeatherData implements ISubject
    - ISubject methods
    - An array list of observers
    - getTemp(), getHumidity(), getPressure(), measurementsChanged()
        - If we add measurements down the line, they can be easily added to this class.
- IObserver
    - update()
- CurrentConditionsDisplay, StatisticsDisplay, and ForecastDisplay all implement IObserver, as well as some IDisplay interface
    - Additional displays can implement the IObserver and IDisplay interfaces

### Design Principles

- Strive for loosely coupled designs between objects that interact.