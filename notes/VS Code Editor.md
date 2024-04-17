# VS Code Editor

## VS Code UI

There are mainly some part in the VS Code Editor. It includes activity bar, sidebar, editor, tabs, status bar and panel at the bottom.

## Command Palette

You can use the key Command + Shift + P to open the command palette.

With the command palette, you can do many things without leaving the current location.

![command_palette](/Users/zhanghaha/Desktop/IT/Learning/vs code/images/command_palette.png)

## VS Code Color Theme & Icon Theme

You can customize your color theme and the icon theme of your vs code editor.

First you may need to download an extension from the extension market. There are many popular related extensions, like `Github Theme`, `Winter is Coming Theme`, `Material Icon Theme`. You can select a preferable color and icon theme for you vs code editor.

## VS Code Font Family

We can configure the font family in the vs code.

**Popular Font**

[Cascadia Code](https://github.com/microsoft/cascadia-code)

[Fira Code](https://github.com/tonsky/FiraCode)

First, we need to download a related font from the Github, then install it on our computer for all users or current user. And then we can configure it in the vs code.

- Press Command + Shift + P,  
- Enter setting and open it, 
- earch 'font family', and enter the font that we want to use.

![font_family](/Users/zhanghaha/Desktop/IT/Learning/vs code/images/font_family.png)

We can set the font size and line height as well in the vs code.![font_size](/Users/zhanghaha/Desktop/IT/Learning/vs code/images/font_size.png)

![line_height](/Users/zhanghaha/Desktop/IT/Learning/vs code/images/line_height.png)

There is a website where you can download a font that you like.[coding font](https://coding-fonts.netlify.app/fonts/codelia/)

## VS Code Appearance Setting

### Tab Size

We can configure the tab size in the settings.

![tab_size](/Users/zhanghaha/Desktop/IT/Learning/vs code/images/tab_size.png)

### Rulers

We can use this setting to render vertical rulers after a certain monospace characters.

![rulers](/Users/zhanghaha/Desktop/IT/Learning/vs code/images/rulers.png)

### Render Indent Guides

This setting defaults on. It controls whether the editor should render indent guides.

![render_indent_guides](/Users/zhanghaha/Desktop/IT/Learning/vs code/images/render_indent_guides.png)

### Word Wrap

It controls how lines should wrap.

![word_wrap](/Users/zhanghaha/Desktop/IT/Learning/vs code/images/word_wrap.png)

### Cursor Blink

It controls the cursor animation style.

![cursor_blink](/Users/zhanghaha/Desktop/IT/Learning/vs code/images/cursor_blink.png)

## VS Code Explorer

We can open the explorer through `Command + B`, here in this section you can see the editor, folders, outline and timeline.

In the open editor section, you can see the files you are editing, with the outline, you can find the outline of the current active file.

## VS Code Editor

We can split the editor window through many ways.

And we can define the startup editor page when we launch the vs code editor.

- `Command + Shift + P`, search `startup editor`, select a option

![startup_eidtor](/Users/zhanghaha/Desktop/IT/Learning/vs code/images/startup_eidtor.png)

- Define the language when start the vs code editor.

![language](/Users/zhanghaha/Desktop/IT/Learning/vs code/images/language.png)

## Find and Replace all things in VS Code

There are three options you can use to search and replace the things.

- option 1

`Command + F`

- option 2

Click the search button on the activity bar

- option 3

Click the button 'open the new search editor' in the search section bar

When you get the related results in the vs code, you can press enter key to review all the results one by one.  If you press the `option` and `Enter` key at the same time, then all the results will be selected and in editing status, you can edit them at the same time, it's very useful when you want to edit them simultaneously.

We can use `find all references`, `peek definition` and `rename symbol` functions to improve our programming productivity.

## Refactoring in VS Code

- Put the cursor in the functions or variables that you want to refactor
- Right click and select the option `rename symbol`
- Make the change and press enter key

Through the top steps, you can change all the references at one time.

## Extension installation

We can download a specific extension from the VS Code market or install extensions from the vs code editor.

- lick the extension button
- Type the extension you want to install
- Select the one and click install 

You can disable extensions that you don't need  to use in your current project, because the extensions that you won't use in the project may impact the performance. So you can disable them.

If you work in a small team or a large team, you can configure the recommended extensions. Copy the extension ID from the extension page, and paste it on the recommendations fields.

Then others who download the project and open it, there will be a prompt showing 'do you want to install the recommended extensions'. 

![recommended_extesions](/Users/zhanghaha/Desktop/IT/Learning/vs code/images/recommended_extesions.png)

## Setting Sync in VS Code

You can sync your settings.

- Click the account button and log in with your Github account
- Then when you log on another computer, the settings and extension will automatically sync to the laptop. It's very useful. You can have a try.

## Using Snippet in VS Code

### Using Snippet extension

You can install a snippet extension.

- Type` @category:"snippets"` or `@snippets`
- Install a specific snippets extension
- Use it follow the docs of the extension

### Define your own snippets

- `Command + Shift + P`
- Type `configure user snippets`
- Select `New global snippets file`
- Define your own snippets
- Use it in your project

## Using Emmet in VS Code

Emmet is an essential toolkit for web-developers.

[emmet](https://emmet.io/)

![emmet1](/Users/zhanghaha/Desktop/IT/Learning/vs code/images/emmet1.png)

![emmet2](/Users/zhanghaha/Desktop/IT/Learning/vs code/images/emmet2.png)

## VS Code Command Line Tools

- `Command + Shift + P`
- Type `install code` and select the `install code command in PATH`
- Use the `code` command  line tool in your terminal

```shell
code getting-started-with-javascript-main
```

## Best HTML and CSS extensions

## color highlight

With this extension, the style of the color will be highlighted with the specific color.

## CSS Peek

This extension is used to go to the css definition.

## Code Spell

It is used to inspect the spelling of the word.

## HTML End Tag

The end html tags will be marked with a suffix.

## HTML CSS Support

You can use this extension to complete the css style exported from external file or link.

- `Command + Shift + P`, open setting
- Search css stylesheet, edit the json file, paste the link

## Extensions using in npm and node

### NPM

You can use this package to initialize the package.json.

Now this extension is deprecated.

### Gitignore

With this extension, you can add a gitignore file, and add the files you don't want to track in it.

### NPM intellisense

This extension will autocomplete the modules in  import statements.

### Version Lens

This extension shows  version information when opening  a package or project.

### Import cost

This extension will display the size of imported package.

## Using Bulit-in Javascript Feature in VS Code

### Intellisense

### JSDoc

### Auto Imports

### Code Actions on save

### Update imports on move

## Using ESLint in VS Code

> What is ESList?

ES is a powerful tool which can help you find the syntax and problems in your project.

> How to use it in your project?

- Install ESLint extension in your VS Code
- Install ESLint package in your project
- Configure `.eslintrc` file in your project

## Best JavaScript Extensions in VS Code

### JavaScript(ES6) code snippets

It's snippets extension.

### Path intellisense

I think VS Code has built it in now.

### Turbo console log

A debugging tool which can make you log more meaningful.

### JavaScript Booster

It will help you refactor you code just with some actions.

## React Extensions

### Simple React Snippet

### JavaScript Booster



## Vue Extension

### Vetur

### Vue VSCode Snippets

### Vue Code Extension Pack



## Tailwind Extensions

### TailWind CSS Intellisense

### Tailwind Docs

### Headwind



## Markdown Extensions

### Markdown Preview Github Stying

It's used to change the default markdown preview style to Github preview style.

### Markdown all in one

It includes all the markdown syntax you need.

### Markdonwlint

You can use it to check the style of markdown file.



## PHP Extensions

### PHP Intelephense

### phpfmt

### Laravel Extra Intenllisense

### Laravel Blade Snippets

### Laravel Blade Spacer



## Quick ways to make VS Code look better

## indent guides

### line height

Recommended  line height is 38.

### breadcrumbs

### Theme: One Dark Pro



## Using Terminal in VS Code

We can use the built-in terminal in vs code or external terminal application. It's up to you.

Press Command + ` will open the terminal interface.

We can split the terminal into more than one parts or open more one terminals.  You can maximize the terminal or open the terminal on the right side of the eidtor.

In the settings, we can customize the terminal size, line height and customize the external terminal application in the vs code.



## Using Git and Github in VS Code

First, we need to initialize the folder with `git init`.

Then, there will be a .git folder generated in current folder. All the changes will be stored here.

When we make some changes in some file or add something in current folder, the changes will appear in the source control panel.

We can stage  and commit the changes through command or UI interface in source control panel.

In the source control panel, we can initialize or push a remote repository from the Github.

In the settings, we can configure the default clone repository.



## Git Extensions

### Git History

It is used to view git log.

### Git Blame

You can see the git blame changes in the status bar.

### gitlink

You can open the online link of current file.

### Git Indicators

If you want to hide the activity bar, but want to see the git changes, you can use it.

### Gitlens

It will supercharge your git experiences in vs code.



## Using multiple projects in VS Code

We can add multiple projects into vs code. We just drag the project folder to the vs code and add it to the workspace. You can switch between these projects you added freely.

There are two common extensions about multiple projects. The first one is `project manager`, with this extension, you can manage you projects easily.

The other one is `peacock`, this is a extension for switching the color of the projects.



## Autosave and Autoformat

With autosave feature, the vs code will save the code automatically when you stop typing. You can configure it in the settings.

Autoformat feature is good function. You can save the code with a default formatter when you press on `Command + S`, it will format your code automatically when you save your code. It's powerful and efficient.



## Get a browser in VS Code

Here are two related extensions you can use it to preview the code with chrome in the vs code.

### live server

A live server that can make your changes take effect right now.

### live preview

It is still under development. With this extension, you can preview our code in the chrome in the vs code.



## Keyboard shortcut

There are three ways to open the keyboard shortcut reference.

- `Command + K + Command + R`
- `Command + Shift + P`, select `open keyboard shortcut`
- `Command + Shift + P`, select `help: keyboard shortcut reference`



## Basic Editing keyboard shortcut

First, we can begin with the basic editing keyboard shortcut.



## Navigation Keyboard Shortcut

`Command + P`

`Command + Shift + P`

`Command + Shift + O`

`control + G`

`Command + Shift + \`



## Multi Cursor and Selection

## Keyboard Shortcut for VS Code's UI



## Github Pull Request and Issues

There is an extension called `Github Pull Request` which can review your pull request and issues directly in the vs code.



## Editing Github Remote repos

Here is a Github extension called `remote repositories`, you can edit the code without downloading the whole repo from the Github, you can edit the code online.

## Called API 

When we want to debug the API, we may use the postman tool, this is a powerful api tool.

But if you want to debug the API without leaving the vs code, there are two extensions you can use.

One is `REST Client`,  with this tool, you can create a new file and change the language to `HTTP`, then you can paste the API and according parameters into the new file, and then you can click the `Send Request` button, there will be a response file generated on the right side of the new file.

The Second extension is `Thunder Client`, it is more powerful, and is similar to the postman tool. There are good user interfaces, you can configure all your parameters in the interface.

## Vim in VS Code

We can use vim to programing in the VS Code.

We just need to install a vim extension in the vs code, there are two extensions about vim. One is a simple vim extensions includes some main vim features. And there is another vim extension called `Vim` with the whole vim features.

If you want to learn vim, you can try to install the `Learning Vim` extension. You can learn the vim feature with it.

[Pokeapi](https://pokeapi.co/)



## Artificial Intelligence Coding Helper

There are two AI coding helper you can use in your programming.

One is `Tabnine`,  the other is `Github Copilot`.

You can improve your programming efficient with them.



## Right Side Bar 

You can make the primary side bar display on the right side of vs code. Right Click the activity bar, and select move the primary side bar right, then the side bar will appear on the right side.





