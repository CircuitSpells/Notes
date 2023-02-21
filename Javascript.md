# JavaScript Tutorial

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

Event listeners allow you to trigger a method when a specific event occurs on an element. The below event listener listens for a click event:

```js
document.getElementById('my-element').addEventListener('click', () => {
    // do something
});
```
