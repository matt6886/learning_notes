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
