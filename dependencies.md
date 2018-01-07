# Dependencies

As described in the [README](README.md), most published NPM packages are transpiled to an old ES version (ES3 or ES5) to ensure maximum compatibility. Some also even include polyfills.

This comes from various reasons. As a JS library maintainer, you want to ensure that:
- As many people can use your code
- Even people not using a modern bundler
- People including your package in a `<script>` tag (using unpkg.com for example)
- Users does not have a complex setup to do besides importing your code (using `require()` or `import`)

But this forced transpilation is not good for developers desiring to ship modern JS to their customers. Here is a proposal to improve this.

## Requirements

- All existing packages needs to keep working as before
- Packages maintainers should not have to setup a complex configuration

## Proposal

- Make it easy for maintainers to publish multiple versions of their lib, transpiled to various targets (ES5, ES6, ES7, …)
- Define a way to point to multiple entry points in `package.json`, each with the minimal set of features needed to support it and pointing to the targets described in the previous point
  - Should it be based on ECMAScript versions?
  - Should it include other hints, like API used?
- Patch the build tools to understand the new `package.json` options and include the "best" version of each lib available
- If a dependency's package is using this new configuration, this package's source should be transpiled

Things to keep in mind:
- It seems a bad idea to make build tools import the original source in any cases, as the build tool might not understand it if the dependency is using a veyr modern syntax. For example, if a package is written using experimental ES7 features, it will not be transpiled by Babel. Hence the need for the package to be transpiled to a known and common target (or multiple ones).
- There have been 2 recent new entries in `package.json`: `jsnext:main` and `module`. They may seem useful to our case at first, but in practice they are not. Like what is described above, they point to an alternate transpilation included in the distributed package, but the only requirement for this version is to be using `import` statements and no `require` or older way of loading files. Most of the time, this alternate transpiled version is the same one as the main entry, except for those `import` statements. It has been created to help optimising published bundles (tree-shaking, …).

## TODO

- Security implications. For example, not including transpiled/minified code makes it easier to audit dependencies and ensure the included code is what we want
