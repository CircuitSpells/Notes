# DevLog

## Create Terraform Files From Pre-Existing Azure Resources

8/30/25

If you have resources in Azure created in Azure portal, you can use the `aztfexport` CLI tool to create new terraform files for you automatically:

```
winget install Microsoft.Azure.AztfExport
```

Make sure you're logged into Azure CLI and have proper permissions to pull resources, then create and enter an `infra` directory and run:

```
aztfexport resource-group rg-myproject
```

This will create the main.tf, provider.tf, and terraform.tf files for you.

## Add Snippets Using Key Commands in VS Code

7/18/25

VS Code allows for custom snippets which we can trigger with key commands. This example will add `<div>` tags around highlighted text when you use the key command `ctrl+alt+w`.

First, add the snippet in html.json by entering `ctrl+shift+p`, type/select "Snippets: Configure Snippets", then select "html". Add the following entry:

```json
"Wrap with <div>": {
  "prefix": "wrapdiv",
  "body": [
    "<div>",
    "  ${TM_SELECTED_TEXT}",
    "</div>"
  ],
  "description": "Wrap selection in a <div>…</div>"
}
```

To tie this snippet to a key command, enter `ctrl+shift+p` and type/select "Open Keyboard Shortcuts (JSON)" (make sure to select the "(JSON)" option), then add the following entry:

```json
{
  "key": "ctrl+alt+w",
  "command": "editor.action.insertSnippet",
  "args": {
    "name": "Wrap with <div>"
  },
  "when": "editorTextFocus && editorHasSelection"
}
```

Now in any html file, if you select text:

```html
<h1>Hello World</h1>
```

And enter `ctrl+alt+w`, the snippet will apply:

```html
<div>
  <h1>Hello World</h1>
</div>
```

## My Angular Startup (Update)

7/18/25

