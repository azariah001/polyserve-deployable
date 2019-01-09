
# polyserve-deployable

A fork of the polyserve scripts with the singular purpose of making it possible to directly deploy Polymer apps without building client side code separate from your server side.

## Installation

    $ npm install polyserve-deployable
    
## Usage

### Import polyserve-deployable

    //* app.js
    const polyserveDeployable = require('polyserve-deployable');
    const path = require('path');
    const send = require("send");
    
    // see options listed below for more configuration options, all existing polyserve options are supported.
    const options = {
      moduleResolution: "node",
      npm: true,
      assetPaths: [ // minimum config follows: note assetPaths defaults to '/*' as per standard polyserve defaults if not defined.
        '/node_modules/*',
        '/src/*',
        '/images/*',
        '/manifest.json',
        '/service-worker.js',
        '/sw-precache-config.js'
      ]
    };
    
    polyserveDeployable.startServers(options).then((server) => {
      console.log("polyserve-deployable loaded and ready");
    
      const { app } = server;
      
      // initilise any other express middleware you wish to use here
    
      // serve your root app file: in this instance we're using the polymer-starter-kit as our example so this file is in the root directory and is a html file. You can serve this from anywhere in the project structure and utilise any server side rendering template engines you prefer using `app.render`
      app.get('/', (req, res) => {
        res.sendFile(path.join(__dirname + '/index.html'));
      });
      
      // include any other express routes here
    
    }).catch((e) => {
      process.exit(69);
    });
    
Note that under the standard polyserve library you can do this import, however, it always served '/*' making it possible to access virtually every file in the root which is not good.

So I've modified the `app.get('/*', ...)` declaration in the `getApp()` function in `start_server.ts` to accept an array of paths which still defaults to just `[ '/*' ]`. 
But now it can be overridden with any combination of paths with the only caveat being that it's a 1/1 between the file path from the project root and the express path;

So `'/node_modules/*'` entry in the array is located at `../node_modules/` in the project folder structure and serves as `example.com/node_modules/*` in express. I believe this currently serves all of your `node_modules` so be aware of that, I will be trying to ensure only paths starting with @'s are accessible ASAP.

### View server

The port defaults to `8081` unless that port is occupied in which case behaviour is the same as polyserve it will keep trying ports until it finds an unused one. Alternatively define `port: ''` in your options object to set the port manually.
 
Navigate to `localhost:8081/` to view your served app.

### Options

 * `--version`: Print version info.                                                           
 * `--root` _string_: The root directory of your project. Defaults to the current working directory.                                                                    
 * `--compile` _string_: Compiler options. Valid values are "auto", "always" and "never". "auto" compiles JavaScript to ES5 for browsers that don't fully support ES6.         
 * `--module-resolution` _string_: Algorithm to use for resolving module specifiers in import and export statements when rewriting them to be web-compatible. Valid values are "none" and "node". "none" disables module specifier rewriting. "node" uses Node.js resolution to find modules.
 * `--compile-cache` _number_: Maximum size in bytes (actually, UTF-8 characters) of the cache used to store results for JavaScript compilation. Cache size includes the uncompiled and compiled file content lengths. Defaults to 52428800 (50MB).
 * `-p`, `--port` _number_: The port to serve from. Serve will choose an open port for you by default.
 * `-H`, `--hostname` _string_: The hostname to serve from. Defaults to localhost.          
 * `-c`, `--component-dir` _string_: The component directory to use. Defaults to reading from the Bower config (usually bower_components/).
 * `-u`, `--component-url` _string_: The component url to use. Defaults to reading from the Bower config (usually bower_components/).
 * `-n`, `--package-name` _string_: The package name to use for the root directory. Defaults to reading from bower.json.
 * `--npm`: Sets npm mode: component directory is "node_modules" and the package name is read from package.json
 * `-o`, `--open`: The page to open in the default browser on startup.
 * `-b`, `--browser` _string[]_: The browser(s) to open with when using the --open option. Defaults to your default web browser.
 * `--open-path` _string_: The URL path to open when using the --open option. Defaults to "index.html".
 * `-P`, `--protocol` _string_: The server protocol to use {h2, https/1.1, http/1.1}. Defaults to "http/1.1".
 * `--key` _string_: Path to TLS certificate private key file for https. Defaults to "key.pem".
 * `--cert` _string_: Path to TLS certificate file for https. Defaults to "cert.pem".
 * `--manifest` _string_: Path to HTTP/2 Push Manifest.    
 * `--proxy-path` _string_: Top-level path that should be redirected to the proxy-target. E.g. `api/v1` when you want to redirect all requests of `https://localhost/api/v1/`.
 * `--proxy-target` _string_: Host URL to proxy to, for example `https://myredirect:8080/foo`.
 * `--help`: Shows this help message

### Run Tests

    $ npm test
