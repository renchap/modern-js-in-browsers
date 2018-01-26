# How to make the browsers load the "best" JS file

In an ideal world, when a browser encounters a `<script>` tag, it should be able to load the "best" version of the script amongs all of those available. Those multiple versions are functionnaly equivalent but vary by the amount of transpilation and polyfills included.

A current and working method is explained by Philip Walton on his blog: [Deploying ES2015+ Code in Production Today](https://philipwalton.com/articles/deploying-es2015-code-in-production-today/)

This allows to deliver ES6 code to modern browsers by abusing their support for `<script type="module">`. It works fine for now as ES6 is the most recent ES version supported by major browsers but is limited to ES6 and can not be modified to support conditionnal loading of later ES versions.

A possible solution would probably involve an extension to the HTML specification and a way to declare in a `<script>` tag what features (APIs, ES version, …) a browser needs to support to load the specified file, and ignore it otherwise. A drawback of this approach would be that browsers might claim to support ES2015 (for example) but not implement the full spec, leading to loading JS files that will not be parsed. This also does not solve the support for features needing a polyfill, so polyfills will need to be loaded in every case (or have additional attributes listing the features that need to be supported).

Another idea would be to be able to specify multiple sources in a `<script>` tag (like `srcset`), each with a browserlist-like string describing the minimal browsers versions allowed to load this file.
