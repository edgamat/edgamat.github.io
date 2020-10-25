---
layout: post
title: 'Loading JavaScript in Angular'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

The project I have been working on integrates with an external API. It requires a JavaScript reference to be added to the `index.html` file (i.e. a `<script>` element). The URL to this script changes, depending on the environment the application uses (local, dev, test, prod). I wanted to explore how to make this reference dynamically loaded via the `environment` configuration file. 

<!--more-->

Here's how the `index.html` file looks now:

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Mark1</title>
  <base href="/">
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
  <script src="https://example-script-cdn.com/my-script.js"></script>
  <app-root></app-root>
</body>
</html>
```

What I need to be able to do is add this URL to the `environment.ts` and `environment.prod.ts` files like this:

```typescript
export const environment = {
  production: false,
  cdnUrl: 'https://example-script-cdn.com/my-script.js'
};
```

I found a great article describing how to accomplish this:  
[http://www.lukasjakob.com/how-to-dynamically-load-external-scripts-in-angular/](http://www.lukasjakob.com/how-to-dynamically-load-external-scripts-in-angular/)

So I decided to give it a try.

### The Script Service

I added a new service to my project using the Angular CLI:

```powershell
ng g s core\script --skip-tests
```

I created a list of scripts I wanted to be load-able via the service. I could add more if I wished, just adding a new key/value pair giving each script a unique key.

```typescript
import { environment } from '../../environments/environment';

const loadableScripts = [
  { name: 'cdn', src: environment.cdnUrl }
];
```

Then modified the service to track each script's loaded status:

```typescript
  private scripts: any = {};

  constructor() {
    loadableScripts.forEach((script: any) => {
      this.scripts[script.name] = {
        loaded: false,
        src: script.src
      };
    });
  }
```

And then added the `loadScript` method. I didn't require the 'load' method so I left it out. Also, I didn't require support for IE, so I just implement the non-IE approach using the `onload` event:

```typescript
  // load the script
  loadScript(name: string): Promise<any>  {
    return new Promise((resolve, reject) => {
      // resolve if already loaded
      if (this.scripts[name].loaded) {
        resolve({script: name, loaded: true, status: 'Already Loaded'});
      } else {
        // load script
        const script = document.createElement('script');
        script.type = 'text/javascript';
        script.src = this.scripts[name].src;

        script.onload = () => {
          this.scripts[name].loaded = true;
          resolve({script: name, loaded: true, status: 'Loaded'});
        };

        script.onerror = (error: any) => resolve({script: name, loaded: false, status: 'Loaded'});

        // finally append the script tag in the DOM
        document.getElementsByTagName('head')[0].appendChild(script);
      }
    });
```

**NOTE** The IE support presented in the blog post didn't seem to compile in Angular 10. It is possible it is only a TypeScript type issue and using `any` might correct the issue. Not sure, but if you are going to use it with IE, you are warned it may need some additional attention.

### Using the Script Service

I injected the new script service into the component that required the CDN URL script. I set a boolean flag to let me know if the script was loaded.

```typescript
  isCdnLoaded: boolean;

  constructor(private scriptService: ScriptService) {
    this.isCdnLoaded = false;
  }
```

Then in the `OnInit` handler, I loaded my script:

```typescript
    this.scriptService.loadScript('cdn').then(() => {
      this.isCdnLoaded = true;
    });
```

### The Results

When I loaded the application, I discovered that the script was not loaded. Or at least that is what I initially thought. But some quick `console.log` statements told me that the script was loading just fine. The issue was with timing. The script was being dynamically added to the DOM _AFTER_ the template needed the script's contents. So it ended up giving me a JavaScript error as if the script wasn't loaded.

To work around this, I added an `*ngIf` to the element in the component that required the script:

```html
<div *ngIf="isCdnLoaded">
...
</div>
```

This allowed the script time to load and once complete, would render the element(s) that require the functions within the script.

**NOTE** If I so desired I could swap out the use of Promises with RxJs Observables. I might do that in the future, but the use of promises for this sort of thing seems to fit quite nicely into my existing application.

### Summary

I was glad to find someone willing to post a solution to the same sort of problem I was faced with. While it wasn't a perfect match for my use case, it gave me enough to run with and get working in my context. Thanks Lukas!
