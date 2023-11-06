# JavaScript Tutorial

---
## Data Types

```js
// Numbers:
let length = 16;
let weight = 7.5;

// Strings:
let color = "Yellow";
let lastName = "Johnson";

// Booleans
let x = true;
let y = false;

// Object:
const person = {firstName:"John", lastName:"Doe"};

// Array object:
const cars = ["Saab", "Volvo", "BMW"];

// Date object:
const date = new Date("2022-03-25"); 
```

When a variable is declared without a value then it will be initialized with a value of `undefined`.

### Numbers

All js numbers use 64bit floating point precision.

`NaN` is a special type of number. If it is used in any operation, the return value of that operation will also be `NaN`. The Javascript `isNaN()` function returns whether or not a number's value is currently `NaN`.

```js
let x = 1 / "apple"; // NaN
typeof x; // returns "number"
isNaN(x); // returns "true"
```

`Infinity` is also a number:
```js
typeof infinity; // returns "number"
```

### Strings

javascript allows for `[]` property access on strings:
```js
let text = "Hello World";
console.log(text[0]);
```
However this is READ ONLY. Strings cannot be written to using `[]` property access. If individual character access is needed for writing purposes, first convert the string to an array of characters:
```js
let arr = text.split("");
```

Strings allow for template literal syntax by using back ticks:
```js
// write out reserved characters like quotes
let text1 =  `He's often called "Johnny"`;

// inline variables
let firstName = "John";
let lastName = "Doe";
let text2 = `Welcome ${firstName}, ${lastName}!`;

// inline expressions
let text3 = `${1 + 2 + 3}`;
```

### Objects

Object properties can be retrieved in two ways:
```js
person.lastName;
// or
person["lastName"];
```

### Arrays

Note that Arrays are objects:
```js
arr = [1, 2, 3];
typeof arr; // returns "object"
```

Javascript arrays can hold more than one type by default:
```js
arr = [Date.now, myFunction, myObjects, 99, "hello"];
```

Looping through arrays can be done in two ways:
```js
for (let i = 0; i < arr.length; i++) {
  // do something
}
// or
arr.forEach(myFunction);


function myFunction(value, index, array) {
  txt += value + "<br>";
}
```

Note that the function takes 3 arguments: the item value, the item index, and the array itself. Unused parameters can be omitted.

Adding and removing from an array:
```js
arr.push(myValue);
arr.pop(myValue);
```

Sorting and reversing an array:
```js
arr.sort();
arr.reverse();
```
Note that sorting converts all values to strings by default, so numbers won't be sorted in numerical order given certain values (e.g. "25" would come after "100"). Use the following to sort an array of numbers:
```js
arr.sort(function(a, b){return a - b});
```

Above you can see that `sort()` takes a compare function with parameters `a` and `b`. The compare function should return a negative, zero, or positive value. If the result is negative, `a` is sorted before `b`. If the result is positive, `b` is sorted before `a`.

The compare function is also useful to compare properties on an array of objects:
```js
const cars = [
  {type:"Volvo", year:2016},
  {type:"Saab", year:2001},
  {type:"BMW", year:2010}
];

cars.sort(function(a, b){return a.year - b.year});
```

The `map()` method creates a new array by performing a function on each array element:
```js
const numbers1 = [1, 2, 3]
const numbers2 = numbers1.map((value) => {
  return value * 2;
});
```
Note that just like the `ForEach()` method, the function parameter takes three optional arguments: the item value, the item index, and the array itself.

The `filter()` method creates a new array with array elements that meet a condition:
```js
const numbers1 = [1, 2, 3]
const numbers2 = numbers1.filter((value) => {
  return value < 3;
});
```
Again, the function parameter takes three optional arguments: the item value, the item index, and the array itself.

---
## Classes

Class syntax:
```js
class Car {
  constructor(name, year) { // note the constructor is named "constructor()"
    this.name = name; // note that "this" here declares variables automatically
    this.year = year;
  }

  method_1() { ... }

  method_2() { ... }
}

let myCar = new("Toyota", 2005);
```

---
## Modules

JavaScript modules allow you to break up your code into separate files. Modules are made viewable to other files with the `export` statement, and imported from external files with the `import` statement.

### Exports

Named Export:
```js
export const name = "John";
export const age = 99;
```
or
```js
const name = "John";
const age = 99;
export {name, age};
```

Default Export (only one allowed per file):
```js
const info = () => {
    const name = "John";
    const age = 99;
    return name + ' is ' + age + ' years old.';
}

export default info; // note the "default" keyword
```

### Imports

Import from named export:
```js
import { name, age } from "./example.js";
```

Import from default export:
```js
import info from "./example.js";
```

Import from `<script>` tag (note the `type` attribute):
```js
<script type="module">import message from "./example.js";</script>
```

---
## JSON

JSON to JS object:
```js
const myObj = JSON.parse(myJsonTextString);
```

JS object to JSON:
```js
const json = JSON.stringify(myObj);
```

---
## Event Listeners

Event listeners allow you to trigger a function when a specific event occurs on an element. The below event listener listens for a click event:

```js
document.getElementById('my-element').addEventListener('click', (event) => {
    // do something
});
```
Note that the `event` parameter above can be omitted if not used.

Remove event listener:
```js
element.removeEventListener("click", myFunction);
```
