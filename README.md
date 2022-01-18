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
- Make two folders, `src` and `build`. Copy/Paste all the old project files into the src folder. Take out the `package.json` and any other non-JS configuration related files (.github folder, .gitignore, any claudia json files)
- Install TypeScript by executing `npm install typescript`. And use `tsc --init` command to build a tsconfig.json file.

`tsconfig.json` should have the input and output directories defined, allow JS to be processed initially, and include the target version of JS. All in all, these options should be present in the file:
```json
{
  "compilerOptions": {
    "outDir": "./built",
    "allowJs": true,
    "target": "es5"
  },
  "include": ["./src/**/*"]
}
```
- At this point, running `tsc` would look for any files with `.ts` or `.js` (because of allowJs) extension and transpile them into the build folder. But it doesn't process other files like html, ejs or css in our project.
- There are two ways to handle what happens with non-JS files that aren't transpiled by the TS compiler itself:
* using build tools like webpack or gulp to handle these files for you
* build re-usable custom logic for projects not integrating build tools

-> install these packages<\br>
`npm install --save-dev ts-node@10 shelljs@0.8 fs-extra@10 nodemon@2 rimraf@3 npm-run-all@4`<\br>
`npm install --save-dev @types/fs-extra@9 @types/shelljs@0.8`

-> Make a folder named tools and a file named copy-assets.ts in it. Use the file to copy assets,views from src to build using the shelljs module.

-> have these scripts in package.json
```json
"scripts": {
    "clean": "rimraf dist/*",
    "copy-assets": "ts-node tools/copyAssets",
    "lint": "tslint -c tslint.json -p tsconfig.json --fix",
    "tsc": "tsc",
    "build": "npm-run-all clean lint tsc copy-assets",
    "dev:start": "npm-run-all build start",
    "dev": "nodemon --watch src -e ts,ejs --exec npm run dev:start",
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
- We need to find a way to handle non-JS (front-end [EJS, CSS] etc.) files, i.e.; copy them from src to build when transpiling. To do that, execute the following commands:
npm install --save-dev ts-node@10 shelljs@0.8 fs-extra@10 nodemon@2 rimraf@3 npm-run-all@4
npm install --save-dev @types/fs-extra@9 @types/shelljs@0.8
- Make a folder named tools and a file named copy-assets.ts in it. Use the file to copy assets,views from src to build using the shelljs module.
- Once installed, paste the following scripts into package.json:
"scripts": {
    "clean": "rimraf dist/*",
    "copy-assets": "ts-node tools/copyAssets",
    "lint": "tslint -c tslint.json -p tsconfig.json --fix",
    "tsc": "tsc",
    "build": "npm-run-all clean lint tsc copy-assets",
    "dev:start": "npm-run-all build start",
    "dev": "nodemon --watch src -e ts,ejs --exec npm run dev:start",
  }
