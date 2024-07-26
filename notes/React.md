# React

## React Introduction and ReactJs Setup

### React Introduction

React is released by the Facebook team in 2013. It's very popular javascript framework now.

### ReactJs Setup

```shell
npx create-react app <projectName>
```

We can use the scaffolding to initialize a React project.

![react_project_outline](/Users/zhanghaha/Desktop/learning_notes/notes/images/react_project_outline.png)

After we initialize the project, we can see the project structure in the file tree. There is only on html file `index.html` in it where we will inject our React component into it. The `App.js` is the parent component which is above all the future React components.



## JSX

> What is JSX?

JSX is javascript. The appearance of the JSX is similar to the html, but it's not quite the same. JSX allows us to put javascript expressions in the code. All in all, we could see the JSX provide a template for the component layout. It's also important to know that JSX renders data as text when display it.



## Functional Components

Now the React teams recommend us to use the functional components in our react projects.

Functional components are similar to the javascript functions, except that the first letter of the components should be capitalized. Then the compiler knows that it's a component, not a function.



## Styles

We can define the styles of the React component.

There are some different ways we can use. 

* Inline Styles
* Variables 
* Separate files

All of these ways can work fine in it. We can choose the ways we want to use.



## React Click Events

We can add a click event on the React component.

When we add the click event, we can pass parameters to the function or not.



## React List

Sometimes we may want to render a list items on the page, then we may need to iterate an array and create the item lists.

Here we need to notice that when we render the list item, we should provide a key attribute to the list item, because the React need to use the key to know which list item it will update, delete or reorder.

If we don't provide this value or we provide a key value which is not unique in the list, there will be some problem when we do some actions.



## Props and Prop Drilling

When we define a component, we may accept some props from the parent component. Then we need to pass the props from the parent component to the child component.

And if there are multiple hierarchies in these components, we may drill down the prop to the child component.



## Controlled Component

The controlled component pattern involves managing the component's state through React's state system. The component's state is controlled and updated through the React, and changes are reflected through props. 

This pattern allows a more predictable and controlled flow of data in the application.

Usually, the controlled component is referring the `input`  and `form` component.



## UseEffect

Here we will discuss when `useEffect` runs.  It will runs when a component is rendered. 

Usually, we will use the hook with dependencies. If you pass an empty dependency to it, then the hook will runs only when the component is loaded. If you pass some dependencies to it, then the hook will be invoked after the dependencies are changed.

There is one thing we need to notice is that `useEffect` is not synchronous, it is asynchronous. It will run after the components are loaded or updated.



## Json Server Rest API

Sometimes, we may need to send a request call when we want to fetch some data from a server and show the data in the App. Then we can use the `json-server` to mock the API call.

**Step 1**

Create a data folder and the related mock data file.

**Step 2**

```shell
npx json-server -p 3500 -w data/db.json
```

You may need to use the node which version is greater than 18.

When we start the local server with the json server, it will provide us a API root url.

We can make operation like creating, deleting, updating and reading through API with it.And the related method will be POST, DELETE, PATCH and GET.

> There is a website called JSONPlaceholder. You can send a request to this server, and get some face data for testing. It is powered by JSON Server + LowDB.

