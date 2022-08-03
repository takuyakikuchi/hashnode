## Why is Vite so fast?

Click here to read the article in Japanese：https://zenn.dev/takuyakikuchi/articles/36eae991f0239d

# Purpose of this article

In The State of JS 2021's Build Tool Rankings, Vite won first place for **satisfaction** and **interest**. 
![the-state-of-js-2021.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659528206817/8bIlwGqtH.png align="left")
https://2021.stateofjs.com/en-US/libraries/build-tools/

It is so popular and we hear a lot about it these days, but I didn't know much about Vite except that it is told: "it's fast!"<br />
This article tries to sort out the basics of how it works and understand why it is fast.

# What does Vite do?

First of all, let's start with what Vite does. Vite is responsible for **building a development environment** and **building a production environment**.<br />
The development environment is built using [esbuild](https://esbuild.github.io), which is developed in Go language(Go is very fast)<br />
On the other hand, the build command for the production environment uses [Rollup](https://rollupjs.org/guide/en/).<br />
The reason why the tools used for building the development environment and the build command for the production environment are different is that although esbuild is super fast, some important functions required for bundling applications are still under development, so Rollup, which is the best solution at this point, is used. (See: https://vitejs.dev/guide/why.html#the-problems)

Tools in the same position as Vite would be **webpack**, **parcel**, **Snowpack**, etc.

# Bundles are slow. Using the browser's native ESM will make it faster
Let's quickly touch on the main question, "Why is Vite fast?

## Traditional bundler-based build setup
First, let's take a look at the attached image below to see how a conventional bundler-based build setup such as webpack works.<br />
As a rough explanation, in a bundler-based build setup, multiple pieces of JavaScript code are combined into one large piece of code through a process called **bundle** before the application is served and then adjusted to make it executable in the browser.<br />
Because it generates large code containing the contents of the entire application, the **bundler-based build setup mechanism has the problem that the larger the application, the slower it will inevitably become**.

![bundle-based-dev-server.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659527422290/sziTIBk_C.png align="left")
(Image source: https://vitejs.dev/guide/why.html)

## Build setup using native ESM
Vite, on the other hand, uses **native ESM** to speed up the process.
To have a rough idea, let's look at the attached image below and what native ESM is.

### Native ESM
What is native ESM? Native ESM is ES Modules. (Syntax to import with `import` declaration and export with `export` declaration).<br />
ES Modules are a system of modules defined as part of the ECMAScript specification and understood directly by the browser.<br />
By "using native ESMs," I mean taking advantage of the browser's direct understanding of ES Modules and having the browser load them directly, with the post-build outputs as multi-file modules.<br />
By using this native ESM, Vite eliminates the heavy process of bundling the entire application, thus making it possible to speed up the process.

![native-esm.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659527814769/bj55fuiae.png align="left")
(Image source: https://vitejs.dev/guide/why.html)

### Vite process to use native ESM
As I said, Vite uses native ESM. As a preliminary process for this, the application modules are divided into the following two categories.
- **Dependencies**: Plain JavaScript that will not change much during development (e.g. component libraries such as MUI)
- **Source code**: JSX, CSS, Vue/React and other components that require conversion

**Dependencies**<br />
The code to be split into dependencies is converted from CommonJS or UMD (Universal Module Definition) to ESM by a process called **Pre-Bundling**. The purpose of this process is below
- To make the code available to browsers as an ECMAScript module.
- To improve page load performance. (The process converts a dependency with many internal modules into a single module.)
  
Pre-bundling also uses esbuild, which means it is 10 to 100 times faster than JavaScript-based bundling.
[Pre-bundling of dependencies | Vite](https://vitejs.dev/guide/dep-pre-bundling.html)

**Source Code**<br />
Vite will only convert source code and serve it using native ESMs upon browser request.

## Fast updates when changing files during development
Due to the above mechanism, Vite is comfortable developing without re-bundling during development.<br />
HMR (optimization by replacing only the modules that have changed, rather than reloading the entire application) is performed on the native ESM, ensuring consistent and fast execution regardless of the size of the application.

# Summary

I have been using Vite recently, and I can really feel the speed, and development is very comfortable.
I encourage everyone to give Vite a try.
You can also try Vite online at StackBlitz
https://stackblitz.com/edit/vitejs-vite-sqdtjb?file=javascript.svg&terminal=dev

# Reference

- [Why Vite | Vite](https://vitejs.dev/guide/why.html)
- [Native ESM時代とはなにか](https://zenn.dev/uhyo/articles/what-is-native-esm-era)