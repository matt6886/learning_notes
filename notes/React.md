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
