## I checked this and that of React basics with `console.log()`

Click here to read the article in Japanese：https://zenn.dev/takuyakikuchi/articles/2c4071a58bd4d7

# `console.log()` to check rendering timings

⚠️ To simplify the logging results, "Strict Mode" is intentionally disabled so that the lifecycle is never called twice.<br />
[Strict Mode – React](https://reactjs.org/docs/strict-mode.html#gatsby-focus-wrapper)

## 1. State update in the parent-component and the child-component, and re-rendering

### Things to confirm
- Check for re-rendering when the state in the parent-component and child-component is changed.

### Code
- Parent component: `App`
- Child component：
  - `ChildA`(receives props from the parent)
    - It has `count` state.
  - `ChildB` (does not receive props from the parent)

```jsx
const ChildA = ({ state }) => {
  const [count, setCount] = React.useState(0);
+ console.log(`rendering in child A component: count has ${count}`);
  return (
    ...
      <button onClick={() => setCount(count + 1)}>Child A: Count-up</button>
    ...
  );
};
const ChildB = () => {
  console.log("rendering in child B component");
  return <div>Child B doesn't have props passed from the parent</div>;
};
export default function App() {
  const [state, setState] = React.useState(false);
  console.log("rendering in parent component");
  return (
    <div className="App">
      ...
      <button onClick={() => setState(!state)}>Update the parent state</button>
      ...
      <ChildA state={state} />
      ...
      <ChildB />
    </div>
  );
}
```

### Console results
```
<!-- 1. Initial rendering -->
rendering in parent component 
rendering in child A component: count has 0 
rendering in child B component 
<!-- 2. Update the parent state -->
rendering in parent component 
rendering in child A component: count has 0 
rendering in child B component 
<!-- 3. Update the child A state -->
rendering in child A component: count has 1 
<!-- 4. Update the parent state -->
rendering in parent component 
rendering in child A component: count has 1 
rendering in child B component 
```

### Confirmed
- When the state of the parent component is changed, re-rendering occurs in both parent and child components, regardless of whether props are passed or not. (See No.2)
- When the state is changed in a child component, re-rendering occurs only in that component. (See No. 3)
- When a parent component is re-rendered and a child component is re-rendered, the state of the child component is kept up-to-date. (See No. 4)

### Demo
%[https://codesandbox.io/embed/rendering-usestate-default-fwk9xt?fontsize=14&hidenavigation=1&theme=dark]

## 2. useState initialState vs. Lazy initial state

### Things to confirm
- Confirm that the lazy initial state is called only at initial rendering.
- On the other hand, confirm that the `initialState` is called at every re-rendering.

[React: useState](https://reactjs.org/docs/hooks-reference.html#usestate)

### Code
- Parent component: `App`
- Child component: `Child`.
  - `childStateA` state: lazy initial state
  - `childStateB` state: initialState

```jsx
const someExpensiveCalculation = (number, type) => {
  console.log(`in the ${type} initial state`);
  return number * 10;
};
const Child = ({ number }) => {
  const [childStateA, setChildStateA] = React.useState(() => {
    return someExpensiveCalculation(number, "lazy");
  });
  const [childStateB, setChildStateB] = React.useState(
    someExpensiveCalculation(number, "default")
  );
  console.log(
    `rendering in child component: A: ${childStateA}, B: ${childStateB}`
  );
  return (
    <>
      <p>{`The childStateA is ${childStateA}`}</p>
      <button onClick={() => setChildStateA(childStateA + 1)}>
        Child A: Count-up
      </button>
      <p>{`The childStateB is ${childStateB}`}</p>
      <button onClick={() => setChildStateB(childStateB + 1)}>
        Child B: Count-up
      </button>
    </>
  );
};
export default function App() {
  const [state, setState] = React.useState(false);
  return (
    <div className="App">
      <button onClick={() => setState(!state)}>Update the parent state</button>
      <Child number={10} />
    </div>
  );
}
```

### Console result

```
<!-- 1. Initial rendering -->
in the lazy initial state 
in the default initial state 
rendering in child component: A: 100, B: 100 
<!-- 2. Parent state update -->
in the default initial state 
rendering in child component: A: 100, B: 100 
<!-- 3. Child state A update -->
in the default initial state 
rendering in child component: A: 101, B: 100 
<!-- 3. Child state B update -->
in the default initial state 
rendering in child component: A: 101, B: 101 
<!-- 4. Parent state update -->
in the default initial state 
rendering in child component: A: 101, B: 101 
```

### Confirmed
- With lazy initial state, **someExpensiveCalculation()` is called only at initial rendering**, and is ignored on re-rendering.
- On the other hand, when a value passed simply as `initialState`, **someExpensiveCalculation()` is called every time the re-rendering runs**.

### Demo
%[https://codesandbox.io/embed/heuristic-goldwasser-gwvki8?fontsize=14&hidenavigation=1&theme=dark]

## 3. Timing of `useEffect`

### Things to confirm
- Make sure that the function passed to `useEffect` runs after the render result is reflected on the screen.

[React: useEffect](https://reactjs.org/docs/hooks-reference.html#useeffect)

### Code
- In `useEffect` where `state` is a dependent value, update the `message` state after fetching data.

```jsx
const dataFetchMock = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("setMessage executed in useEffect");
  }, 1500);
});
export default function App() {
  const [message, setMessage] = React.useState();
  const [state, setState] = React.useState(false);
  React.useEffect(() => {
    console.log(`in useEffect. state: ${state}`);
    dataFetchMock.then((value) => {
      setMessage(value);
    });
  }, [state]);

  console.log(`rendering: just before return jsx. message: ${message}`);
  return (
    <div className="App">
      <button onClick={() => setState(!state)}>Update the parent state</button>
      <p>{message === undefined ? "undefined" : message}</p>
    </div>
  );
}
```

### Console result
```
<!-- 1. Initial rendering -->
rendering: just before return jsx. message: undefined 
in useEffect. state: false 
rendering: just before return jsx. message: setMessage executed in useEffect 
<!-- 2. State(dependency of the useEffect) updated -->
rendering: just before return jsx. message: setMessage executed in useEffect 
in useEffect. state: true 
rendering: just before return jsx. message: setMessage executed in useEffect 
```

### Confirmed
- **useEffect is working after render. **
  - Initial rendering (see No.1), render first => `useEffect` => change of `message` state in `useEffect` triggered rendering again
  - In updating the state contained in the dependency array of `useEffect` (see No.2), rendering by updating the state => `useEffect` => re-rendering by changing the `message` state in `useEffect`.

### Demo
%[https://codesandbox.io/embed/useeffect-timing-4qjgsk?fontsize=14&hidenavigation=1&theme=dark]

# Summary

React can be used with a vague understanding.
However, I thought it would be useful to check the timing of re-rendering and so on by myself.
