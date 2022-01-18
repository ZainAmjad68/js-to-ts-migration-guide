# js-to-ts-migration-guide

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
- Make two folders, src and build. Copy/Paste all the old project files into the src folder. Take out the package.json and any other non-JS config related files (.github folder, .gitignore, any claudia json files)
- Build a tsconfig.json file and add the appropriate options to it.
- In package.json, change the main entering point of the app and also specify the files property to only include build folder files.
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
