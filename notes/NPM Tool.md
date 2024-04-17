### NPM

## What is NPM?

> NPM is a node package manager

## How to use it?

> NPM is shipped with the node. So if want to install NPM, you just need to download the node and install it, then the NPM will be available.

## Package.json

We can create the package.json file with the command `npm init` or `npm init --yes`.

And we can config the default value for some fields.

For example, we can config the default value for the author and license.

```shell
npm config set init-author-name=Matt
npm set license=MIT
```

## Install package locally

We can install some packages in our project with the  command `npm install`.

When we execute the command `npm install moment`, the package will be download from the NPM service, it may take some minutes to install it.

Then if you open the package.json file, there will a new filed call dependencies, and the package you just installed will appear here with the package name and the specific version number.

We can also install the package just for development environment.

```shell
npm install lodsh --save-dev
```

With the command, the package will just be installed for the dev environment.

If you open the package.json file, the package and version will appear under the devDependencies.

## dependencies VS devDependencies VS peerDependencies

### dependencies

If you run the command `npm install <packageName>`, you'll find that the specific package will appear in the dependencies section in the package.json file. It will install the packages in dependencies section in default.

Dependencies packages are required for you running your application.

### devDependencies

If you install a package followed by the option `--save-dev`, then the package will just be installed for the development environment. And you'll find the package will land directly in the devDependencies section. That means these package don't need in the production environment.

You can run the command `NODE_ENV='production npm install'`, then you may find the packages under the devDependencies will not be installed for your project.

### peerDependencies

peerDependencies are not real dependencies, actually we will not need it when we run our own application, it usually appear in the  NPM package when you are gonna to issue a new package.

Sometimes when you install a package, the new package may depend on another package, and if you list this package in you dependencies section, there may be a problem. When the others install the package, they may find the dependent package has been installed with a different version, then there may be multiple same package to be installed.

So peerDependencies appear, if you don't install the  dependent package, there will be a warning displaying on the screen and tell you that you may need to install the particular package. And if you have installed, there is nothing need to do.

## Install packages globally

```shell
npm install typescript -g
```

With the option `-g`, you can install the packages globally.

## local modules VS global modules

If you want to install the node packages under your project, you just need to use the command `npm install <packageName>`, but if you want to install the package globally, you need to follow another option -g, like this `npm install <packageName> -g`.

You can check the local packages in the node modules folder of the project, and if you want to check the global packages you installed globally, you may use the command `npm root -g` to check the path of the global modules. Then you can go to the folder and check the global packages.

![npm_root](/Users/zhanghaha/Desktop/IT/Learning/vs code/images/npm_root.png)

## Behind the command `npm install`

When a developer created a node package and upload it to the NPM service, all the source files will be hosted in NPM service. And the service will need a `.tgz` format file.

If you execute the command `npm view lodash`, you will see there is a link called tarball, this is the source files of the package, and if you paste it into the browser, a file with `.tgz` extension will be download to your local folder. If you decompress it, you can see all the source files.

![npm_view](/Users/zhanghaha/Desktop/IT/Learning/vs code/images/npm_view.png)

When we execute the command `npm install lodash`,  it will download the package with the link and save it in node modules folder of your project.

## List packages

```shell
npm list
npm list --depth 1
```

You can check the packages installed locally.

```shell
npm list --global true
npm list --global true --depth 1
```

With these commands, all the packages installed globally will displayed.

## NPM semantic versioning

If you install a package but not specify the version, then the latest version will be is installed.

But the you want to install a specific version, then you need to specify the version number. There are three numbers about the version which is separated by the dot symbol, like this `4.17.21`. 

The first number represents the major version. When there are some break changes or some incompatible API changes, the number will be changed. The second number is the minor version, when you add functionality in a backwards compatible manner,  you may update this number. The last number is the patch version. Patch version may be changed when you make a backwards compatible bug fixes, such as something like that.

When you specify the version, you can just specify the major number or minor number, or both of them, you can also specify all the three numbers.

```shell
npm install lodash@4
npm install lodash@4.16
npm install lodash@4.16.1
```

Even though there is a semantic versioning, but most developers may don't follow the rule. It causes that when you update a package which just minor number changed, but it is possible that there are some break changes for the new package. It's a pain.  But we should also know the semantic versioning rule. 

