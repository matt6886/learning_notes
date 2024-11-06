# React Native

* **React Native allows developers who know React to create native apps.** 
* React Native brings the best parts of developing with React to native development, it's a best-in-class JavaScript library for building user interface.

* Write in JavaScript, rendered with native code.
* React Native lets you create truly native apps and doesn't compromise your users' experience. It provides a set of platform agnostic native components like `View`, `Image`, and `Image` that map directly to the platform's UI native building blocks.

## Environment setup

We believe that the best way to experience React Native is through a framework, a toolbox with all necessary APIs to let you build product ready apps.

Without a Framework, you'll either have to write your own solutions to implement core features or you'll have to piece together a collection of pre-existing libraries to create a skeleton of a Framework.

### Expo

`Expo` provides features like file-based routing, high-quality universal libraries, and the ability to write plugins to modify native codes without having to manage the native files.

```shell
npx create-expo-app@latest
```

## Workflow

### Metro

> Metro is a JavaScript bundler, it take in an entry file and various options and gives you back a single JavaScript file that includes all your codes and its dependencies.

Metro has three separate stages in its building process.

* Resolution
* Transformation
* Serialization

#### Resolution

Metro needs to build a graph of all modules that are required from the entry point. To find which file is required from another file Metro uses resolver. In reality this stage happens in parallel with the transformation stage.

#### Transformation

All modules go through a transformer. A transformer is responsible for converting a module to a format that is understandable by the target platform(eg. React Native). Transformation of modules happens in parallel  based on the amount of cores that you have.

#### Serialization

As soon as all the modules have been transformed they will be serialized. A serializer combines the modules to generate one or multiple bundles. A bundle is literally a bundle of modules combined into a single JavaScript file.

## UI && Interaction

### Layout with Flexbox

A component can specify the layout of its children using the Flexbox algorithm. Flexbox is designated to provide a consistent layout on different size screens.

## Performance

### Performance overview

A compelling reason to use React Native instead of WebView-based tools is to achieve at least 60 frames per second and provide a native look and feel to your apps.

#### Frame

Realistic motion in video is an illusion created by quickly changing static images at a consistent speed. We refer to each of these images as frames.

#### JS frame rate(JavaScript Thread)

For most React Native applications, your business logic will run on the JavaScript thread. This is where your React application lives, API calls are made, touch events are processed. Updates to native-backed views are batched and sent over to the native side at the end of each iteration of event loop, before the frame deadline.

#### UI frame rate(main thread)

## Understanding single thread concept of JavaScript

JavaScript is a single thread language, which means it can only do one thing when executes codes. It will manage the sequence of tasks depending on an event loop. Through the mechanism, JavaScript can handle asynchronous operations, such as network request, timer and so on, but it still keep the feature of single thread.

### Event Loop

It is a mechanism of JavaScript engine for managing and coordinate the sequence of executing tasks. And it runs on the main thread of JavaScript.

The main task of event loop is monitoring the Call Stack and Task Queue, and execute tasks as their according sequence.

**work flow**

1. Execute global codes: when the JavaScript engine begin to execute codes, it will execute the synchronous codes in the global context, and these codes will be put into the Call Stack and executed accordingly.
2. Event loop start: when Call Stack is empty, event loop will run.
3. Check the micro tasks: event loop will check the micro task queue and execute all micro tasks until it is empty.
4. Execute macro tasks: when micro task queue is empty, event will pick up one task from macro task queue and execute it.
5. Rendering updates: the browser may execute rendering updates and reflect the changes of DOM at the end of each event loop cycle.
6. Repeat above step: event loop will repeat above steps until there is not any task which is needed to handle.

### Task Queue

The tasks are divided into two categories in the mechanism of JavaScript's event loop, macro tasks and micro tasks.

* Macro Task
* Micro Tasks

#### Macro Task Queue

The macro tasks will be added into the macro task queue.

Macro tasks include:

* setTimeout
* setInterval
* setImmediate(Node.js environment)
* I/O operation
* UI rendering

The tasks will be executed as the sequence that they are added into the queue. It will only pick up one task and execute it in each loop cycle of the event loop.

#### Micro Task Queue

The micro tasks are added into the micro task queue. The priority is higher than the macro tasks.

Micro tasks includes:

* callback function of Promise(functions registered through then/catch/finally)
* MutationObserver
* Process.nextTick(node.js environment)

Event loop will execute all micro tasks in micro task queue after finishing the micro task each time, only when the micro task queue is empty, it will continue to execute next macro task.









