# 20. React, Part II

## Function Components and *React* Hooks

*   State hook
*   Effect hook

```js
//index.js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import reportWebVitals from './reportWebVitals';

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('app')
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

```js
//App.js
import React, { useState } from 'react';
import { AddThoughtForm } from './AddThoughtForm';
import { Thought } from './Thought';
import { generateId, getNewExpirationTime } from './utilities';
import './styles.css';

export default function App() {
  const [thoughts, setThoughts] = useState([
    {
      id: generateId(),
      text: 'This is a place for your passing thoughts.',
      expiresAt: getNewExpirationTime(),
    },
    {
      id: generateId(),
      text: "They'll be removed after 15 seconds.",
      expiresAt: getNewExpirationTime(),
    },
  ]);

  const addThought = (thought) => {
    setThoughts((prev) => [thought, ...prev]);
  };

  const removeThought = (thoughtIdToRemove) => {
    setThoughts((thoughts) =>
      thoughts.filter((thought) =>
      thought.id !== thoughtIdToRemove)
    );
  };

  return (
    <div className="App">
      <header>
        <h1>Passing Thoughts</h1>
      </header>
      <main>
        <AddThoughtForm addThought={addThought} />
        <ul className="thoughts">
          {thoughts.map((thought) => (
            <Thought key={thought.id} thought={thought} removeThought={removeThought} />
          ))}
        </ul>
      </main>
    </div>
  );
}
```

```js
//Thought.js
import React, { useEffect } from 'react';
import './styles.css';

export function Thought(props) {
  const { thought, removeThought } = props;

  useEffect(() => {
    const timeRemaining = thought.expiresAt - Date.now();
    const timeout = setTimeout(() => {
      removeThought(thought.id);
    }, timeRemaining);
    return () => {
      clearTimeout(timeout);
    };
  }, [thought]);

  const handleRemoveClick = () => {
    removeThought(thought.id);
  };

  return (
    <li className="Thought">
      <button
        aria-label="Remove thought"
        className="remove-button"
        onClick={handleRemoveClick}
      >
        &times;
      </button>
      <div className="text">{thought.text}</div>
    </li>
  );
}
```

```js
//AddThoughtForm.js
import React, { useState } from 'react';
import { generateId, getNewExpirationTime } from './utilities';
import './styles.css';

export function AddThoughtForm(props) {
  const [text, setText] = useState('');

  const handleTextChange = ({target}) => {
    setText(target.value);
  }

  const handleSubmit = (event) => {
    event.preventDefault();
    if (text.length > 0) {
      const newThought = {
        id: generateId(),
        text: text,
        expiresAt: getNewExpirationTime()
      };
      props.addThought(newThought);
      setText('');
    }
  }

  return (
    <form className="AddThoughtForm"
          onSubmit={handleSubmit}>
      <input
        type="text"
        aria-label="What's on your mind?"
        placeholder="What's on your mind?"
        value={text}
        onChange={handleTextChange}
      />
      <input type="submit" value="Add" />
    </form>
  );
}
```

```js
//utilities.js
export function getNewExpirationTime() {
    return Date.now() + 15 * 1000;
  }
  
  let nextId = 0;
  export function generateId() {
    const result = nextId;
    nextId += 1;
    return result;
  }
```

## Functional Components

*   In the latest versions of React, function components can do everything that class components can do.
    *   In most cases, however, function components offer a **more elegant, concise way of creating React components.**
    *   *Function components* are React components defined as JavaScript functions
    *   Function components must return JSX
    *   Function components may accept a `props` parameter. Expect it to be a JavaScript object

## React Hooks

*   React Hooks, plainly put, are functions that let us manage the internal state of components and handle post-rendering side effects directly from our function components. Hooks don’t work inside classes — they let us use fancy React features without classes. Keep in mind that function components and React Hooks do not replace class components. They are completely optional; just a new tool that we can take advantage of.
    *   *If you’re familiar with* [lifecycle methods](https://reactjs.org/docs/state-and-lifecycle.html#adding-lifecycle-methods-to-a-class) *of class components, you could say that Hooks let us “hook into” state and lifecycle features directly from our function components.*
    *   React offers a number of built-in Hooks. A few of these include `useState()`, `useEffect()`, `useContext()`, `useReducer()`, and `useRef()`
*   There are two main rules to keep in mind when using Hooks:
    *   only call Hooks at the top level
    *   only call Hooks from React functions

### useState()

*   `useState()` is a JavaScript function defined in the React library. When we call this function, it returns an array with two values:

    *   *current state* - the current value of this state

    *   *state setter* - a function that we can use to update the value of this state

    *   ```js
        const handleChange = ({target}) => setEmail(target.value);
        ```

    *   **Set From Previous State**

        *   ```js
             setCount(prevCount => prevCount + 1)
            ```

    *   While there are times when it can be helpful to store related data in a data collection like an array or object, **it can also be helpful to separate data that changes separately into completely different state variables**. **Managing dynamic data is much easier when we keep our data models as simple as possible.**

### useEffect()

*   use the Effect Hook to run some JavaScript code after each render, such as: 

    *   fetching data from a backend service

    *   subscribing to a stream of data

    *   managing timers and intervals

    *   reading from and making changes to the DOM

*   There are three key moments when the Effect Hook can be utilized:

    1.  When the component is first added, or *mounted*, to the DOM and renders
    2.  When the state or props change, causing the component to re-render
    3.  When the component is removed, or *unmounted*, from the DOM.

*   ```react
    import React, { useState, useEffect } from 'react';
    
    ...
    
    useEffect(() => {
        document.title = `Hi, ${name}`;
    }, [<dependencies>]);
    ```

    *   The first argument passed to the `useEffect()` function is the callback function that we want React to call after each time this component renders. We will refer to this callback function as our *effect*.
    *   We pass an empty array to `useEffect()` as the second argument. This second argument is called the *dependency array*.
        *   The default behavior of the Effect Hook is to call the effect function after every single render.
        *   **When our effect is responsible for fetching data from a server**, we pay extra close attention to when our effect is called. **Unnecessary round trips back and forth between our React components and the server can be costly** in terms of:
            *   Processing
            *   Performance
            *   Data usage for mobile users
            *   API service fees

*   Some effects require cleanup. For example, we might want to add event listeners to some element in the DOM, beyond the JSX in our component. When we [add event listeners to the DOM](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener), it is important to remove those event listeners when we are done with them to avoid [memory leaks](https://auth0.com/blog/four-types-of-leaks-in-your-javascript-code-and-how-to-get-rid-of-them/)!

    *   ```react
        useEffect(()=>{
          document.addEventListener('keydown', handleKeyPress);
          return () => {
            document.removeEventListener('keydown', handleKeyPress);
          };
        })
        ```

        *   If our effect didn’t return a *cleanup function*, then a new event listener would be added to the DOM’s `document` object every time that our component re-renders.
        *   If our effect returns a function, then the `useEffect()` Hook always treats that as a cleanup function.
            *   React will call this cleanup function before the component re-renders or unmounts.