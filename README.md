# js-to-ts-migration-guide

If you've reached here, you might already know that TypeScript is a superset of JavaScript and that TypeScript is eventually transpiled to JavaScript so that it can be run. That begs the question, **why would anyone want to migrate to TS?**

## Why do it?
There are several reasons why one might want to convert his JavaScript project to TypeScript.

While the size, scope, and complexity of programs written in JavaScript has grown exponentially, the ability of the JavaScript language to express the relationships between different units of code has not. Combined with JavaScript’s rather peculiar runtime semantics, this mismatch between language and program complexity has made JavaScript development a difficult task to manage at scale.

It's always good to validate that you're doing something for the right reasons, because 

Here are some of the good reasons for deciding to migrate to TypeScript:

- Taking advantage of its Type Annotations
- Language Features provided on top of JavaScript
- Intellisense | API Documentation

And here are some not so good reasons for migrating:

- Thinking it's Easier than JavaScript
- Type Correctness === Program Correctness
- Everybody's doing it

## Overview of the Process

All the changes documented:
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
                "build": "npm-run-all clean tsc copy-assets",   // runs the above mentioned scripts with a single command
                "start": "node .",  // command to run dev server
                "dev:start": "npm-run-all build start",    // transpiles and then starts the dev server
                "dev": "nodemon --watch src -e ts,ejs,png,css --exec npm run dev:start",    // starts the dev server in watch mode
            }
            ```
- In the package.json, change the main entering point of the app and also specify the files property to only include build folder files.
```json
  "main": "build/app.js",
  "files": [
    "build/**/*.*",
    "package.json"
  ]
```

*You are now ready to convert your first JS file into TS.* Choose any file (try to start with smaller files and go in the same flow as your application would go [i.e.; for an express application: index.js -> router -> middleware -> handler]) that you would like to convert and change its extension to `.ts`. Deal with any error that might arise. And after you're satisfied, run `npm run build`. It should transpile the typescript code and give you the desired result.

Repeat the process until all the files in the project are converted to TS. If possible, run the project after every conversion, it would make it easier to catch any errors made by you during the converion and will help to isolate the cause of failure to that single recently converted file. 