## package-lock.json file

If you work on a project which is using the NPM which version is greater than or equal to 6, then there will a package-lock.json file in it.

**So why is there a package-lock.json file**

Imagine that when you install a package which version number is `^4.0.0` in the package json file, and after a time, you need to change to another machine, and you will setup the project environment again, but when you execute the `npm install` and run the project, you find your project crash.

You will be curious what happened to your project, you didn't make any changes, you just reinstall the dependencies.

If you check some package, you will find that the package has beed updated to another version, and the previous API doesn't work in the new version. That's why your project doesn't work properly any more.

That's why the package-lock.json file appears. It will lock specific version numbers of all the dependencies you used in your project. So when you change to another system or a new machine, it will also install the previous package and make you run in the same environment.

## Install from package.json

When you open the npm project, there may be a readme file, and it will tell you execute the `npm install` command. That is because there is a `package.json` file listed all the dependency packages.

When you execute this command, all the packages will be installed locally.

You may see that there is caret symbol at the beginning at the version, like this.

```shell
"dependencies": {
    "lodash": "^4.17.21",
    "moment": "^2.30.1"
 }
```

The caret symbol means that it will retrieve the minor version and the patch version not the major version.

And you will also see the tilde symbol instead of the caret symbol in front of the version. For this kind of version, when you execute the installing command, it will retrieve the minor version, but the major and minor  version will keep as they are.

You can leave nothing at the beginning of the version, then even there is a newer version, it will still install the specific version.

## Update package

You may need to update the packages. 

If you want to update all the packages, you can execute the command `npm update`, the all the packages listed in the package.json file will be updated to the latest version.

If you just want to update some specific package, then you need to specify the specific package name, like this.

```shell
npm update lodash
```

Then this package will be updated to the latest version.

You can also update the global package with the top command followed by the option `-g`.

```shell
npm update typescript -g
```

If you don't specify the package name, all the global package will be updated.

And you can update the npm version as well, definitely, you need the administrator privilege.

```shell
npm update npm@latest -g
```

## Npm Prune

Sometimes there may be some packages exist in the node_modules folder, but not in the package.json file, then you may need to remove these extraneous packages.

You just need to run the command `npm prune`, then all the extraneous packages will be removed.

## NPM Script

There is scripts property under the `package.json` file which supports a number of scripts and their preset life cycle events as well as arbitrary scripts.

![npm_scripts](/Users/zhanghaha/Desktop/IT/Learning/vs code/images/npm_scripts.png)

We can use the built-in scripts and life cycle scripts to do some other things when we execute the scripts.

- Built-in scripts

> start, test...

- Life cycle scripts

> prepare, prepack, postpack

## NPX

There is a utility call NPX. It mainly has two functions for this tool.

- Execute the local binaries

When you are in the project folder, and you execute a command of a specific package, if you execute the command directly in the terminal, it won't work. You need to add the `npx` before the command. But if you put the command under scripts section of the `package.json` file, then it will work.

![npx_local](/Users/zhanghaha/Desktop/IT/Learning/vs code/images/npx_local.png)

- Execute the global binaries

When are out the project box, when you execute a command of a specific package, it will first download it and then execute command.

![npx_global](/Users/zhanghaha/Desktop/IT/Learning/vs code/images/npx_global.png)

## Change the installation location of global NPM packages

You can check the current installation location of the global NPM packages.

```shell
npm root -g
```

> /usr/local/lib/node_modules

You will find all the global NPM packages under this path.

If you want to change it to another place. You can create a new folder and change the NPM config file.

```shell
npm config set prefix <NEW PATH>
```

You can view all the configuration of the NPM.

```shell
npm config ls -l
```

## Cache mechanism of NPM

When you install a NPM package, the package will be saved in the NPM cache folder, you don't need to hit the network the next time when you want to install the same version of the package.

It's very convenient, and will save you a lot of time. Sometimes you know, there may be a very large packages you need to install, with this mechanism, you can install the package quickly.

```shell
cd .npm
```

In the top folder, you can see all the cache files you have installed.

If you want to clean the cache, you can execute

```shell
npm cache clean --force
```

