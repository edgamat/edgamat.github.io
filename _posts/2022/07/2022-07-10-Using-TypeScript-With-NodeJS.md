---
layout: post
title: 'Using typeScript With NodeJS'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I recently started a new project using NodeJS. The code uses several NodeJS applications with TypeScript. I wanted to document the steps to set up a new NodeJS application for future reference.

<!--more-->

### NodeJS

First, initial a folder as a NodeJS application:

```bash
mkdir myapp
cd myapp
npm init -y
```

### TypeScript

Now install the typescript dependencies:

```bash
npm install typescript ts-node @types/node --save-dev
npx tsc --init
```

Add this to the `scripts` section in the `package.json` file:

```json
"start": "ts-node ./src/app.ts",
```

Now you can use `npm start` to build and run the application locally.

Create the entry point for your application. Add a folder called `src` and add an `app.ts` file:

```javascript
console.log("Hello, World!");
```

To configure how the TypeScript output is generated, modify the `tsconfig.json` file. 

For example, if the output JS files need to be in separate folder, set the `outDir` property:

```json
"outDir": "./build",
```

### What's Next

This setup is a generic one, regardless of the type of application you are creating. If you are creating a command-line application, a REST API or a cloud function, you can use this base setup for all application types.

You can also use formatting (prettier), linting (ES Lint) and unit testing (Mocha/Chai) with TypeScript which I will explore in other posts.