Since my last [Angular startup](#my-angular-startup), I've made modifications to more closely match Rainer Hahnekamp's suggestions [here](https://dev.to/this-is-angular/my-favorite-angular-setup-in-2025-3mbo), including an automatic flat structure, and the use of husky/lint-staged to enforce linting, testing, and running prettier before every commit. He also recommends using Sheriff for architecture testing to enforce boundary rules between modules.

My github repo for the Angular startup is [here](https://github.com/CircuitSpells/angular-starter). I've modified Rainer's repo a bit; I've updated to Angular v20, removed Tailwind, and changed from scss to css (raw CSS is extremely powerful if you know how to use it!).

## Enable Hibernate

6/16/25

If Hibernate is not available in the Windows power menu, you can enable it by opening a cmd prompt as an admin and running:

```
powercfg /hibernate on
```

To confirm:

```
powercfg /a
```

To make Hibernate show up on the Start menu:

- Go to Control Panel > Hardware and Sound > Power Options > Choose what the power buttons do.
- Click "Change settings that are currently unavailable".
- Check Hibernate under "Shutdown settings".
- Click Save changes.

## Javascript Event Loop

6/5/25

Fantastic explanation of the JS event loop:

https://www.youtube.com/watch?v=eiC58R16hb8

## Automatically Generate Release Notes

I created a few PowerShell scripts to automatically create release notes:

- [Get-NewCommits](https://github.com/CircuitSpells/PowerShellScripts/blob/main/cmdlets/Get-NewCommits.ps1)
- [ConvertTo-ReleaseNotes](https://github.com/CircuitSpells/PowerShellScripts/blob/main/cmdlets/ConvertTo-ReleaseNotes.ps1)

In the future I'd like to add the ability to parse and sort by [conventional commit](https://www.conventionalcommits.org/en/v1.0.0/) type.

## Good Demo on Flexbox

5/16/25

https://codepen.io/chriscoyier/pen/kNZRER

## My Angular Startup

5/14/25

Create a new Angular project:

```
ng new project-name --defaults --strict --inline-style --inline-template --skip-tests
```

This automatically inlines styles and templates into newly generated components, and test files will not be automatically generated. CSS is the style preference and there is no built-in server-side rendering. Lastly, the `--strict` flag enforces strict typescript rules.

New components can be generated like this:

```
ng g component components/component-name --flat
```

Note: `ng new` and `ng g` offer the `-d` flag which is the alias for `--dry-run`.

## Angular Signals Quick Reference

5/13/25

Create and listen to signals:

```ts
price: number = 123.45;

// Init signal
quantity: WriteableSignal<number> = signal<number>(1);

// Listen to a signal's updates, return a read-only signal
totalPrice: Signal<number> = computed<number>(
  () => this.price * this.quantity() // use parenthesis after a signal's name to get its value
);

// Listen to a signal's updates, do not return a new value. Use for logging and calling external APIs
effect(() => console.log(this.quantity()));
```

Modify signal values:

```ts
// Replace the signal's value
this.quantity.set(qty);

// Update the signal's value based on its current value
// Immutable; returns a brand-new value
this.quantity.update((qty: number) => qty * 2);

// Modify the content in place (not the value itself)
// Mutable; directly modify the existing object or array (do not use with primitive types--primitives are always immutable)
this.selectedItem.mutate((i: Item) => (i.price = i.price * 1.2));
```

Listen to signals in templates:

```html
<!-- Display the signal's current value and register the signal as a view dependency -->
<!-- If the signal value changes, the view is re-rendered -->
<div>Total: {{ totalPrice() }}</div>
```

## Angular/Typescript Generics Using the "keyof" and "extends keyof" Operators

5/13/25

Say we want to implement a generic update and logging method for the `snack` and `user` signals:

```ts
export class App {
  snack = signal<Snack>(SNACK);
  user = signal<User>(USER);

  updateSnack() {
    // generic update method here
    // generic logging method here
  }

  updateUser() {
    // generic method here
    // generic logging method here
  }
}

export const SNACK: Snack = {
  id: 1,
  name: "popcorn",
  price: 2.0,
  isInStock: true,
};
export const USER: User = { id: 5, name: "Bilbo", userName: "Hobbit1" };

export interface Snack {
  id: number;
  name: string;
  price: number;
  isInStock: boolean;
}
export interface User {
  id: number;
  name: string;
  userName: string;
}
```

The `keyof` operator refers to the properties on an object. Using this knowledge, instead of accessing the property via the usual `this.snack().price` syntax, we can instead use the `keyof` operator to get a generic property using the alternate `this.snack()['price']` syntax:

```ts
export function logSignal<T>(sg: Signal<T>, prop?: keyof T) {
  if (prop) {
    // if "prop" param was passed in
    console.log(sg()[prop]);
  } else {
    console.log(sg());
  }
}
```

`keyof` enforces compile-time type safety, so we cannot for instance call `logSignal(this.snack, 'userName')`, but we can call `logSignal(this.snack, 'price')` (note that the angular brackets aren't needed, e.g. `logSignal<Snack>(this.snack)`, because the type is implied).

Now we can try and make a method for updating a property. To update signals, use the `WritableSignal` class:

```ts
export function updateProperty<T>(
  sg: WritableSignal<T>,
  prop: keyof T,
  value: T[keyof T]
) {
  sg.update((obj) => ({
    ...obj,
    [prop]: value,
  }));
}
```

Here, `T[keyof T]` looks at all properties on the `T` object. If we passed in the `user` signal, this param would accept `number | string`, but not `null`, `boolean`, etc. This is almost what we want, but not quite; we could accidentally try and update a property with a value of the wrong type and the compiler would not try to stop us.

The fix for this is by using the `extends keyof` operator:

```ts
export function updateProperty<T, K extends keyof T>(
  sg: WritableSignal<T>,
  prop: keyof T,
  value: T[K]
) {
  sg.update((obj) => ({
    ...obj,
    [prop]: value,
  }));
}
```

Now, we get compile-time type safety for the values that are passed in. Finally, we can call these new generic methods:

```ts
export class App {
  snack = signal<Snack>(SNACK);
  user = signal<User>(USER);

  updateSnack() {
    updateProperty(this.snack, "name", "peanuts");
    logSignal(this.snack, "name");
  }

  updateUser() {
    updateProperty(this.user, "name", "Frodo");
    logSignal(this.user);
  }
}
```

## .NET DateTimeProvider

5/12/25

.NET 8 introduced a built-in `TimeProvider` class so that you do not have to make your own time provider abstraction:

```C#
// Program.cs
builder.Services.AddSingleton<TimeProvider>(TimeProvider.System);
```

Then in your service, inject the `TimeProvider`:

```C#
public sealed class ReportService
{
    private readonly TimeProvider _timeProvider;

    public ReportService(TimeProvider timeProvider) => _timeProvider = timeProvider;

    // business rule: a report is stale after 24 h
    public bool IsReportStale(DateTimeOffset lastRunUtc) =>
        _timeProvider.GetUtcNow() - lastRunUtc > TimeSpan.FromHours(24);
}
```

In your tests, you can use the `Microsoft.Extensions.TimeProvider.Testing` package to deterministically alter the time:

```C#
[Fact]
public void IsReportStale_Should_ReturnFalse_When_ReportIsOlderThan24Hours()
{
    var fakeClock = new FakeTimeProvider(DateTimeOffset.Parse("2025‑01‑01T00:00:00Z"));
    var reportService = new ReportService(fakeClock);

    // 1 h later – should still be fresh
    fakeClock.Advance(TimeSpan.FromHours(1));
    Assert.False(reportService.IsReportStale(fakeClock.Start));

    // 25 h later – now stale
    fakeClock.Advance(TimeSpan.FromHours(24));
    Assert.True(reportService.IsReportStale(fakeClock.Start));
}
```

Here is how to replace the `TimeProvider` Singleton for WebApplicationFactory:

```C#
public sealed class ApiFactory : WebApplicationFactory<Program>
{
    public FakeTimeProvider Clock { get; } =
        new(DateTimeOffset.Parse("2025‑01‑01T00:00:00Z"));

    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(s =>
        {
            s.RemoveAll<TimeProvider>();
            s.AddSingleton<TimeProvider>(Clock);
        });
    }
}
```

```C#
public class ReportTests : IClassFixture<ApiFactory>
{
    private readonly ApiFactory _factory;

    public ReportTests(ApiFactory factory) => _factory = factory;

    [Fact]
    public async Task Report_Should_BeInvalid_When_ReportIsOlderThan24Hours()
    {
        var client = _factory.CreateClient();

        // reset clock before each test
        _factory.Clock = DateTimeOffset.Parse("2025‑01‑01T00:00:00Z");

        // ...

        _factory.Clock.Advance(TimeSpan.FromHours(25));

        // ...
    }
}
```

Note that race conditions will not be an issue here; in xUnit, tests in the same test fixture run in serial, and tests in a separate test fixture will get their own separate factory (and therefore separate clock) instance.

## Alternatives to DateTime in .NET

5/12/25

As of .NET 6, the `DateTime` class is largely unnecessary. Instead use:

- `DateOnly` for pure dates
- `TimeOnly` for pure times
- `DateTimeOffset` for any absolute timestamp

`DateTimeOffset` provides a built-in UTC offset value for every timestamp, removing any ambiguity about which timezone the timestamp is in:

```C#
DateTime dtUtc = DateTime.UtcNow;
DateTimeOffset dtoUtc = DateTimeOffset.UtcNow;

Console.WriteLine(dtUtc); // 05/12/2025 10:30:25
Console.WriteLine(dtoUtc); // 05/12/2025 10:30:25 +00:00
```

Other ways of setting `DateTimeOffset`:

```C#
// Create with local offset
var dtoLocal = new DateTimeOffset(2025,5,12,10,30,25, TimeZoneInfo.Local.GetUtcOffset(DateTime.Now));

// Create explicitly with offset
var dtoUtcMinus7 = new DateTimeOffset(2025,5,12,10,30,25, TimeSpan.FromHours(-7));

// Compare instants
bool isSameInstant = dtoLocal.UtcDateTime == dtoUtcMinus7.UtcDateTime;
```

## New Global Error Handling in .NET 8

5/9/25

Instead of using `app.UseMiddleware<>();`, .NET 8 introduced a new way of global error handling via the `IExceptionHandler` interface. It works largely the same, however you do not need to handle any delegates; the error handling works automatically.

Here's how it works:

`GlobalExceptionHandler.cs`

```C#
public class GlobalExceptionHandler : IExceptionHandler
{
    private readonly ILogger<GlobalExceptionHandler> _logger;

    public GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger)
    {
        _logger = logger;
    }

    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        _logger.LogError(exception, "Exception occurred: {Message}", exception.Message);

        var problemDetails = new ProblemDetails
        {
            Status = StatusCodes.Status500InternalServerError,
            Title = "Server Error",
            Type = "https://datatracker.ietf.org/doc/html/rfc7231#section-6.6.1"
        }

        httpContext.Response.StatusCode = StatusCodes.Status500InternalServerError;

        await httpContext.Response.WriteAsJsonAsync(problemDetails, cancellationToken);

        return true;
    }
}
```

`Program.cs`

```C#
// ...

builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();

// ...

app.UseExceptionHandler();

// ...
```

## Integration Testing Using Docker In .NET

4/30/25

Integration Testing Using Docker In .NET: https://www.milanjovanovic.tech/blog/testcontainers-integration-testing-using-docker-in-dotnet

## CQRS and MediatR

4/30/25

- Article on CQRS and MediatR: https://www.milanjovanovic.tech/blog/cqrs-pattern-with-mediatr
- Mediator design pattern: https://refactoring.guru/design-patterns/mediator
- Interesting article series on the pitfalls of MediatR, and alternative solutions: https://arialdomartini.github.io/mediatr

## Angular Extensions

1/16/25

- Angular Language Service

## Angular Optimizations

12/18/24

Angular optimizations can sometimes have negative and unpredictable behavior. They can be enabled in the build configuration in `angular.json` as well as by setting the `production` boolean in the `environment.ts` file to `true` in order to call `enableProdMode()`. Make sure that if optimizations are turned on for production that they are turned on for lower environments as well so that no unexpected differences occur in the prod build.

## Check Equality for Collections of Different Types

11/27/24

The below `CheckEquality` method is a handy tool for tests to check if two collections of different types are equivalent (note that this uses the FluentAssertions package):

```C#
using FluentAssertions;
using System;
using System.Collections.Generic;
using System.Linq;

namespace IntegrationTests.Utilities

internal static class EqualityComparer
{
    internal static void CheckEquality<TExpected, TActual>(
        List<TExpected> expectedList,
        List<TActual> actualList,
        Func<TExpected, object> getExpectedCompareProperty,
        Func<TActual, object> getActualCompareProperty,
        Action<TExpected, TActual> checkItemEquality)
    {
        expectedList.Should().NotBeNull();
        actualList.Should().NotBeNull();
        foreach (var expectedItem in expectedList)
        {
            TActual actualItem = actualList.FirstOrDefault(actual => getExpectedCompareProperty(expectedItem).Equals(getActualCompareProperty(actual)));
            actualItem.Should().NotBeNull("actual and expected should have matched on a property but failed");

            checkItemEquality(expectedItem, actualItem);

            actualList.Remove(actualItem);
        }
    }
}
```

Note that the order of each item in the collection does not matter, however the number of items in each collection needs to match. The items are matched on an initial "compare property" (usually an id or other unique identifier), then those matched items are compared by a delegate that you write.

Usage:

```C#
List<SomeType> collectionExpected = new()
{
    // ...
};
List<SomeOtherType> collectionActual = new()
{
    // ...
};

EqualityComparer.CheckEquality(
    collectionExpected,
    collectionActual,
    expected => expected.Id,
    actual => actual.SomeOtherId,
    (expected, actual) =>
    {
        actual.SomeProperty.Should().Be(expected.SomeOtherProperty);
        actual.SomeProperty2.Should().Be(expected.SomeOtherProperty2);
    });
```

## AddSingleton, AddScoped, AddTransient

11/1/24

### AddSingleton

Singleton services live for the lifetime of the application. They are instantiated upon the first request to the API unless you specify to immediately create them when adding the service. They are not thread-safe and can be accessed concurrently. They will not let go of memory unless explicitly told to. Make sure to design the service to be stateless or implement locks for shared resources. Do not inject scoped or transient services into the singleton.

Common singleton services: loggers, database caching, configuration providers

### AddScoped

Scoped services last for the lifetime of a request, which is useful when a service is dependency injected in multiple classes such as the controller class and a service class. If said service has a state change in the controller, and again in a service called by the controller, they will be accessing the same service instance.

Common scoped services: database contexts

### AddTransient

Transient services last for the lifetime of the service scope, similar to instantiating a new instance of a class. If two classes dependency inject the same transient service, they will be two separate instances.

Common transient services: data processing, validators, HttpClient

### Bonus: HttpClientFactory

HttpClientFactory is a common service to create HttpClients in order to make http requests. The HttpClientFactory is a singleton, and the generated HttpClients are transient services. The HttpClientFactory does not need to be registered explicitly; instead, when you register the HttpClient, an HttpClientFactory singleton will be created if one doesn't already exist.

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

C# does not do this. non-nullable primitives have the same behavior as C in regards to their nullability. However, `int?` is just syntactic sugar for `Nullable<int>`, which is a struct that also holds a boolean value for whether or not the int value is valid (aka "null"). C# objects on the other hand _do_ have pointers referencing their data, and those pointers will point to a null reference when that object is null. What this means is that `int? foo = null;` and `MyClass myClass = null` are not actually doing the same thing under the hood.

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
- ChatGPT 4 is only aware of C# 11 and .NET 7, however it _can_ be prompted to search the internet for newer versions.
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
