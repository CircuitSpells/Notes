# React

## Basic Component

```js
import { useState } from "react";

export default function Form({ inputOne, inputTwo }) {
  const [person, setPerson] = useState({
    firstName: "John",
    lastName: "Doe",
    email: "JohnDoe@example.com",
  });

  function handleFirstNameChange(e) {
    setPerson({
      ...person,
      firstName: e.target.value,
    });
  }

  function handleLastNameChange(e) {
    setPerson({
      ...person,
      lastName: e.target.value,
    });
  }

  function handleEmailChange(e) {
    setPerson({
      ...person,
      email: e.target.value,
    });
  }

  return (
    <>
      {/* display input parameters */}
      <p>{inputOne}</p>
      <p>{inputTwo}</p>

      {/* state change */}
      <label>
        First name:
        <input value={person.firstName} onChange={handleFirstNameChange} />
      </label>
      <label>
        Last name:
        <input value={person.lastName} onChange={handleLastNameChange} />
      </label>
      <label>
        Email:
        <input value={person.email} onChange={handleEmailChange} />
      </label>

      {/* display state */}
      <p>
        {person.firstName} {person.lastName} ({person.email})
      </p>
    </>
  );
}
```

## Tips

html attribute with string interpolation:

```js
export default function Example({ id }) {
  return <div id={`some-${id}`}></div>;
}
```
