# TypeScript

## Inro

### What is typescript?

TypeScript is a alternative of JavaScript, it's a superset of JavaScript.

### Why does we need the typescript?

If you are a web developer, you know that the JavaScript has some problems. It's a dynamic language, the variables can be changed from one type to another easily. 

With TypeScript, you don't need to worry about this kind of issues. **TypeScript allow us to use strict types.**

The second important of typescript is that you can use the modern features of JavaScript as you like. You know not every browser support the modern features of JavaScript, such as arrow function, let, const.

And the TypeScript also provide some extra specific features, like generics, interface and tuple.

With TypeScript, you can program more easily and more productive. But when you are going to deploy the project which is using TypeScript, you need first compile it into the regular JavaScript, this is also very easy to be done. You just need to download the TypeScript compiler and use it to complete the compilation.



## Compiling TypeScript

We can write codes in a TypeScript file freely, when we are going to deploy it, we first need to compile it into the regular JavaScript. As we mentioned before, we can transform it with the TypeScript compiler more easily .

```shell
tsc <ts file>
```

We can use the top command compile the specific TypeScript to a JavaScript file easily. If the according JavaScript file exists, then the content in the file will be changed accordingly. If not, then it will generate a new JavaScript file with the same name and `.js` extension.

If we want watch the change of the TypeScript file and compile it into the JavaScript file, we can use the below command.

```shell
tsc <ts file> -w
```

The `w` represents watch.



## Type Basics

As we know, we can change the type of variable to another type easily in regular JavaScript, but we can't do this in TypeScript, this can prevent us making such kind of mistakes.

When you populate a variable with a specific value, the TypeScript will infer the type of the variable, you don't need to explicitly the type. And when you populate it with another kind of value, there will be a red wave line under the variable. Then you know you made some mistakes and you can change it.

![ts_type_basics](./images/ts_type_basics.png)