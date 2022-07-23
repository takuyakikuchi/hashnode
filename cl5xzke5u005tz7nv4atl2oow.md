## A high-level overview of Concurrent React

Click here to read the article in Japanese：
https://zenn.dev/takuyakikuchi/articles/91ccf7037d6375

## About this Article
> Concurrent React is more important than a typical implementation detail — it’s a foundational update to React’s core rendering model. So while it’s not super important to know how concurrency works, it may be worth knowing what it is at a high level.<br/>
quoted from [React v18.0 – React Blog](https://reactjs.org/blog/2022/03/29/react-v18.html#suspense-in-data-frameworks)

Here's an article to get a high-level overview of Concurrent React, an essential change to React's core rendering model.

## Concurrent React is not a feature, but a new mechanism

> Concurrency is not a feature, per se. It’s a new behind-the-scenes mechanism that enables React to prepare multiple versions of your UI at the same time. <br/>
quoted from [React v18.0 – React Blog](https://reactjs.org/blog/2022/03/29/react-v18.html#suspense-in-data-frameworks)

First of all, **not a feature**, as quoted above, Concurrent React is a new mechanism (React's new core rendering model) to allow React to have multiple versions of the UI ready at the same time.

What is **new** is the following
- Before: rendering is collective, uninterrupted, and synchronous
- Concurrent: **rendering is interruptible and asynchronous** (a key characteristic of Concurrent React)

With this new mechanism,<br/>
Users get a **smoother user experience**, <br/>
And, developers will be able to **more declaratively describe the control of the UI based on the loading state of components.**

To understand more concretely what this means, let's talk about one of Concurrent React's features, **Suspense**.

## Suspense

Here is a conceptual explanation of Suspense with a simple code example.<br/>
(Code taken from [React Top-Level API – React](https://reactjs.org/docs/react-api.html#reactsuspense))
```jsx
<Suspense fallback={<Spinner />}>
  <Comments />
</Suspense>
```

**Suspense handles the situation where an internal component is loading but not yet ready to render.**

In the previous section, I said that **rendering is interruptible and asynchronous**, which is an important characteristic of Concurrent React.<br/>
In this example,<br/> 
1. rendering `<Comments>` is suspended while it is loading(interruptible)
2. `<Spinner>` is displayed as a fallback UI while `<Comments>` is loading
3. when `<Commnets>` completed loading, it is rendered(**Asynchronous rendering**)

This way, the UI is displayed consistently thus UX is smooth.

Here is another example.
```jsx
function ProfilePage() {
  return (
    <PageLayout>
      <Suspense fallback={<MyProfileSkeleton />}>
        <MyProfile />
      </Suspense>
      <Suspense fallback={<AchievementsSkeleton />}>
        <Achievements />
      </Suspense>
      <Suspense fallback={<OrganizationSkeleton />}>
        <Organizations />
      </Suspense>
      <Suspense fallback={<ContributionsSkeleton />}>
        <Contributions />
      </Suspense>
    </PageLayout>
  );
};
```

In this example, each is enclosed in `<Suspense>`. <br/>
This allows them to have independent skeleton views and to be displayed asynchronously from the point where loading is complete.
Thus, **by changing the way Suspense is placed, asynchronous UI rendering can be finely controlled**.

## Suspense Use Cases

Now, let's look at a concrete use case of Suspense.

### Lazy loading of components using `React.lazy`

**The only use case** at this time. (See: [React Top-Level API – React](https://reactjs.org/docs/react-api.html#reactsuspense))

```jsx
// This component is loaded dynamically
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    // Displays <Spinner> until OtherComponent loads
    <React.Suspense fallback={<Spinner />}>
      <div>
        <OtherComponent />
      </div>
    </React.Suspense>
  );
}
```

This is supported since React v.16.6, so you may already be familiar with this use case.

The above example displays content `<Spinner>` for fallback while waiting for the loading of a lazy component `<OtherComponent>` using `React.lazy`.

`React.lazy` is a function used to perform code splitting, and rendering the lazy component within a `Suspense` component ensures that the UI is displayed consistently.
[Code-Splitting – React](https://reactjs.org/docs/code-splitting.html#reactlazy)

### Suspense with data fetching

The only officially supported use case at this time is using `React.lazy`, but the vision the React team has for Suspense seems to be much larger.

> As in previous versions of React, you can also use Suspense for code splitting on the client with React.lazy. But our vision for Suspense has always been about much more than loading code — the goal is to extend support for Suspense so that eventually, the same declarative Suspense fallback can handle any asynchronous operation (loading code, data, images, etc).<br/>
Quoted from [React v18.0 - React Blog](https://reactjs.org/blog/2022/03/29/react-v18.html#suspense-in-data-frameworks)

One of these is the use of Suspense in data fetching.

**Conventional way**<br/>

[**※Alert**: The following is a code example using the Apollo Client notation, but the Apollo Client does not support Suspense at this time.<br/>
Github issue about Suspense support: https://github.com/apollographql/apollo-client/issues/9627]
```jsx
// Dog.jsx
function Dogs() {
  const { loading, error, data } = useQuery(GET_DOGS);

  if (loading) return 'Loading...';
  if (error) return `Error! ${error.message}`;

  return (
    <ul>
      {data.dogs.map((dog) => (
        <li key={dog.id}>{dog.breed}</li>
      ))}
    </ul>
  );
}
```

In components that perform asynchronous loading, the "process during loading" and the "process when loading is completed" are combined.

**Suspenseful way**
```jsx
// Dogs.jsx
function Dogs() {
  const { data } = useQuery(GET_DOGS);

  return (
    <ul>
      {data.dogs.map((dog) => (
        <li key={dog.id}>{dog.breed}</li>
      ))}
    </ul>
  );
};
```
```jsx
// App.jsx
function App() {
  return (
    <React.Suspense fallback={<Spinner />}>
      <Dogs />
    </React.Suspense>
  );
};
```

Whereas the previous program was procedural, such as `if (isLoading)`, **the processing of the loading state has been made more declarative**. This simplifies the responsibilities of the component responsible for loading data.

The above is just an idea as a code example, but if you want to start using it in practice, you can start using Suspense for data fetching in React 18 by using frameworks such as Relay, Next.js, Hydrogen, and Remix. (*Not yet recommended as a general strategy in the sense that it is technically possible.)<br/>
In the future, they may provide new basic functionality to easily access data with Suspense without using a framework, so we look forward to future updates.<br/>
See [Suspense in Data Framework](https://reactjs.org/blog/2022/03/29/react-v18.html#suspense-in-data-frameworks)

### Other Use Cases

The following are other use cases, which are only referenced links.<br/>
Server-side components + Suspense is a feature that I am quite personally excited about.

- Server-side rendering capability in streaming
https://ja.reactjs.org/docs/react-api.html#reactsuspense-in-server-side-rendering
- Suspense during hydration
https://ja.reactjs.org/docs/react-api.html#reactsuspense-during-hydration

## Summary

Concurrent React is not only a better user experience, but as a developer, I felt that we need to design for the features that will be available with Concurrent React.<br/>
I am sure that both new features by Concurrent React and Concurrent support for the React ecosystem will be updated more and more in the future, so keep an eye on Concurrent React in the future.

## Reference

- [React 18に備えるにはどうすればいいの？　5分で理解する - Qiita](https://qiita.com/uhyo/items/bbc22022fe846fd2b763)
- [React v18.0 – React Blog](https://reactjs.org/blog/2022/03/29/react-v18.html#suspense-in-data-frameworks)