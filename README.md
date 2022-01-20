# JavaScript-to-TypeScript-migration-guide

If you've reached here, you might already know that TypeScript is a superset of JavaScript and that TypeScript is eventually transpiled to JavaScript so that it can be run. That begs the question, **why would anyone want to migrate to TS?**

## Why do it?
There are several reasons why one might want to convert his JavaScript project to TypeScript, with the most obvious being gaining the ability to detect errors while writing code and not during compilation.

Also, even though the size, scope, and complexity of programs written in JavaScript has grown exponentially, the ability of the JavaScript language to express the relationships between different units of code has not. Combined with JavaScript’s rather peculiar runtime semantics, this mismatch between language and program complexity has made JavaScript development a difficult task to manage at scale.

TypeScript can help you fill that mismatch by providing OOP like features that can help you model clearer relationships between different parts of your code.

But that typically only applies to new projects, because converting a large project - written using functional programming paradigm - into OOP's class based structure requires a lot of time and resources, so much so that you're better off writing the whole thing from scratch.

So, the main reason for converting an existing project to TypeScript would be to improve the developer's experience and possibly find errors before run-time (especially useful for projects where each deployment takes a long time).

So, now that you're here, it's always good to validate that you're doing this for the right reasons.

Here are some of the good reasons for deciding to migrate to TypeScript:

- Taking advantage of its Type Annotations
- Language Features provided on top of JavaScript
- Enhanced Intellisense | API Documentation

And here are some not so good reasons for migrating:

- Thinking it's Easier than JavaScript
- Asumming that Type Correctness === Program Correctness
- Everybody's doing it

## Overview of the Process

The steps below will take you through the whole process concisely. There's also a [guide](https://github.com/ZainAmjad68/js-to-ts-migration-guide#common-errors-and-their-fixes) at the end for fixing common errors once you start converting TS files to JS.


- Make two folders, `src` and `build`. Copy/Paste all the old project files into the src folder. Take out the `package.json` and any other non-JS configuration related files (.github folder, .gitignore, claudia webpack, gulp files etc.) into the root directory
- Install TypeScript by executing `npm install typescript`. And use `tsc --init` command to build a tsconfig.json file.
    * `tsconfig.json` should have the input and output directories defined, allow JS to be processed initially, and include the target version of JS. All in all, these options should be present in the file:
    ```json
    {
    "compilerOptions": {
        "outDir": "./build",
        "allowJs": true,
        "target": "es5"
    },
    "include": ["./src/**/*"]
    }
    ```
