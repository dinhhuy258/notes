# React Hooks

## useState

- Stores stateful values in a component.
- Updating state triggers a re-render.
- Good for values shown in UI.

```js
const [count, setCount] = useState(0);
setCount(count + 1);
```

## useEffect

- Runs side effects after render.
- Useful for fetching data, subscriptions, timers, DOM updates.
- Dependency array controls when it runs:
  - [] → run once on mount
  - [value] → run when value changes
  - no array → run on every render

```js
useEffect(() => {
  console.log("Run when count changes");
}, [count]);
```

## useContext

- Shares values between components without passing props.
- Requires:
  - Creating a Context
  - Wrapping components with a Provider
  - Using useContext() inside child components

```js
import { createContext, useContext, useState } from "react";

// 1. Create Context
const ThemeContext = createContext();

// 2. Provider component
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");

  const value = {
    theme,
    toggleTheme: () => setTheme((t) => (t === "light" ? "dark" : "light")),
  };

  return (
    <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>
  );
}

// 3. Child component using the context
function ThemeButton() {
  const { theme, toggleTheme } = useContext(ThemeContext);

  return <button onClick={toggleTheme}>Current theme: {theme}</button>;
}

// 4. App root
function App() {
  return (
    <ThemeProvider>
      <ThemeButton />
    </ThemeProvider>
  );
}

export default App;
```

## useRef

- Stores a mutable value that persists across re-renders.
- Changing ref.current does NOT trigger a re-render.

| Feature                             | `let` variable                          | `useRef`                                                   | `useState`                          |
| ----------------------------------- | --------------------------------------- | ---------------------------------------------------------- | ----------------------------------- |
| **Persists across re-renders**      | ❌ No                                   | ✅ Yes                                                     | ✅ Yes                              |
| **Triggers re-render when updated** | ❌ No                                   | ❌ No                                                      | ✅ Yes                              |
| **Stored value type**               | Normal JS variable                      | `.current` property                                        | State value                         |
| **When value updates happen**       | Every render (recreated)                | Anytime (mutable)                                          | Through `setState`                  |
| **React tracks changes?**           | ❌ No                                   | ❌ No                                                      | ✅ Yes                              |
| **Good for…**                       | Temporary values inside a single render | DOM refs, timers, previous values, persistent mutable data | UI state, data shown on screen      |
| **Lifespan**                        | Only within one render                  | Component lifetime                                         | Component lifetime                  |
| **Common problems**                 | Loses value on re-render                | Won’t update UI                                            | Can cause re-renders, async updates |

**Example — DOM reference**

```js
const inputRef = useRef();

useEffect(() => {
  inputRef.current.focus();
}, []);

return <input ref={inputRef} />;
```

**Example — storing mutable value**

```js
const countRef = useRef(0);

function handleClick() {
  countRef.current += 1; // persists across renders
  console.log(countRef.current);
}
```

## useMemo

- Memoizes expensive computations.
- Runs only when dependencies change.
- Prevents unnecessary recalculations.

```js
const total = useMemo(() => heavyCalculation(data), [data]);
```
