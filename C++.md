# C++ Notes

Table of Contents:

- [C++ Notes](#c-notes)
  - [6: How the C++ Compiler Works](#6-how-the-c-compiler-works)
  - [7: How the C++ Linker Works](#7-how-the-c-linker-works)
  - [13: Best Visual Studio Setup for C++ Projects](#13-best-visual-studio-setup-for-c-projects)
  - [16: Pointers](#16-pointers)
  - [17: References](#17-references)
  - [21: The ```static``` Keyword](#21-the-static-keyword)
  - [27: ```virtual``` Functions](#27-virtual-functions)
  - [28: Interfaces (Pure Virtual Functions)](#28-interfaces-pure-virtual-functions)
  - [30: Arrays](#30-arrays)
  - [31: Strings](#31-strings)
  - [32: String Literals](#32-string-literals)
  - [33: The ```const``` Keyword](#33-the-const-keyword)
  - [34: The ```mutable``` Keyword](#34-the-mutable-keyword)
  - [48: Local ```static``` Variables](#48-local-static-variables)
  - [49: How to Use Libraries (Static Linking)](#49-how-to-use-libraries-static-linking)
  - [55: Macros](#55-macros)
  - [56: The ```auto``` Keyword](#56-the-auto-keyword)

## 6: How the C++ Compiler Works

For an application to be generated, first the C++ files have to be compiled and then linked.

The C++ compiler converts all cpp files into their own **translation unit**. Each translation unit results in an object file (aka ".obj" files). If cpp files ```#include``` other cpp files, **they will be treated as one large file and therefore result in one translation unit.**

Before these object files are created, the compiler goes through a pre-processor stage which runs through all pre-processor statements (```#include```, ```#define```, ```#pragma```, ```#if```, ```#ifdef```, etc.).

```text
Tip: You can request the pre-processor generate a file of all items that were pre-processed, as well as the order they were processed in. To generate this file in Visual Studio:

Right-click the project > properties (make sure you're editing the desired configuration and platform) > click the "C/C++" dropdown > Preprocessor > set "Preprocess to a File" to "Yes" > click "OK". Build the project and check the output directory for the generated .i file. Note that enabling this means an .obj file will not be produced.
```

The generated .obj file is in binary and is not human readable. However, it is possible to enable a setting to generate an assembly (".asm") file in Visual Studio: Right-click the project > Properties (make sure you're editing the desired configuration and platform) > click the "C/C++" dropdown > Output Files > set "Assembler Output" to "Assembly Only Listing" > click "OK". Build the project and check the output directory for the generated .asm file. Note that compiling in Debug mode by default will *not* enable any compiling optimizations.

To enable optimizations in Debug mode in Visual Studio: Right-click the project > Properties > set "Configuration" to "Debug" > click the "C/C++" dropdown > Optimization > set "Optimization" to "Maximize Speed" > back to the "C/C++" dropdown > Code Generation > set "Basic Runtime Checks" to "Default" (this disables runtime checks. Runtime checks insert files to assist with debugging) > click "OK".

---

## 7: How the C++ Linker Works

After the compilation process, the linking process can begin. The linker links together all .obj files into one program, as well as linking the entry point function (almost always ```main()```).

The linking stage also ties function declarations to their respective definitions. This is important to keep in mind and is largely the reason .h files should ideally only contain function declarations; if a .h file contains a function definition and that .h file is ```#include```d in multiple .cpp files, the pre-compiler will have placed that function definition in both .cpp files. The linker will then throw an error that there are multiple function definitions as it is ambiguous which definition should be used when that function is called.

```text
Tip: Compiler errors will begin with "C" and linking errors will begin with "LNK".
```

---

## 13: Best Visual Studio Setup for C++ Projects

In VS, create a new empty C++ project. Right click the project and select "Open Folder in File Explorer" to see the actual directory structure. Note that the solution and the project have the same name, which isn't ideal if there will be more than one project in a solution.

In the project folder, VS creates a .vcxproj file and a .filters file. The filters file holds virtual folders. These folders appear to exist in Visual Studio even though in reality the directory structure is flat. This can trick you into thinking your files are organized when in reality they're not.

To see the actual folder structure in VS, make sure to click the "Show all files" icon. Now when creating folders in VS, folders will actually be created as you expect.

In the project folder, create a folder called "src" for all code files to live in. Then create Main.cpp in this folder.

When building a project, by default VS puts the intermediate files into a folder called "Debug" contained in the project folder, then places the executable in *another* folder called "Debug" that is contained in the same directory as the solution file. This is garbage.

In VS, right click the project and select "Properties". Note the ```Output Directory``` and ```Intermediate Directory``` fields. By default, all paths defined in these fields are relative to the *project* directory unless specified otherwise.

To edit these values, first select "All configurations" from the Configuration dropdown menu, then select "All platforms" from the Platform dropdown menu. Then, set ```Output Directory``` to ```$(SolutionDir)bin\$(Platform)\$(Configuration)\```. Then, set ```Intermediate Directory``` to ```$(SolutionDir)bin\intermediates\$(Platform)\$(Configuration)\```. If the project has already been built before making these changes, go into file explorer and delete the old output and intermediate directories manually as cleaning the solution does not get rid of them.

To see the available macros or edit their values, click the down arrow next to the value on a field > Edit > Macros.

---

## 16: Pointers

A pointer is an integer that holds a memory address. The pointer's type is the type of data that (presumably) resides at the memory address that the pointer is referencing.

Traditionally the ```NULL``` macro was used when pointers needed to point to null. However, C++11 introduced the ```nullptr``` keyword as an alternative which is now the standard.

How to set a pointer's value:

```C++
int var = 8;
int* ptr = &var; // "&" gets the address of var
```

How to read a pointer's value:

```C++
cout << ptr << endl; // prints the address that ptr is holding
cout << *ptr << endl; // dereferences ptr, prints the value contained at ptr's held address
```

How to write over a pointer's dereferenced value:

```C++
*ptr = 10;
```

The pointer's in the examples above have been created on the stack. Pointer's can also be allocated on the heap:

```C++
char* buffer = new char[8]; // buffer now holds the memory address of the first array item
memset(buffer, 0, 8); // memset() will fill 8 bytes of data with zero, starting at the address contained at buffer

delete[] buffer; // delete the allocated memory
```

Pointers can also point to other pointers:

```C++
char* buffer = new char[8];
memset(buffer, 0, 8);

char** ptr = &buffer;

// get the memory address of buffer:
cout << &buffer << endl;
// or
cout << ptr << endl;

// get the memory address of the first array item:
cout << buffer << endl;
// or
cout << *ptr << endl;

// get the value of the first array item:
cout << *buffer << endl;
// or
cout << **ptr << endl;

delete[] buffer;
```

To view data at a certain memory location in Visual Studio:

1. Create a breakpoint just after the pointer is assigned
2. Hover mouse over variable, copy memory address
3. Go to: Debug > Windows > Memory > Memory 1
4. Paste in the "Address:" field, remove copied info that isn't just the memory address (you can also type the name of the pointer variable!)

For a 4 byte data type like ```int```, the first 4 hexadecimal values will be the integer data contained at that address. The left-most byte is the lower order byte (smaller values) nad the right-most byte is the higher order byte (larger values).

---

## 17: References

References must reference a pre-existing variable. They do not copy memory, but instead are directly referencing another value. References can only be set once, and must immediately be assigned a value upon declaration.

```C++
int a = 5;
int& ref = a; // note the "&" is part of the type in this case
```

In the above example, ```ref``` is just an alias for ```a```. The compiler will not create a new variable for ```ref```; they are literally the same thing.

The power of references occurs when creating functions. Consider The ```Increment()``` function below. It is pass by value, and therefore makes a copy of ```int value``` before executing the function body:

```C++
void Increment(int value) { value++; }
// ...
int a = 5;
Increment(a);
LOG(a); // prints "5"
```

```value``` will only increment in the the ```Increment()``` function body. Its value will not carry over to ```a``` because ```value``` and ```a``` are effectively two entirely different variables. To change ```a```'s value, we can reference ```a``` using pointers:

```C++
void Increment (int* value) { (*value)++; } // note the order of operations; the dereference must occur before the increment
// ...
int a = 5
Increment(&a);
LOG(a); // prints "6"
```

However, there is an alternate syntax to this method. Instead of ```value``` being a pointer, make ```value``` a reference instead:

```C++
void Increment (int& value) { value++; }
// ...
int a = 5
Increment(a);
LOG(a); // prints "6"
```

This does exactly the same thing as the example above it with the advantage of not having to explicitly dereference the pointer every time the underlying value needs to be read or modified.

---

## 21: The ```static``` Keyword

The ```static``` keyword has two meanings depending on the context it is used in; one applies to when it is used *outside* a class or struct and the other applies when it is used *inside* a class or struct.

Outside a class/struct: the linkage of the symbol you declare to be static is going to be internal; that is, it will only be visible to the translation unit that it is defined in (see chapter [7: How the C++ linker works](#7-how-the-c-linker-works)).

Normally, if two separate .cpp files define some global variable with the same name and type, the linker would complain (even if neither .cpp file ```#include```s each other). However, if the ```static``` keyword is used on one or both, the linker will treat the variables as two separate entities (If you actually did want both the global variables to be the same, you could prepend one with the ```extern``` keyword, which promises the compiler that the variable is defined externally in another translation unit). In general, it is best to use the ```static``` keyword whenever possible in this context so as to prevent unnecessary clashing of global variables across translation units.

Inside a class/struct: only one instance of the static variable/method will be created in memory. All instances of the class/struct that the static variable/method is defined in will use that same memory location.

---

## 27: ```virtual``` Functions

```virtual``` functions allow for child classes to ```override``` that function's definition.

```C++
class Entity
{
public:
    virtual std::string GetName() { return "Entity"; }
}

class Player : public Entity
{
public:
    Player(const std::string& name) : m_Name(name) {}

    std::string GetName() override { return m_Name; }
}
```

Note that the ```override``` keyword here *is not necessary* and everything will behave the same without it. ```override``` is an addition from C++11 that exists solely for the purpose of verbosity. That being said, it is best practice to include the ```override``` keyword wherever applicable. 

It is also important to note that overriding only takes effect *if the child object is being treated as a parent type* (aka polymorphism). If the child is of its own type (that is, ```Player``` object is of type ```Player``` and not of type ```Entity```), its own function definition will always be called regardless of the ```virtual``` keyword in its parent class.

However, the ```virtual``` keyword should be added in wherever applicable because function definitions such as the following can occur:

```C++
void PrintName(Entity* entity)
{
    std::cout << entity->GetName() << std::endl;
}
```

Even if the ```Player``` object is of type ```Player```, if that object is passed into the above ```PrintName()``` function it will be treated as an ```Entity``` type.

There are two runtime costs to using virtual functions: the memory required to store the v-table (a map that holds all virtual functions), and the processing time to access the v-table to determine which function to map to. In general, these costs are minimal and shouldn't typically dissuade you from using virtual functions.

---

## 28: Interfaces (Pure Virtual Functions)

A pure virtual function is the same as an abstract method in C# or Java. It allows us to define a function without an implementation in a base class while forcing sub-classes to create their own implementations.

```C++
class Entity
{
public:
    virtual std::string GetName() = 0; // pure virtual
}

class Player : public Entity
{
public:
    Player(const std::string& name) : m_Name(name) {}

    std::string GetName() override { return m_Name; }
}
```

Interfaces are a special type of class that are comprised entirely of pure virtual functions. The above ```Entity``` class is an interface. Interfaces are unable to be instantiated.

---

## 30: Arrays

An array is a collection of variables. Arrays store their data contiguously in memory.

To create an array on the stack:

```C++
int example[3];
for (int i = 0; i < 3; i++)
    example[i] = i * 2;
// or
int example2[3] = {0, 2, 4};

// arrays on the stack can use the C++ foreach syntax
for (int item : example)
{
    std::cout << item << std::endl;
}

LOG(example); // prints the address of example[0]
```

To create an array on the heap:

```C++
int* example = new int[3];
for (int i = 0; i < 3; i++)
    example[i] = i * 2;
// or
int* example2 = new int[3] {0, 2, 4};

// arrays on the heap can NOT use the C++ foreach syntax
for (int i = 0; i < 3; i++)
{
    std::cout << example[i] << std::endl;
}

delete[] example;
delete[] example2;
```

Note that arrays on the stack can use the C++ foreach syntax, but arrays on the heap cannot.

There are various tradeoffs for allocating an array on the stack vs the heap; allocating on the heap gives the advantage of data retention outside of the scope it is declared in. However, consider the following objects:

```C++
class Entity1
{
public:
    int example[5]; // allocate on the stack

    Entity1()
    {
        for (int i = 0; i < 5; i++)
            example[i] = 0;
    }
}

class Entity2
{
public:
    int* example = new int[5]; // allocate on the heap

    Entity2()
    {
        for (int i = 0; i < 5; i++)
            example[i] = 0;
    }
}
```

In the above example, ```Entity1``` and the first element of ```Entity1```'s ```example``` array will have the same memory address; the data will be as compact as possible.

However, ```Entity2```'s data is a bit more fragmented; ```Entity2```'s memory address will be shared with its int pointer. That pointer will have to be dereferenced to get to the underlying data. This process takes a bit more time than accessing the data for ```Entity1```, and risks more cache misses.

Arrays don't contain additional information besides the data itself. Therefore, the size of an array must be kept in a separate variable if needed. However, there is a method to backwards engineer an array's size:

```C++
int example[5];
int count = sizeof(example) / sizeof(int);
```

Assuming ```sizeof(int)``` equals 4 bytes, ```sizeof(example)``` would equal 20 (5 items x 4 bytes). Therefore, ```sizeof(example) / sizeof(int)``` would equal 5. *Note that this method only works for arrays allocated on the stack.*

In general, this method should be avoided. If the size of the array is needed, it should simply be tracked in a separate variable.

Arrays got an update in C++11 that prevents the need to do these sorts of calculations at the cost of a bit more memory. The ```Array``` class has a number of advantages including bounds checking and helpful functions like ```size()```, ```front()```, and ```back()```. The following is the syntax for the ```Array``` class:

```C++
#include <array>
// ...
std::array<int, 3> example;
for (int i = 0; i < example.size(); i++)
    example[i] = i * 2;
// or
std::array<int, 3> example2 = {0, 2, 4};
```

---

## 31: Strings

Strings represent text. They are functionally ```const char``` arrays, and as such they are immutable and contain a null terminator at the end of the array. Although the null terminator is included in the array's true size as an additional byte, ```strlen()``` and a string's ```size()``` function will return the size of the string not including the null terminator.

```C++
#include <string>
// ...
std::string example = "Hello, World!";
```

Note that ```<iostream>``` also contains a definition for string, but this is likely not the desired data type in most scenarios. The ```<iostream>``` string type does not contain an overload of the ```<<``` or ```>>``` operators.

The following will not work:

```C++
#include <string>
// ...
std::string example = "foo" + "bar"; // invalid
```

This does not work because it is the equivalent of trying to add together two separate ```char``` arrays before assigning the value to ```example```. To concatenate strings, do the following:

```C++
#include <string>
// ...
std::string example = "foo";
example += "bar";
// or
std::string example = std::string("foo") + "bar";
// or
std::string example = "foo"s + "bar";
```

Only once the first string is created can the ```+``` operator be successfully overloaded. the "s" in the last example is a special syntax that is equivalent to the second example.

Because it is a class, strings also come with helpful functions such as ```size()``` and ```find()```. The latter returns the index of the first occurrence in the string. Because strings do not have any sort of "contains" function, ```find()``` can be used instead:

```C++
#include <string>
// ...
std::string example = "foo";
bool contains = example.find("oo") != std::string::npos;
```

Note that string operations can get very expensive very quickly for large strings. In general, try to avoid copying strings (passing by value) and instead opt for const references:

```C++
void Print1(std::string string) // bad
{
    // ...
}

void Print2(const std::string& string) // good
{
    // ...
}
```

---

## 32: String Literals

The following char array is read only:

```C++
char* example = "foo";
example[0] = 'b'; // invalid
```

```example``` lives in readonly memory and this operation will compile but not work as intended (C++11 and onward actually now requires the ```const``` keyword to be prepended to ```char*```). Instead, write the following:

```C++
char example[] = "foo";
example[0] = 'b'; // valid
```

There are additional char pointer types:

```C++
const char* ex = u8"foo"; // always 1 byte per char (the "u8" is optional here)
const wchar_t* ex2 = L"foo"; // length is up to the compiler, usually 2 or 4 bytes per char

const char16_t* ex3 = u"foo"; // always 2 bytes per char
const char32_t* ex4 = U"foo"; // always 4 bytes per char
```

To write multi-line strings:

```C++
const char* example = R"(Line1
Line2
Line3)"; // these lines automatically include a new string
// or
const char* example2 = "Line1\n"
    "Line2\n"
    "Line3\n"; // these lines need the newline character written explicitly. Note the lack of "+" symbols
```

---

## 33: The ```const``` Keyword

```const``` is a promise that you will not modify a variable's data.

```C++
const int MAX_AGE = 100;
```

In *some* cases, this promise can be broken:

```C++
const int MAX_AGE = 100;
int* a = new int;
*a = 2;
a = (int*)&MAX_AGE; // const behavior is bypassed
```

In many cases this can cause your program to crash and should largely be avoided.

When dealing with pointers, the order that the ```const``` keyword appears matters:

```C++
const int* a = new int; // the pointer's dereferenced value cannot change (it must be a const int)
int const* b = new int; // equivalent to the above
int* const c = new int; // the address stored in foo cannot change
const int* const d = new int; // the same as both "a" and "c"
```

```const``` has a third usage than the two above: it can be placed after a function name, *but only in a class*:

```C++
class Entity
{
private:
    int m_X;
public:
    int GetX() const
    {
        return m_X;
    }
}
```

In the above example, the ```const``` keyword is saying that the ```GetX()``` function promises not to alter any of the ```Entity``` class's methods. That is, the following would be invalid:

```C++
class Entity
{
private:
    int m_X;
public:
    int GetX() const
    {
        m_X = 3; // invalid
        return m_X;
    }
}
```

Note that all three types of ```const``` can all technically be used at once:

```C++
class Entity
{
private:
    int* m_X;
public:
    const int* const GetX() const // just C++ things...
    {
        return m_X;
    }
}
```

```const``` has a purpose outside of simply enforcing that a class function doesn't change any class properties; it is necessary if functions external to that class have a ```const``` reference to that class:

```C++
void PrintEntity(const Entity& e)
{
    std::cout << e.GetX() << std::endl;
}
```

The above example would require the Entity class's ```GetX()``` method to use the ```const``` keyword after the function name. However, the following example would require that ```GetX()``` *does not* contain the ```const``` keyword after the function name:

```C++
void PrintEntity(Entity e)
{
    std::cout << e.GetX() << std::endl;
}
```

Because of this difference, it is not uncommon to see two different function definitions to allow for both behaviors:

```C++
class Entity
{
private:
    int m_X;
public:
    int GetX() const
    {
        return x;
    }

    int GetX()
    {
        return x;
    }
}
```

In general, all getters should be marked with ```const```. Not doing this prevents any functions from using a ```const``` reference to that class if using that getter is required.

Of course, it wouldn't be the C++ way if this behavior couldn't be bypassed. Sometimes for the purposes of debugging, or some strange edge cases, you simply need to be able to modify a variables value while keeping a function ```const```. This is where the ```mutable``` keyword comes in:

```C++
class Entity
{
private:
    int m_X;
    mutable int debugCount = 0;
public:
    int GetX() const
    {
        debugCount++;
        return x;
    }
}
```

---

## 34: The ```mutable``` Keyword

The ```mutable``` keyword has a few different uses: allowing variables to be modified in ```const``` functions (see the last example from the last chapter), or in lambda functions such as the following:

```C++
int x = 0;
auto f = [=]() mutable
{
    x++;
    std::cout << x << std::endl;
}

f();
// x still equals 0 here
```

Lambda functions haven't been covered yet, but just note that the equals sign contained within the brackets (```[=]```) notates that all passed in values are by value. If that equals sign was instead an ampersand (```[&]```) it would default all parameters to be pass by reference. Because ```x``` is passed in by value, x would not be able to be modified. The ```mutable``` keyword after the lambda function definition works around that.

That being said, using ```mutable``` in the lamda case is very rare compared to with class functions to the point where you may never even see it in practice.

---

## 48: Local ```static``` Variables

The lifetime of a local ```static``` variable *exists beyond the scope that that variable is defined in*.

```C++
void Foo()
{
    int i = 0;
    i++;
    std::cout << i << std::endl;
}

void Bar()
{
    static int i = 0;
    i++;
    std::cout << i << std::endl;
}

int main()
{
    Foo(); // prints "1"
    Foo(); // prints "1"

    Bar(); // prints "1"
    Bar(); // prints "2"
}
```

This behavior imitates that of global variables except that local ```static``` variables cannot be accessed outside of the scope that they are defined in.

This technique can be used to make code a bit more readable and reduce the amount of boilerplate code for something like initialization functions or singletons:

```C++
class Singleton
{
public:
    static Singleton& Get()
    {
        static Singleton instance; // only one instance will ever be created
        return instance;
    }

    void Foo()
    { 
        // ...
    }
}

int main()
{
    Singleton::Get().Foo();
}
```

---

## 49: How to Use Libraries (Static Linking)

Static linking takes a .lib library file and places that library in an application's .exe file. On the other hand, dynamic libraries get linked at runtime and are generated as .dll files or .lib files that point to .dll files. Because of linker optimizations, statically linking a library technically allows an application to run a bit faster than if that library were dynamically linked.

Sometimes a library can ship with both its static and dynamic versions, in which case there would exist two .lib files. To tell the difference, typically static library .lib files have larger file sizes.

To bring in external libraries in a C++ project, create a dependencies folder. Create a subfolder and paste in the .lib file. Right click the project > Properties > set "Configuration" to "All Configurations" > "C/C++" dropdown > General > set "Additional Include Directories" to "$(SolutionDir)\Dependencies\ExampleLib". Now, you can include the library in your project:

```C++
#include "examplePackage.h"
```

It doesn't technically matter whether brackets or quotes are used in the above example, but quotes will check relative paths first followed by the compiler include paths (where common libraries like iostream exist). A good rule of thumb is to use quotes if the package file is contained in your solution, otherwise (such as with system or external libraries) use brackets.

Now that the package is properly referenced, it now needs to be linked. Recall that linking does things like linking function declarations to their definitions. Without linking your code will not compile. To link in VS: Right click the project > Properties > set "Configuration" to "All Configurations" > "Linker" dropdown > General > set "Additional Library Directives" to "$(SolutionDir)\Dependencies\ExampleLib" > back in the "Linker" dropdown > Input > add the name of the .lib file to "Additional Dependencies" (you can add it to the front of the list).

---

## 55: Macros

Macros let you define snippets of code with the #define keyword. Macros are conventionally written in all caps.

```C++
#define WAIT std::cin.get()

int main() {
    WAIT;
}
```

Macros can also let you define function-like behavior

```C++
#include <iostream>
#define LOG(x) std::cout << x << std::endl

int main()
{
    LOG("hello world");
}
```

The difference between a macro and a function is that a macro is simply an alias; the pre-compiler replaces all instances of the macro with the macro's value.

---

## 56: The ```auto``` Keyword

The ```auto``` keyword allows for dynamically determining the type of a variable.

```C++
auto foo = 1 / 3;
auto var = "example";
```

Note: references still need to be made explicitly. The two below snippets are equivalent.

```C++
std::vector<int>& foo = bar;
```

```C++
auto& foo = bar;
```

There are instances (such as with complicated templates) where the type is always unknown and ```auto``` is required. Typically this sort of code should be avoided if possible.