- At this point, running `tsc` would look for any files with `.ts` or `.js` (because of allowJs) extension and transpile them into the build folder. But it doesn't process other files like html, ejs or css in our project.
- There are two ways to handle what happens with non-JS files that aren't transpiled by the TS compiler itself:
    * using build tools like webpack or gulp to handle these files for you
        - There's an official [guide](https://www.typescriptlang.org/docs/handbook/migrating-from-javascript.html#integrating-with-build-tools) for this that you can refer to.
    * build re-usable custom logic for projects not integrating build tools
        - This method use libraries [ts-node](https://www.npmjs.com/package/ts-node), [shelljs](https://www.npmjs.com/package/shelljs), [nodemon](https://www.npmjs.com/package/nodemon), [rimraf](https://www.npmjs.com/package/rimraf), and [npm-run-all](https://www.npmjs.com/package/npm-run-all)
        - Install all the above mentioned packages and their type definitions
        - Make a folder named tools and a file named copy-assets.ts in it. Use the file to copy assets (views folder etc.) from src to build using the shelljs module.
            ```javascript
            import * as shell from "shelljs";

            // Copy all the view templates
            shell.cp("-R", "src/views", "build/");

            // add commands for any other stuff to copy over
            ```
        - Add these scripts to `package.json`:
            ```javascript
            "scripts": {    // remove comments if you copy this as json doesn't support comments
                "clean": "rimraf build/*",  // deletes everything from build
                "tsc": "tsc",   // transpiles typescript to javascript
                "copy-assets": "ts-node tools/copyAssets",  // copies relevant assets from src to build
                "build": "npm-run-all clean tsc copy-assets",   // runs all of the above mentioned scripts with a single command
                "start": "node .",  // command to run dev server
                "dev:start": "npm-run-all build start",    // transpiles and then starts the dev server
                "dev": "nodemon --watch src -e ts,ejs,png,css --exec npm run dev:start",    // starts the dev server in watch mode
            }
            ```
- In the package.json, change the main entering point of the app and also specify the files property to only include build folder files.
```json
  "main": "build/index.js",
  "files": [
    "build/**/*.*",
    "package.json"
  ]
```

---------------------------------------------------------------------------------------------------------------------------------

***You are now ready to convert your first JS file into TS.*** 

Choose any file (try to start with smaller files and go in the same flow as your application would go [i.e.; for an express application: index.js -> router -> middleware -> handler]) that you would like to convert and change its extension to `.ts`. Deal with any error that might arise - the guide below sheds some light on some common errors that you might encounter. And after you're satisfied, run `npm run build`. It should transpile the typescript code and give you the desired result.

Repeat the process until all the files in the project are converted to TS. If possible, run the project after every conversion, it would make it easier to catch any errors made by you during the converion and will help to isolate the cause of failure to that single recently converted file. 

---------------------------------------------------------------------------------------------------------------------------------

## Common Errors and their Fixes:

Below are some errors that one usually has to face when converting JavaScript code to TypeScript.

-> Error #1: On trying to transpile using `tsc` command, you get **`No inputs were found in config file`**
> Error means that TS couldn't find any file to compile. Create an empty ts file in src folder or convert any existing js one and the error should go away.

-> Error #2: **`Could not find a declaration file for module 'module-name'.`**
> Error means that the type annotations for that module are missing, so we can fix this easily by executing `npm install -D @types/module-name`

-> Error #3: **`File is not a module`**
> The way typescript exports/imports files is different to javascript. For example, you import a module using `const xyz = require("xyz");`, the same can be done in typescript in following ways: ```javascript import * as app1 from "./test"; import app2 = require("./test"); import {App} from "./test";```. Similarly, for exports you convert `module.exports = foo` to `export = foo`. More details [here](https://stackoverflow.com/a/32805764)

-> Error #4: **`Element implicitly has an 'any' type`**
> Probably the most common error you'll encounter when you convert a JS file to TS. It means that typescript doesn't know the type of the 'Element' and has assigned the generic `any` type to it and thus can't provide you with sort of intellisense or error reporting. You can solve this problem by giving the Element a type, whether it be a primitive type like `string` or a custom defined `interface`

-> Error #5: **`Object is possibly 'undefined'`**
> You might encounter this if you're trying to access some nested property of an object. The error says that both you and typescript can't possibly know if that property will exist at runtime and so you need to handle the case where it doesn't exist. There's a pretty simple fix for this: just add '?' to each property that might be undefined and TS will automatically handle and assign undefined in case it doesn't exist. Example: `const data = change?.in?.data();`

-> Error #6: **`Typescript Error: Property 'abc' does not exist on type 'XYZ'`**
> This means that "abc" is not a property that is available natively in the XYZ object of the library you're utilizing. If you're sure that something (like a middleware) adds such property to the XYZ object, you can override the XYZ type this way:
```javascript
declare module "module-name" { 
  export interface XYZ {
    abc: string,
  }
}
```

-> Error #7: **`Object is of type 'unknown'.`**
> Typescript assigns unknown to the err in the catch block of a try/catch because it doesn't know what type of error might be thrown. As the developer, you have to help typescript out a little bit here and tell it the error type so it can do its job. For example, if you're making an API call, you're likely to get either a StatusCodeError or a RequestError. So, you can handle the error like this:
```javascript
try {
    perform some task
} catch(err) {
     if (err instanceof StatusCodeError) 
     { 
         handle server/auth error   // typescript is intelligent enough to only show you properties that are available on a Status Code Error here
     } else if (err instanceof RequestError) 
     { 
         handle bad request error   // can access all Request Error properties here
     }
}
```

-> Error #8: **`'Foo' only refers to a type, but is being used as a value here.`**
> If a variable can have different types of value depending on a scenario, you might want to use 'instanceof' to handle each of the scenarios. This is where the above error can be usually seen. You might need to use Type Guards to solve this. More details [here](https://stackoverflow.com/a/46703380).

-> Error #9: **`No overload matches this call.`**
> Happens when you have passed an incorrect number of arguments to an anonymous function, for example you might see this error when adding middleware in express. The way to go about this one is to use type assertion (as) to tell typescript what you're trying to do. e.g.; `Router.get("/work", module.doSomeWork as unknown as express.RequestHandler);`