[JSONPlaceholder](https://jsonplaceholder.typicode.com/)



## Rules of hooks

* Call hooks from react function components
* Call hooks from custom hooks



## Custom hooks

A custom hook is a React function whose name starts with  `use` and that may call other hooks.

When we want to share logic between different JavaScript functions, we extract it to a third function. Both components and hooks are functions, so this works for them too!

```react
import { useEffect, useState } from "react";

const useWindowSize = () => {
  const [windowSize, setWindowSize] = useState({
    width: undefined,
    height: undefined,
  });

  useEffect(() => {
    const handleReize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    };

    handleReize();

    window.addEventListener("resize", handleReize);

    return () => window.removeEventListener("resize", handleReize);
  }, []);

  return windowSize;
};

export default useWindowSize;
```



## State Management - `useContenxt` API 

When we develop an application, there will be some states which may be used in different components, we can make these states as a global states and pass them into a context provider. We don't need to drill these props down to all the components.

The states which are only used in a component, it can be contained within the component, it should not be included in the context provider.



### Usage of Context

* reate a context with the method `createContext`

```react
const DataContext = createContext({});
```

* Create a context provider

```react
export const DataProvider = ({ children }) => {
  return (
    <DataContext.Provider
      value={{
        ...
      }}
    >
      {children}
    </DataContext.Provider>
  );
};
```

We can create more than one context if we need.

* Wrap the components which will share the states in the Provider

```react
function App() {
  return (
    <div className="App">
      <Header title={"React JS Blog"} />
      <DataProvider>
        <Nav />
        ...
      </DataProvider>
      <Footer />
    </div>
  );
}
```

* Get the states from the context with hook `useContext`

```react
const { posts, setPosts } = useContext(DataContext);
```

The `DataContext` is the context which you created above.



## easy-peasy

`Easy Peasy` is an abstraction of Redux.

It provides a reimagined API focussing on **developer experience**, allowing you to **quickly** and **easily** manage your state, whilst leveraging the strong **architectural guarantees** and providing integration with the extensive **eco-system** that Redux has to offer.

[easy-peasy](https://easy-peasy.vercel.app/docs/introduction/)



## Deploy your application to Netlify

* Initialize your project with `git init`
* Create a repository in Github and push your local repository to your remote repository
* Log into the Netlify with your Github account
* Create a new site with your project on Github
* Deploy the selected project to Netlify
* Open the link after you complete the deployment

[Netlify Login](https://app.netlify.com/)





## useCallback

The hook is used to cache a function definition  between re-renders.

```react
const cachedFn = useCallback(fn, dependencies)
```

On the initial render, `useCallback` returns the `fn` function you have passed.

During subsequent renders, it will either return an already stored `fn`  function from the last render (if the dependencies haven’t changed), or return the `fn` function you have passed during this render.

```react
const sum = useCallback(() => {
    console.log(num1, num2);
    return num1 + num2;
}, [num1, num2]);

useEffect(() => {
    console.log(sum());
}, [sum]);
```





## useMemo

`useMemo` is used to cache the results of a function, and it will only recalculate the value only when the dependencies are changed.

The difference between `useMemo` and `useCallback` is that `useMemo` is used to cache the results of a function, but `useCallback` is used to cache the definition of a function. We can use them according to different situations.

```react
  const getArray = () => {
    for (let i = 0; i < 1000000000; i++) {
      //do something expensive
    }
    return ["Dave", "Gray"];
  };

  const fib = useCallback((n) => {
    return n <= 1 ? n : fib(n - 1) + fib(n - 2);
  }, []);

  const fibNumber = useMemo(() => fib(userNumber), [userNumber, fib]);

  const myArray = useMemo(() => getArray(), []);

  useEffect(() => {
    console.log("New array");
  }, [myArray]);
```



## useRef

`useRef` is a React Hook that lets you reference a value that's not needed for rendering.

* The ref object will keep the same between re-renders.

So you can store information that between re-renders.

* Changing it will not trigger a re-render.

So refs are not appropriate for storing information you want to display on the screen.

```react
const ref = useRef(null)
```



## useReducer

`useReducer` is a React Hook that lets you add a `reducer` to your component.

In a real project, there are usually some different states in a component, then we can use `reducer` to manage the states.

```react
import { useReducer } from 'react'

const reducer = (state, action) => {
  switch (action.type1) {
    case increment:
    	return {...state, count: state.count + 1}
    case decrement:
      return {...state, count: state.count - 1}
      defualt:
      throw new Error()
  }
}

const [state, dispatch] = useReducer(reducer, {count: 0})
```

With reducer, when need to pass the state down to the child components, we only need to pass two props, it's more simple.



## useEffect VS useLayoutEffect

### useEffect

`useEffect` is a React Hook that lets you synchronize a component with an external system.



### useLayoutEffect

**`useLayoutEffect` is a version of `useEffect` that fires before the browser repaints the screen.**

React guarantees that the code inside `useLayoutEffect` and any state updates scheduled inside it will be processed **before the browser repaints the screen.** 



## useImperativeHandle

`useImperativeHandle` is a hook that lets you customize the handle exposed as a `ref`.

By default, components don't expose their DOM nodes to parent components, if a parent component want to access to the DOM nodes inside the child components, we need to import the `forwardRef` and wrap the child components in it, then the parent component can access all the DOM nodes inside the child components.

However, if you don't want to expose the entire DOM nodes, you can use the `useImperativeHandle` and expose the handle a custom value instead.

```react
import { forwardRef, useRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => {
    return {
      focus() {
        inputRef.current.focus();
      },
      scrollIntoView() {
        inputRef.current.scrollIntoView();
      },
    };
  }, []);

  return <input {...props} ref={inputRef} />;
});
```

Now, if the parent component gets a ref to `MyInput`, it will be able to call the `focus` and `scrollIntoView` methods on it. However, it will not have full access to the underlying `<input>` DOM node.
