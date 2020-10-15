# Alias solution for rewired create-react-app

This is more than simple alias. This is also a multi-project `src`
directory. Currently, `create-react-app` (CRA) does not support more than one
`src` directory in the project. Monorepo, multi-repo and library projects with
examples require more than one directories like `src`.

This is merely an alias and multi-source solution for CRA
and this is not a replacement for multi-package management tools like
[Lerna](https://github.com/lerna/lerna).

This project requires the use of **[react-app-rewired](https://github.com/timarney/react-app-rewired)**,
which allows to overwrite the Webpack configuration
of CRA projects without ejecting them.

[![Npm package](https://img.shields.io/npm/v/react-app-rewire-alias.svg?style=flat)](https://npmjs.com/package/react-app-rewire-alias)
[![Npm downloads](https://img.shields.io/npm/dm/react-app-rewire-alias.svg?style=flat)](https://npmjs.com/package/react-app-rewire-alias)
[![Dependency Status](https://david-dm.org/oklas/react-app-rewire-alias.svg)](https://david-dm.org/oklas/react-app-rewire-alias)
[![Dependency Status](https://img.shields.io/github/stars/oklas/react-app-rewire-alias.svg?style=social&label=Star)](https://github.com/oklas/react-app-rewire-alias)
[![Dependency Status](https://img.shields.io/twitter/follow/oklaspec.svg?style=social&label=Follow)](https://twitter.com/oklaspec)

#### This allows:

* quality and secure exports from outside `src`
* absolute imports
* any `./directory` at root outside of `src` with Babel and CRA features

#### This is designed for:

* monorepo projects
* multi-repo projects
* library projects with examples

#### Advantages over other solutions:

 * provided fully functional aliases and allows the use of Babel, JSX, etc.
   outside of `src`

 * provided fully secure aliases and uses the same module scope plugin from
   the original create-react-app package for modules (instead of removing it),
   to minimize the probability of including unwanted code
   
#### Installation

```sh
yarn add --dev react-app-rewired react-app-rewire-alias
```

or

```sh
npm install --save-dev react-app-rewired react-app-rewire-alias
```

#### Usage

Modify **config-overrides.js** like this:

```js
const {alias} = require('react-app-rewire-alias')

module.exports = function override(config) {
  alias({
    example: 'example/src',
    '@library': 'library/src',
  })(config)

  return config
}
```

This is compatible with [customize-cra](https://github.com/arackaf/customize-cra),
just insert it into the override chain.

#### Using config paths from *jsconfig.json* or *tsconfig.json*

You can also configure your paths in your `jsconfig.json` or `tsconfig.json` like this:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "example/*": ["example/src/*"],
      "@library/*": ["library/src/*"]
    }
  }
}
```

To keep aliases in one place, the provided `configPaths()` function
loads the paths from `jsconfig.json` or `tsconfig.json` with slight adaptations.
Use it like this:

```js
const {alias, configPaths} = require('react-app-rewire-alias')

module.exports = function override(config) {
  alias(configPaths())(config)

  return config
}
```

The `tsconfig.json` is prioritized over the `jsconfig.json` in this scenario.

If you placed the paths in a custom file, use the function like so instead:

```js
const {alias, configPaths} = require('react-app-rewire-alias')

module.exports = function override(config) {
  alias({
    ...configPaths('tsconfig.paths.json')
  })(config)

  return config
}
```

#### Using react-app-rewired

Integrating `react-app-rewired` into your project is very simple
(see [its documentation](https://github.com/timarney/react-app-rewired#readme)):
Create `config-overrides.js` mentioned above in the project's root directory
(the same including the `package.json` and `src` directory)
and rewrite the `package.json` like this:

```diff
  "scripts": {
-   "start": "react-scripts start",
+   "start": "react-app-rewired start",
-   "build": "react-scripts build",
+   "build": "react-app-rewired build",
-   "test": "react-scripts test",
+   "test": "react-app-rewired test",
    "eject": "react-scripts eject"
}
```

That is all. Now you can continue to use `yarn` or `npm` start/build/test commands as usual.

#### Workaround for "aliased imports are not supported"

CRA [overwrites](/blob/v3.4.1/packages/react-scripts/scripts/utils/verifyTypeScriptSetup.js#L242)
your `tsconfig.json` at runtime and removes `paths` from the `tsconfig.json`,
which is not officially supported, with this message:

> ```
> The following changes are being made to your tsconfig.json file: 
>   - compilerOptions.paths must not be set (aliased imports are not supported)
> ```

The [suggested workaround](https://github.com/facebook/create-react-app/issues/5645#issuecomment-435201019)
is to move the paths to a different `.json` file, e.g. `tsconfig.paths.json`, like this:

```json
/* tsconfig.paths.json */
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "example/*": ["example/src/*"],
      "@library/*": ["library/src/*"]
    }
  }
}
```

with that file's subsequent inclusion in the `tsconfig.json` using `extends`:

```json
/* tsconfig.json */
{
  "extends": "./tsconfig.paths.json"
}
```

## Tips

* keep only one `node_modules` directory

When using this (or some other) integration solution to join mulitiproject system in one
application it is possible to meet strange unclear errors or problems with workning or
building and starting the application. There is no possible to define criteia about
what is the error - unpredictable strange errors. For example `<Route>` component from
`react-router` do not see that it under subtree of `<Router>` and it falls with:

> should not use Route or withRouter() outside a Router

This may be a result of some confusions in node_modules for multirepo projects. This
problem may appears also in plain `create-react-app` if one or more additional
`node_modulest` directory appers in `src`.

The simplest way to avoid problems is to keep only one `node_modules` folder for all
projects integrated together in one application. Otherwise some subproject can use different
versions of same library (for example React or Router etc) which can be not compatible with
each other in internal data structs (react contexts and so on) and that brings problems like
this. Therefore to avoid this problem it need to **manage correct versions in all
`node_modules`** or more simple just **keep only one `node_modules` directory**.
