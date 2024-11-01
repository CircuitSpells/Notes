# DevLog

## Useful Tools

11/1/24

In addition to the [Fluent libraries](#fluent-libraries) mentioned below, here are some more useful tools:
- [Bogus](https://github.com/bchavez/Bogus) (quickly create fake objects with realistic fake data)
    - Advice: When using RuleFor() for a string, use `f.Random.Words()` instead of `f.Random.String()`. The latter generates random garbage text that sometimes has issues during json serialization/deserialization. The former generates a few random english words.
- [WireMock](https://github.com/WireMock-Net/WireMock.Net) (mock external APIs in-box)
    - The docs on [stubbing](https://github.com/WireMock-Net/WireMock.Net/wiki/Stubbing) are good place to start.

## PowerShell Policies

7/18/24

```pwsh
Set-ExecutionPolicy Restricted # Will not allow any powershell scripts to run.  Only individual commands may be run.

Set-ExecutionPolicy AllSigned # Will allow signed powershell scripts to run.

Set-ExecutionPolicy RemoteSigned # Allows unsigned local script and signed remote powershell scripts to run.

Set-ExecutionPolicy Unrestricted # Will allow unsigned powershell scripts to run.  Warns before running downloaded scripts.

Set-ExecutionPolicy Bypass # Nothing is blocked and there are no warnings or prompts.
```

To change the policy only in the current session:
```pwsh
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process
```

## Fluent Libraries

7/17/24

Fluent makes some really great and robust libraries for common needs in a .NET API:
- [FluentValidation](https://docs.fluentvalidation.net/en/latest/)
- [FluentAssertions](https://fluentassertions.com/)
- [FluentResults](https://github.com/altmann/FluentResults)

## Article on Logging with Message Templates

7/9/24

[Good article on logging with message templates (the bottom of the article shows loggers that support message templates)](https://messagetemplates.org/)

## Gitflow: Merging Dev into Main Without Conflicts

1/26/24

Azure DevOps can usually handle conflicts with Dev in Main okay because it allows you the option to "take source" or "take target" when merging. However, GitLab doesn't offer this nicety, and even the way Azure DevOps handles it under the hood probably isn't doing what you expect.

So, ideally, what you should do is merge main into dev right after merging dev into main. It should look something like this:
- Make a PR for dev -> main
- Merge dev -> main
- Locally, pull dev and main
- Locally, create a branch off of dev, say "merge-main-into-dev"
- Locally, merge main into "merge-main-into-dev"
- Create a PR for "merge-main-into-dev" -> dev

If you don't do this, you'll end up with a lot of conflicts. To resolve these:
- Locally, pull dev and main
- Locally, create a branch off of dev, say "dev-clone"
- run the following: `git merge -s ours main`
  - this will ignore everything from main, and take everything from "dev-clone"
- Make an MR for "dev-clone" -> dev
- Merge "dev-clone" -> dev

Remember how I mentioned earlier that Azure DevOps "take source" and "take target" might not actually behave in the way that you expect? I believe this is because what Azure DevOps does is this, with the `main` branch checked out: `git merge --strategy-option theirs`. What this does is instead of ignoring everything from main, it merges everything it can, and then takes everything from dev that creates conflicts. I imagine the nuance in "merge everything it can" and "take everything from dev no matter what" is a subtle difference that might actually have real world implications. I haven't tested it, so can't say for sure though.

## C# Nullability and Boxing: What Happens Under the Hood

1/21/24

In a language like C, primitive data types are unable to be null. This is because all bits for that type are dedicated to storing the value of the type in question. One way around this is to store a pointer to that data type, and if you need the value to be null, you can point the pointer to a null reference.

C# does not do this. non-nullable primitives have the same behavior as C in regards to their nullability. However, `int?` is just syntactic sugar for `Nullable<int>`, which is a struct that also holds a boolean value for whether or not the int value is valid (aka "null"). C# objects on the other hand *do* have pointers referencing their data, and those pointers will point to a null reference when that object is null. What this means is that `int? foo = null;` and `MyClass myClass = null` are not actually doing the same thing under the hood.

In C#, the exception to this is "boxing". Boxing occurs when a primitive type needs to be stored on the heap. That primitive type is wrapped in an `object`, and a pointer references that object as if it were any other class.

The most basic example of this is when an `object` type references a primitive type:
```C#
int myInt = 10;
object myObject = myInt; // Boxing occurs here
```

And similarly:
```C#
void Display(object obj) { /* ... */ }

int myInt = 10;
Display(myInt); // Boxing occurs here
```

Recall that in addition to primitives, structs are also value types. Boxing can occur for structs under certain conditions, such as casting the struct to an interface:
```C#
struct MyStruct : IMyInterface
{
    // ...
}

MyStruct myStruct = new();
IMyInterface myInterface = myStruct; // Boxing occurs here
```

Fun fact: before C# supported generic types (pre C# version 2.0), C# used the much more aptly named `ArrayList` and `HashTable` types. However, because these types didn't use generics and instead treated all values as an `object` type, boxing would occur for every item in the collection, causing a notable performance impact. These non-generic collections, while still usable, are now considered deprecated and should not be used.

However, even generics have gotchas. For example, if the generic uses behavior specific to reference types (e.g. `object.Equals()`), then boxing will occur:
```C#
void MyGenericMethod<T>(T item)
{
    Console.WriteLine(item.Equals(null)); // Boxing if T is a value type
}

MyGenericMethod(myInt); // Boxing occurs here if T is a value type
```

Boxing can have performance implications, especially in high-performance or memory-sensitive applications, due to the overhead of heap allocation and the subsequent garbage collection. It's often recommended to be mindful of boxing and avoid it when possible, especially in performance-critical code paths.

## Memoization

1/20/24

Memoization is an optimization that caches expensive function calls. This is usually in the form of using a hash table to store the function's arguments as a key, and the function's return as the value. This can be particularly useful for recursion.

One example where this is useful is the Fibonacci sequence. Here's the naive approach:
```Python
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)
```

Using memoization:
```Python
def fibonacci(n, memo={}):
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fibonacci(n - 1, memo) + fibonacci(n - 2, memo)
    return memo[n]
```

## Dynamic Programming

1/20/24

Memoization is a form of dynamic programming. Dynamic programming is the concept of breaking a problem down into smaller parts. Those smaller parts combine together to create the solution. Typically, dynamic programming involves the process of having to compute the same thing more than once, which is why memoization is such a useful technique.

## CLI For Quick C# Packages & Tests

1/14/24

Install nuget CLI:
```
winget install NuGet -e
```

Search packages:
```
nuget search MyPackage
```

Install MSTest:
```
dotnet add MSTest.TestFramework
```

Use MSTest:
```C#
using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace MyNamespace;

[TestClass]
public class MyClassNameTests
{
    [TestMethod]
    public void MyMethod_ShouldSucceed_WhenThisHappens()
    {
        int foo = 42;
        int bar = MyMethod(foo);

        Assert.IsNotNull(bar)
        Assert.AreEqual(foo, bar);
    }
}
```

## AI Awareness of C#/.NET

1/14/24

As of 1/14/24:
- ChatGPT 3.5 is aware of C# 10 and .NET 6. It cannot search the internet for updates.
- ChatGPT 4 is only aware of C# 11 and .NET 7, however it *can* be prompted to search the internet for newer versions.
- CoPilot, while backed by ChatGPT 4, is only aware of C# 9 and .NET 5. It cannot search the internet for updates.

## C# pass by value, reference, and the equivalent of C++ "const reference"

1/14/24

Consider the following scenarios:
  1. A value type is "passed by value"
  2. A value type is "passed by reference"
  3. A reference type is "passed by value"
  4. A reference type is "passed by reference"

Recall that value types include primitives and structs, whereas reference types include objects (the simplest test to determine if a type is value or reference is to determine of that type can be `null` without using the `Nullable<t>` class, a.k.a. the `?` operator).

Passing a value type by value:
```C#
void PassValueTypeByValue(int x)
{
    x = 42; // value *is not* modified outside method's scope.
}
```

Passing a value type by reference:
```C#
void PassValueTypeByReference(ref int x)
{
    x = 42; // value *is* modified outside of the method's scope.
}
```

Passing a reference type by value:
```C#
void PassReferenceTypeByValue(MyClass myClass)
{
    myClass.Foo = 42; // value *is* modified outside of the method's scope.

    // The following *is not* modified outside of the method's scope:
    MyClass anotherClass = new()
    {
        Foo = 101
    };
    myClass = anotherClass;

    // the outer class's Foo property will have a value of 42
}
```

Passing a reference type by reference:
```C#
void PassReferenceTypeByReference(ref MyClass myClass)
{
    myClass.Foo = 42; // value *is* modified outside of the method's scope.

    // The following *is* modified outside of the method's scope:
    MyClass anotherClass = new()
    {
        Foo = 101
    };
    myClass = anotherClass;

    // the outer class's Foo property will have a value of 101
}
```

Sometimes, pass by reference behavior is needed while also promising that the argument's value won't change, such as const references in C++.

In C#, there are two ways to achieve the C++ "const reference" behavior:

1. Using the `in` keyword:
```C#
void PassByReadonlyReferenceWithIn(in int x) // x is passed by reference through a temporary variable
{
    x += 1; // invalid: compiler error.
}

// at the call site:
PassByReadonlyReferenceWithIn(y); // the compiler decides whether to pass by value or reference. if y is an int, it will be passed by reference.
// or 
PassByReadonlyReferenceWithIn(in y); // ensures the argument is passed by reference. Throws a compilation error if y is, say, a short
```

1. Or as of C# 12, using `ref readonly`:
```C#
void PassTByReadonlyReference(ref readonly int x) // x is passed by reference
{
    x += 1; // invalid: compiler error.
}

// at the call site:
PassTByReadonlyReference(in y); // argument is a variable, and is writable
// or
PassTByReadonlyReference(ref z);
```

Note that these are toy examples; because `int` is no larger than a reference in most modern machines, there is no benefit to passing a single `int` as a readonly reference.

The use case for `in` or `ref readonly`: you have a very large struct. `ref readonly` was created in part to account for readonly structs, which couldn't previously take advantage of the `in` keyword because structs are not variables.

## C# Delegates

1/13/24

C# delegates let you save functions as variables:

```C#
// Define a delegate with a specific signature
public delegate void MyDelegate(string message);

class Program
{
    static void Main()
    {
        // Create an instance of the delegate, pointing to a method
        MyDelegate myDelegate = new MyDelegate(PrintMessage);

        // Invoke the delegate, which calls the referenced method
        myDelegate("Hello, delegates!");

        // You can also use the shorthand syntax
        MyDelegate shorthandDelegate = PrintMessage;
        shorthandDelegate("Using shorthand syntax!");

        // Delegates can be multicast (refer to multiple methods)
        MyDelegate multicastDelegate = PrintMessage;
        multicastDelegate += PrintAnotherMessage;
        multicastDelegate("Multicast delegates!");

        // Remove a method from the multicast delegate
        multicastDelegate -= PrintMessage;
        multicastDelegate("After removing a method.");
    }

    // Methods that match the delegate signature
    static void PrintMessage(string message)
    {
        Console.WriteLine(message);
    }

    static void PrintAnotherMessage(string message)
    {
        Console.WriteLine("Another message: " + message);
    }
}
```

`Action` and `Func` are generic delegate types. `Action` is for methods that return void, and `Func` is for all other methods.

`Action`:

```C#
Action<string> printMessage = (message) => Console.WriteLine(message);

// Invoke the Action
printMessage("Hello, Action!");
```

`Func` (the last value is the return value):

```C#
Func<int, int, int> addNumbers = (a, b) => a + b;

// Invoke the Func
int result = addNumbers(3, 5);
Console.WriteLine("Result of addition: " + result);
```

Event Handlers use delegates under the hood. Events in C# use the `event` keyword, and can be used like so:

```C#
using System;

class EventPublisher
{
    public event EventHandler MyEvent;

    public void TriggerEvent()
    {
        // Invoke() calls the delegate method
        // it is equivalent to calling the delegate directly
        MyEvent?.Invoke(this, EventArgs.Empty);
    }
}

class Program
{
    static void Main()
    {
        EventPublisher publisher = new EventPublisher();

        publisher.MyEvent += MyEventHandler;

        publisher.TriggerEvent();
    }

    static void MyEventHandler(object sender, EventArgs e)
    {
        Console.WriteLine("Event handled!");
    }
}
```

Though not utilized in this example, `object sender` and `EventArgs e` are standard arguments in event handlers:
- The `object sender` parameter refers to the object that triggered the event (in this case the EventPublisher object). This is useful when the event handler is used by multiple objects, e.g. multiple buttons in a GUI do the same thing.
- `EventArgs` may contain additional information about the event. While EventArgs itself may not contain much information, it is a common practice to use a derived class when more specific data needs to be passed to the event handler.

## Minimal APIs in .NET

1/11/24

[Great video on minimal APIs](https://www.youtube.com/watch?v=pYl_jnqlXu8)