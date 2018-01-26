# Shipping modern Javascript to browsers

## Why ?

ECMAScript/Javascript has been evolving quickly in the last years, with a lot of modern language construct appearing in newer versions. Most of those features are now implemented by the latest versions of web browsers, and each new version brings support for more.
But when developing a website, you usually want to ensure compatibility with a wide range of browsers, so your code needs to run on the older browser versions you want to support. Usually, this means using ECMAScript 5 (ES5).
Those old browsers market share is slowly diminishing but will most probably still represent a significant share of users in the coming years, especially in specific markets (big corps, some continents/countries, …).

Nowadays, more and more code is written using a modern ECMAScript implementation (ES6, ES7) to take advantage of the new syntax and features, and is converted (transpiled) to ES5 before being published so it can be ran on most browsers. Polyfills for new functions are also added to the published code. Those are JS implementation of functions that appear in newer versions, and are loaded at runtime if there is no native version available.

This document aims at defining and implementing various techniques to be able to send modern Javascript to browsers that support it. This have two main benefits:

1. Output is smaller (thus faster to load and parse), due to multiple factors:
    1. ES6/ES7 syntax offer shorter syntax for many common paterns, for example `const [a, b, c] = myArray` (ES7) compared to `var a = myArray[0], b = myArray[1], c = myArray[2]` (ES5)
    2. More importantly, when transpiling the output is most often bigger than what you would manually do, to take into account edge cases. For example, the previous example is transpiled by Babel as:

      ```javascript
      function _sliceIterator(arr, i) { var _arr = []; var _n = true; var _d = false; var _e = undefined; try { for (var _i = arr[Symbol.iterator](), _s; !(_n = (_s = _i.next()).done); _n = true) { _arr.push(_s.value); if (i && _arr.length === i) break; } } catch (err) { _d = true; _e = err; } finally { try { if (!_n && _i["return"] != null) _i["return"](); } finally { if (_d) throw _e; } } return _arr; }

      function _slicedToArray(arr, i) { if (Array.isArray(arr)) { return arr; } else if (Symbol.iterator in Object(arr)) { return _sliceIterator(arr, i); } else { throw new TypeError("Invalid attempt to destructure non-iterable instance"); } }

      var _myArray = myArray,
      _myArray2 = _slicedToArray(_myArray, 3),
      a = _myArray2[0],
      b = _myArray2[1],
      c = _myArray2[2];
      ```

    3. Finally, polyfills add unused code (as the browser natively support those new features), but it still needs to be loaded/parsed by the browser
2. Modern JS code is faster

  Using the newer syntax and functions allows the Javascript engine to optimize our code better than when processing transpiled code, leading to faster execution, lower memory usage and lower CPU utilisation


## How ?

The overall goal is to allow developers to write their frontend Javascript code using the syntax they prefer (ES6, ES7, Typescript, …) and then easily get multiple versions (ES5, ES6, …) they can deliver to their users.

This can drill down to two steps, detailed below.

### Make browsers load Javascript files as modern as possible, but that they can handle

There is no standard way to do this.

A working method is explained by Philip Walton on his blog: [Deploying ES2015+ Code in Production Today](https://philipwalton.com/articles/deploying-es2015-code-in-production-today/)

In summary, it relies on the fact that recent browsers support `<script type="module" src="modern-es6-version.js" />` and `<script nomodule src="legacy-es5-version.js" />` tags.

It allows to publish an ES6 (aka ES2015) version that will be loaded by those modern browsers, and an ES5 version loaded by all other browsers. The main drawback of this method is the lack of evolutivity in the future. As it relies on an indirect way to conditionally load a different file depending on modules support, we will not be able to use the same method to, for example, load an ES8 file on browsers supporting the ES8 specification once it is released.

Some ideas on how to better solve this problem are described in [browser-side.md](browser-side.md).

### Generate multiple transpiled versions of the source code

Once we have a way to tell the browser the load the "best" (most modern) available code amongst multiple files, we need to have a way to generate those files from our source code.

#### Application code

This is the easiest part, as existing and wildly used tools already allow you to specify how compatible your output code needs to be.

If you are using Babel, `babel-preset-env` dynamically selects what to transpile (and which core syntax polyfills to include) depending on a list of browsers you want to support. For example, if we use the `<script module>` trick described above, we can configure `babel-preset-env` with the following list of browsers, and it will transpile our source code to the best possible version that runs on them:
```
Chrome >= 60, Safari >= 11, iOS >= 11, Firefox >= 54, Edge >= 15
```

If you are using Typescript, you can [configure the compiler](https://www.typescriptlang.org/docs/handbook/compiler-options.html) with `target: "ES2015"` and it will output what you expect.

Other build tools have similar options.

#### Dependencies

You now have 2 versions (ES5 and ES6) of your source code ready to be served to your clients, this is great! But if you use external dependencies inspect your ES6 version, you will see that your dependencies are most probably still using the ES5 version. This is because today most (> 99%) of the packages published on NPM are transpiled to ES5 and sometimes minified. This comes from various reasons like a lack of tooling, documentation, or the will to publish code that will run anywhere.

Because of this, most people are configuring their dependencies to be ignored by their transpiler (for example `exclude: /node_modules/` in Webpack's Babel config) as they expect it to be already transpiled and want to avoid additional transpilation time.

A [proposal to fix this](dependencies.md) is described. in in progress.

## Current status

The current focus is to find out the various use cases and draft a proposal to move towards shipping modern JS in browsers.

## Contribute

This is a work in progress and everybody is welcome to contribute. Feel free to comment on this file, open issues to discuss specific points or PR to suggest changed and improvements to this document.
