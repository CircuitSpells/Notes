# Golang

## Setup

playground: https://go.dev/play

- install Go
- install VS Code
- install Go extension
- install Go libraries:
  - in VS Code, Ctrl+Shift+P to go to command palette.
  - search "Go: Install/Update Tools".
  - click the top checkbox in order to install all tools and click "OK".

## CLI

simply typing "go" will show quick commands. Add "help" to expand, e.g. `go help mod`, or `go help mod init`.

## Getting Started

init project:

```
go mod init demo
```

this will create a go.mod file which describes the metadata of the project.

create a `main.go` file with the following contents:

```go
package main

import "fmt"

func main() {
  fmt.Println("Hello, World!")
}
```

run the project (in current directory):

```
go run .
```

## Variables and Data Types

four simple data types:

- Strings
- Numbers
- Booleans
- Errors

variable declaration:

```go
var myName string // initialized to its "zero value", in this case the empty string
var myName string = "Alice" // verbose
var myName = "Bob" // inferred type
myName := "Carol" // short declaration syntax
```

constants:

```go
const a = 42 // type determined at runtime
const b int = 101 // explicitly typed
const c = a // const assignment
const ( // group of constants
  d = true
  e = 3.14
)
const f = 2 * 5 // constant expression (determined at compile-time; no dynamic behavior e.g. function calls)
```

## Aggregate Types

### Arrays

an array is a fixed-size collection of the same type.

```go
var arr [3]int // initialized to its "zero value"
fmt.Println(arr) // [0, 0, 0]

arr = [3]int{1, 2, 3} // array literal (pre-defined values)
fmt.Println(arr[1]) // 2

fmt.Println(len(arr)) // 3

// Arrays copy by value:

arr := [3]string{"foo", "bar", "baz"}
arr2 := arr
fmt.Println(arr2) // {"foo", "bar", "baz"}

arr[0] = "quux"
fmt.Println(arr) // {"quux", "bar", "baz"}
fmt.Println(arr2) // {"foo", "bar", "baz"}

arr == arr2 // false (arrays are comparable)
```

### Slices

slices are subsets of array. They are reference types, so if the array changes, so does the slice.

```go
var s []int // slice. Note that no size is defined
fmt.Println(s) // [] (syntactic sugar for nil)

s = []int{1, 2, 3}
fmt.Println(s[1]) // 2

s[1] = 99
fmt.Println(s) // [1 99 3]

s = append(s, 5, 10, 15) // add elements to the slice
```

slices are reference types:

```go
s := []int{1, 2, 3}
s2 := s // copied by reference. Use slices.Clone to clone

s[0] = 99
fmt.Println(s, s2) // [99 2 3] [99 2 3]

s == s2 // compile time error (slices are not comparable)
```

### Maps

maps are hashtables (key/val pairs).

```go
var m map[string]int
fmt.Println(m) // map[] (nil)
m = map[string]int{
  "foo": 1,
  "bar": 2, // note the comma on the last item (arrays and slices require this as well)
}

fmt.Println(m["foo"]) // 1
m["bar"] = 99

delete(m, "foo") // remove entry
m["baz"] = 42 // add value to map

// key that doesn't exist:
fmt.Println(m["foo"]) // 0 (queries always return results, in this case the zero value)

// alternatively, use ok syntax:
v, ok := m["foo"] // ok is true if "foo" exists in the map, false otherwise
fmt.Println(v, ok) // 0, false
```

### Structs

a struct is a fixed-size collection (like arrays) of varying types.

```go
var s struct{ // declare an anonymous struct
  name string
  id int
}

// structs are value types:
fmt.Println(s) // {"" 0}

s.name = "Alice"

// it is common to use types to represent the struct:
type myStruct struct{
  name string
  id int
}

var s myStruct // declare variable with custom type
s = myStruct{
  name: "Bob",
  id: 42,
}
s2 := s

// because structs are value types, they are comparable:
s == s2 // true (go checks for the same fields _and_ the same field order)
```

slice of structs:

```go
type score struct {
  name string
  score int
}

scores := []score{
  {name: "Alice", score: 87},
  {name: "Bob", score: 96},
  {name: "Carol", score: 64},
}
```

## Branches

if statement:

```go
if test { ... }
else if { ... }
else { ... }

if initializer; test { ... }
```

switch statement:

```go
i := 999
switch i {
  case 1:
    // ...
  case 2, 3:
    // ...
  default:
    // ...
}

// or use initializer syntax:
switch i := 999; i {
  case 1:
  // ...
}
```

## Loops

all loops are for loops in Go:

```go
for { ... } // infinite loop

for condition { ... } // loop until
// e.g.:
i := 1
for i < 3 {
  i += 1
}

for initializer; test; post clause { ... } // counter-based loop
// e.g.:
for i := 1; i < 3; i++ {
  // ...
}
```

looping over collections:

```go
for key, value := range collection { ... } // able to loop over arrays, slices, and maps
// e.g.:
arr := [3]int{1, 2, 3}
for i, val := range arr {
  fmt.Println(i, val) // print index and value
}

// if you don't need the values:
for key := range collection { ... }

// if you don't need the keys:
for _, value := range collection { ... }
```

