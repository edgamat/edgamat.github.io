---
layout: post
title: 'Setup of a NodeJS Console Application with TypeScript'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I recently wrote a simple NodeJS console application using TypeScript. This is how I set it up.
 
<!--more-->

**Create a new folder for the project:**

```powershell
mkdir ts-sample
cd ts-sample
```

**Initialize the NodeJS application:**

```powershell
npm init -y
```

The resulting `package.json` file contains the following:

```json
{
  "name": "ts-sample",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

I manually change the main entry point from `index.js` to `app.js`.

**Add TypeScript Support:**

```powershell
npm i -D typescript # Typescript compiles to plain JS  
npm i -D ts-node # ts-node to run typescript code without compiling to JS  
```

The two packages were added as development dependencies:

```json
  "devDependencies": {
    "ts-node": "^10.4.0",
    "typescript": "^4.5.4"
  }
```

**Configure TypeScript**

I manually added the `tsconfig.json` file (`wsl touch tsconfig.json`) with the following contents:

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "es6",
    "lib": ["es6"],
    "outDir": "dist",
    "strict": true,
    "sourceMap": true
  },
  "exclude": [
    "node_modules"
  ] 
}
```

**Setup NPM Scripts**

I replaced the `scripts` section in the `package.json` file with the following:

```json
  "scripts": {
    "start": "ts-node ./app.ts",
    "build": "tsc --build",
    "clean": "tsc --build --clean"
  },
```

**Hello, World!**

I added an `app.ts` TypeScript file to test that everything was setup correctly:

```typescript
console.log('Hello, World!')

const args = process.argv

args.forEach((val, index) => {
  console.log(`${index}: ${val}`)
})
```

To run the application, I used the following command:

```powershell
npm start
```
 
**Setup Git**

The last thing I did was set up Git. First, I added a `.gitignore` file:

```powershell
# Create a .gitignore file
$url = "https://raw.githubusercontent.com/github/gitignore/master/Node.gitignore"
$output = ".gitignore"
Invoke-WebRequest -Uri $url -OutFile $output
```

Then initialized Git:

```powershell
git init
git add .
```

You should see only these files being added:

```powershell
git status
```

```
new file:   .gitignore
new file:   app.ts
new file:   package-lock.json
new file:   package.json
new file:   tsconfig.json
```

If all looks good, then commit the initial set of changes:

```powershell
git commit -m "initial commit"
```
