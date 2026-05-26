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
m = map[string]int{"foo": 1, "bar": 2}

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

