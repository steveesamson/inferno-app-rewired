# inferno-app-rewired

Tweak the create-inferno-app webpack config(s) without using 'eject' and without creating a fork of the inferno-scripts.

All the benefits of create-inferno-app without the limitations of "no config". You can add plugins, loaders whatever you need.

> All you have to do is create your app using create-inferno-app and then rewire it.


 
[![npm version](https://img.shields.io/npm/v/infeno-app-rewired.svg)](https://www.npmjs.com/package/inferno-app-rewired)
[![npm monthly downloads](https://img.shields.io/npm/dm/inferno-app-rewired.svg)](https://www.npmjs.com/package/inferno-app-rewired)
 

# Rewire Your App

Tweak the create-inferno-app webpack config(s) without using 'eject' and without creating a fork of the inferno-scripts.

All the benefits of create-inferno-app without the limitations of "no config". You can add plugins, loaders whatever you need.

> All you have to do is create your app using [create-inferno-app](https://github.com/infernojs/create-inferno-app) and then rewire it.

⚠️ **Please Note:**

> By doing this you're breaking the ["guarantees"] that CIA provides. That is to say you now "own" the configs. **No support** will be provided. Proceed with caution.

# How to rewire your create-inferno-app project


#### 1) Install inferno-app-rewired
```bash
$ npm install inferno-app-rewired --save-dev
```

#### 2) Create a `config-overrides.js` file in the root directory

```javascript
/* config-overrides.js */

module.exports = function override(config, env) {
  //do stuff with the webpack config...
  return config;
}
```

```
+-- your-project
|   +-- config-overrides.js
|   +-- node_modules
|   +-- package.json
|   +-- public
|   +-- README.md
|   +-- src
```

**Note:** You can use one of the default rewires (see the [packages](/packages) dir) or [injectBabelPlugin](https://github.com/steveesamson/inferno-app-rewired#utilities-injectbabelplugin)

#### 3) 'Flip' the existing calls to `inferno-scripts` in `npm` scripts
```diff
  /* package.json */

  "scripts": {
-   "start": "inferno-scripts start",
+   "start": "inferno-app-rewired start",
-   "build": "inferno-scripts build",
+   "build": "inferno-app-rewired build",
-   "test": "inferno-scripts test --env=jsdom",
+   "test": "inferno-app-rewired test --env=jsdom"
}
```

#### 4) Start the Dev Server
```bash
$ npm start
```


#### 5) Build your app
```bash
$ npm run build
```


## Utilities 

#### 1) injectBabelPlugin

Adding a Babel plugin can be done via the `injectBabelPlugin(pluginName, config)` function.  You can also use the "rewire" packages from this repo or listed below to do common config modifications.

```javascript
const rewireMobX = require('inferno-app-rewire-mobx');
const rewirePreact = require('inferno-app-rewire-preact');
const {injectBabelPlugin} = require('inferno-app-rewired');

/* config-overrides.js */
module.exports = function override(config, env) {
  // add a plugin
  config = injectBabelPlugin('emotion/babel',config)
  
  // use the Preact rewire
  if (env === "production") {
    console.log("⚡ Production build with Preact");
    config = rewirePreact(config, env);
  }
  
  // use the MobX rewire
  config = rewireMobX(config,env);
  
  return config;
}
```

#### 2) compose(after v1.3.4)

You can use this util to compose rewires.
> A functional programming utility, performs `right-to-left` function composition.     
More detail you can see [ramda](http://ramdajs.com/docs/#compose) or [redux](http://redux.js.org/docs/api/compose.html#composefunctions)  

Before:
```javascript
/* config-overrides.js */
module.exports = function override(config, env) {
  config = rewireLess(config, env);
  config = rewirePreact(config, env);
  config = rewireMobX(config, env);
  
  return config;
}
```
After use `compose`:
```javascript
/* config-overrides.js */
const { compose } = require('inferno-app-rewired');

module.exports = compose(
  rewireLess,
  rewirePreact,
  rewireMobx
  ...
)
//  custom config 
module.exports = function(config, env){
  const rewires = compose(
    rewireLess,
    rewirePreact,
    rewireMobx
    ...
  );
  // do custom config
  // ...
  return rewires(config, env);
}
```
Some change with rewire, if you want to add some `extra param` for `rewire`  
1. Optional params:  
you can see [inferno-app-rewire-less](https://github.com/steveesamson/inferno-app-rewire-less)  

2. Required params:  
```javascript
// rewireSome.js
function createRewire(requiredParams){
  return function rewire(config, env){
    ///
    return config
  }
}
module.exports = createRewire;
```

## Extended Configuration Options
By default, the `override-config.js` file exports a single function to use when customising the webpack configuration for compiling your inferno app in development or production mode. It is possible to instead export an object from this file that contains up to three fields, each of which is a function. This alternative form allows you to also customise the configuration used for Jest (in testing), and for the Webpack Dev Server itself.

This example implementation is used to demonstrate using each of the object require functions. In the example, the functions:
* use the inferno-app-rewire-less package to add less support to your project
* have some tests run conditionally based on `.env` variables
* set the https certificates to use for the Development Server, with the filenames specified in `.env` file variables.
```javascript
module.exports = {
  // The Webpack config to use when compiling your inferno app for development or production.
  webpack: function(config, env) {
    // ...add your webpack config customisation, rewires, etc...
    // Example: add less support to your app.
    const rewireLess = require('inferno-app-rewire-less');
    config = rewireLess(config, env);

    return config;
  },
  // The Jest config to use when running your jest tests - note that the normal rewires do not
  // work here.
  jest: function(config) {
    // ...add your jest config customisation...
    // Example: enable/disable some tests based on environment variables in the .env file.
    if (!config.testPathIgnorePatterns) {
      config.testPathIgnorePatterns = [];
    }
    if (!process.env.RUN_COMPONENT_TESTS) {
      config.testPathIgnorePatterns.push('<rootDir>/src/components/**/*.test.js');
    }
    
    return config;
  },
  // The function to use to create a webpack dev server configuration when running the development
  // server with 'npm run start' or 'yarn start'.
  // Example: set the dev server to use a specific certificate in https.
  devServer: function(configFunction) {
    // Return the replacement function for create-inferno-app to use to generate the Webpack
    // Development Server config. "configFunction" is the function that would normally have
    // been used to generate the Webpack Development server config - you can use it to create
    // a starting configuration to then modify instead of having to create a config from scratch.
    return function(proxy, allowedHost) {
      // Create the default config by calling configFunction with the proxy/allowedHost parameters
      const config = configFunction(proxy, allowedHost);

      // Change the https certificate options to match your certificate, using the .env file to
      // set the file paths & passphrase.
      const fs = require('fs');
      config.https = {
        key: fs.readFileSync(process.env.REACT_HTTPS_KEY, 'utf8'),
        cert: fs.readFileSync(process.env.REACT_HTTPS_CERT, 'utf8'),
        ca: fs.readFileSync(process.env.REACT_HTTPS_CA, 'utf8'),
        passphrase: process.env.REACT_HTTPS_PASS
      };

      // Return your customised Webpack Development Server config.
      return config;
    }
  }
}
```

#### 1) Webpack configuration - Development & Production
The `webpack` field is used to provide the equivalent to the single-function exported from config-overrides.js. This is where all the usual rewires are used. It is not able to configure compilation in test mode because test mode does not get run through Webpack at all (it runs in Jest). It is also not able to be used to customise the Webpack Dev Server that is used to serve pages in development mode because create-inferno-app generates a separate Webpack configuration for use with the dev server using different functions and defaults.

#### 2) Jest configuration - Testing
Webpack is not used for compiling your application in Test mode - Jest is used instead. This means that any rewires specified in your webpack config customisation function _will not be applied_ to your project in test mode.

React-app-rewired automatically allows you to customise your Jest configuration in a `jest` section of your `package.json` file, including allowing you to set configuration fields that create-inferno-app would usually block you from being able to set. It also automatically sets up Jest to compile the project with Babel prior to running tests. Jest's configuration options are documented separately at the [Jest website](https://facebook.github.io/jest/docs/en/configuration.html).

If you want to add plugins and/or presets to the Babel configuration that Jest will use, you need to define those plugins/presets in either a `babel` section inside the `package.json` file or inside a `.babelrc` file. React-app-rewired alters the Jest configuration to use these definition files for specifying Babel options when Jest is compiling your inferno app. The format to use in the Babel section of package.json or the .babelrc file is documented separately at the [Babel website](https://babeljs.io/docs/usage/babelrc/).

The `jest` field in the module.exports object in `config-overrides.js` is used to specify a function that can be called to customise the Jest testing configuration in ways that are not possible in the jest section of the package.json file. For example, it will allow you to change some configuration options based on environment variables. This function is passed the default create-inferno-app Jest configuration as a parameter and is required to return the modified Jest configuration that you want to use. A lot of the time you'll be able to make the configuration changes needed simply by using a combination of the `package.json` file's jest section and a `.babelrc` file (or babel section in package.json) instead of needing to provide this jest function in `config-overrides.js`.

#### 3) Webpack Dev Server
When running in development mode, create-inferno-app does not use the usual Webpack config for the Development Server (the one that serves the app pages). This means that you cannot use the normal `webpack` section of the `config-overrides.js` server to make changes to the Development Server settings as those changes won't be applied.

Instead of this, create-inferno-app expects to be able to call a function to generate the webpack dev server when needed. This function is provided with parameters for the proxy and allowedHost settings to be used in the webpack dev server (create-inferno-app retrieves the values for those parameters from your package.json file).

React-app-rewired provides the ability to override this function through use of the `devServer` field in the module.exports object in `config-overrides.js`. It provides the devServer function a single parameter containing the default create-inferno-app function that is normally used to generate the dev server config (it cannot provide a generated version of the configuration because inferno-scripts is calling the generation function directly). React-app-rewired needs to receive as a return value a _replacement function_ for create-inferno-app to then use to generate the Development Server configuration (i.e. the return value should be a new function that takes the two parameters for proxy and allowedHost and itself returns a Webpack Development Server configuration). The original inferno-scripts function is passed into the `config-overrides.js` devServer function so that you are able to easily call this yourself to generate your initial devServer configuration based on what the defaults used by create-inferno-app are.


React-app-rewired requires a custom inferno-scripts package to provide the following files:
* config/env.js
* config/webpack.config.dev.js
* config/webpack.config.prod.js
* config/webpackDevServer.config.js
* scripts/build.js
* scripts/start.js
* scripts/test.js
* scripts/utils/createJestConfig.js

#### 3) Specify config-overrides as a directory
Inferno-app-rewired imports your config-overrides.js file without the '.js' extension. This means that you have the option of creating a directory called `config-overrides` at the root of your project and exporting your overrides from the default `index.js` file inside that directory.

If you have several custom overrides using a directory allows you to be able to put each override in a separate file. An example template that demonstrates this can be found in [Guria/rewired-ts-boilerplate](https://github.com/Guria/rewired-ts-boilerplate/tree/master/config-overrides) at Github.

#### 4) Specify config-overrides location from command line
If you need to change the location of your config-overrides.js you can pass a command line option --config-overrides <path> to the inferno-app-rewired script.


# Why This Project Exists

See: [Create React App — But I don’t wanna Eject.](https://medium.com/@timarney/but-i-dont-wanna-eject-3e3da5826e39#.x81bb4kji)



