# lazy-module-loader

Lazy Module Loader is a universal JavaScript module loader allowing to load modules with asynchronous resolution of their dependencies.

[![Build Status](https://travis-ci.org/aswitalski/lazy-module-loader.svg?branch=master)](https://travis-ci.org/aswitalski/lazy-module-loader)
[![npm version](https://img.shields.io/npm/v/lazy-module-loader.svg?style=flat)](https://www.npmjs.com/package/lazy-module-loader)

## Intention

Lazy Module Loader had been widely used in Opera Desktop browser as a dedicated solution for modularity
and bundling mechanism for Web UI code before the ES modules were fully supported.

## Usage

Once the Lazy Module Loader script is loaded, it defines a global `loader` instance, which can be used as follows:

```js
const module = await loader.require('my/module');
```

The requested module is loaded with all its required dependencies resolved.

## Path resolution

By default the resource path is caluclated with simple rules:

No extension - the JS file extension is appended:

- `my/module` to `my/module.js`

Directory-like path - points to the main module in the directory:

- `my/module/` to `my/module/main.js`

Custom extension - returns the unchanged path:

- `my/module/base.css` to `my/module/base.css`

## Module format

Modules utilize the CommonJS format, they can define the `async init()` method which is used to inject references to dependencies.

```js
let Sevice;

const Module = {

  async init() {
    Service = await loader.require('some/dependendency');
  }

  loadData() {
    return Service.loadSomeData();
  }
};

module.exports = Module;
```

## Lazy loading

Modules can also define optional dependencies (in a form of symbols) to other modules which are loaded at runtime, if needed.
This is particularly useful when no direct references are needed, for example:

```js
const WelcomePage = loader.symbol('pages/welcome/');
const ContactPage = loader.symbol('pages/contact/');

class Router extends Component {

  render() {
    switch (this.props.page) {
      case 'contact':
        return ContactPage;
      default:
        return WelcomePage;
    }
  }
}

module.exports = Router;
```

The dependencies don't have to be loaded until the router component decides to render the particular page.

## Preloading

In order to achieve the maximum performance and responsiveness at runtime, modules can be preloaded at startup, not to lazy-load them upon user actions.

```js

const module = await loader.preload('my/module');
```

Such call will load recursively all the required and optional dependencies.

## Bundling

Modules can also be bundled together with all their dependencies into a single file. What allows them to be loaded synchronously in a bunch.

```js
require('./bundler.js');

bundler.generate('root/module', 'My Library', {
  prefix: 'some/prefix/',
});
```

## Plugins

Loader can chain the interceptor plugins, to serve as a fallback to custom loaders.

```js
const plugin = {
  path(key) {
    const path = super.path(key);
    if (someCondition) {
      return `http://www.example.com/module/${path}`;
    }
    return path;
  },
};
loader.use(plugin);
```

## Registry

All the loaded modules are kept in a registry. It is global and can be accessed as follows:

```js
loader.registry
```

 The registry is a map allowing to access the module information and easily analyze the dependency graph.
 Each module can be checked for all its dependencies and clients referencing it.
