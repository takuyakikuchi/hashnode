## Problems with data fetching Effect, and Cleanup

Click here to read the article in Japanese：https://zenn.dev/takuyakikuchi/articles/a96b8d97a0450c

I was reading the Official React Docs [You Might Not Need an Effect](https://beta.reactjs.org/learn/you-might-not-need-an-effect), which presents examples where `useEffect()` is not required. <br/>
I wrote this article because I needed to wrap my head around the "fetching data" part where I learned a lot.


## Problematic code
(The example code used in this article is taken directly from [You Might Not Need an Effect](https://beta.reactjs.org/learn/you-might-not-need-an-effect))

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    fetchResults(query, page).then(json => {
      setResults(json);
    });
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
````

This example may cause a problem called a "race condition".<br/>
[Race condition - Wikipedia](https://en.wikipedia.org/wiki/Race_condition)

To take the example from the article, consider typing "hello" quickly.<br/>
The query changes from "h" to "he", "hel", "hell", and "hello", and this change in input initiates separate data fetches.<br/>
Since "hello" is typed last, we would expect the "hello" result to be the last one returned, but the problem is that this may not be the case.<br/>
It is possible that the "hell" response comes after the "hello" response, and if that is the case, the "hell" result will be displayed as the `setResults()` is executed last.

The visualization looks like this.<br/>
The order of the results is switched during data fetch, and the "hell" results will be the final `results`.

![スクリーンショット 2022-07-09 21.31.48.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1657372037361/76_AGR-U2.png align="left")

## Solution using cleanup code

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1); 
  useEffect(() => {
    let ignore = false;
    fetchResults(query, page).then(json => {
      if (!ignore) {
        setResults(json);
      }
    });
    // ====== 💫 here's the point =====
    return () => {
      ignore = true;
    }
    // ============================
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

Now, if we look at the code for the solution, we see that cleanup has been added.<br/>
The cleanup uses a variable called `ignore` to control the execution of `setResults()`.<br/>
Here, I needed to wrap my head around it.

First, let's see when the `useEffect()` cleanup is executed.<br/>
In the React official doc, [Using the Effect Hook – React](https://reactjs.org/docs/hooks-effect.html#effects-with-cleanup)
1. React performs cleanup when a component is unmounted.
2. React also cleans up side-effects from the previous render before executing the next side-effect.

The timing of 2 is important in this case.<br/>
The `useEffect()` is executed in the order of "he", "hel", "hell", and "hello", and the previous `useEffect()` is cleaned up at the timing before the next `useEffect()` is executed.<br/>
In this example, `ignore` is set to `true` in the cleanup, so `setResults()` will not be executed for the `useEffect()` that is executed before the cleanup has completed data fetch.<br/>
"hello", which is the last `useEffect()` to be executed, has no next `useEffect()`, so cleanup is not executed, resulting in `setResults()` being the last to be executed.

This is what I think the visualization would look like.

![スクリーンショット 2022-07-09 21.32.15.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1657372355309/SVbb0M0T-.png align="left")

This is the Effect of fetching data using cleanup.

## Finally

In this article, we learned about cleanup in `useEffect()` and why it is important to implement cleanup in `useEffect()` data fetch.

It is considered good practice to extract the side effects of data fetch to a custom hook. 
The original article, which introduces that and many other situations where `useEffect()` should not be used, is very interesting and I encourage you to read it.
[You Might Not Need an Effect](https://beta.reactjs.org/learn/you-might-not-need-an-effect) Translated with www.DeepL.com/Translator (free version)