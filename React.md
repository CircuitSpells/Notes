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

## Render and Sorting Lists

App.js

```js
import { useEffect, useState } from "react";
import Articles from "./components/Articles";
import "./App.css";

function App({ articles }) {
  const [sortedArticles, setSortedArticles] = useState(articles);

  const handleMostUpvoted = () => {
    setSortedArticles((prev) =>
      [...prev].sort((a, b) => b.upvotes - a.upvotes)
    );
  };

  const handleMostRecent = () => {
    setSortedArticles((prev) =>
      [...prev].sort((a, b) => new Date(b.date) - new Date(a.date))
    );
  };

  useEffect(() => {
    handleMostUpvoted();
  }, [articles]);

  return (
    <>
      <div className="App">
        <div>
          <label>Sort By:</label>
          <button
            data-testid="most-upvoted-link"
            className="small"
            onClick={handleMostUpvoted}
          >
            Most Upvoted
          </button>
          <button
            data-testid="most-recent-link"
            className="small"
            onClick={handleMostRecent}
          >
            Most Recent
          </button>
        </div>
        <Articles articles={sortedArticles} />
      </div>
    </>
  );
}

export default App;
```

Articles.js

```js
function Articles({ articles = [] }) {
  const articleItems = articles.map((article) => (
    <tr data-testid="article" key={`article-index-${article.id}`}>
      <td data-testid="article-title">{article.title}</td>
      <td data-testid="article-upvotes">{article.upvotes}</td>
      <td data-testid="article-date">{article.date}</td>
    </tr>
  ));

  return (
    <>
      <table>
        <thead>
          <tr>
            <th>Title</th>
            <th>Upvotes</th>
            <th>Date</th>
          </tr>
        </thead>
        <tbody>{articleItems}</tbody>
      </table>
    </>
  );
}

export default Articles;
```

Note: React requires the `key` attribute on the outer html element, and each key needs to be unique.

## Tips

add a css file:

```js
import "./styles.css";
```

html attribute with string interpolation:

```js
export default function Example({ id }) {
  return <div id={`some-${id}`}></div>;
}
```
