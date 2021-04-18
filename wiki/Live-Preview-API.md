## Overview

TODO

### Document API

TODO

### Server API

Brackets ships with 3 server implementations currently

* `UserServer` - Serves dynamic and static content from the user-specified base URL (File > Project Settings...). Supports live CSS editing but does not support live HTML editing. Typically used for server-side languages like PHP.
* `StaticServer` - Serves static content from the built-in HTTP server (via Node.js).  Supports both live CSS editing and live HTML editing. Live HTML editing works by serving 
* `FileServer` - Serves static `file:` URL sites from the local file system (subject to browser constraints).

```
/**
 * @constructor
 * Base class for live preview servers
 *
 * @param {!{baseUrl: string, root: string, pathResolver: function(string): string}} config
 *    Configuration parameters for this server:
 *        baseUrl       - Optional base URL (ProjectManager.getProjectRoot().fullPath)
 *        pathResolver  - Function to covert absolute native paths to project relative paths (ProjectManager.makeProjectRelativeIfPossible(absoluteFilePath))
 *        root          - Native path to the project root (and base URL)
 */
function BaseServer(config) {}

/**
 * Returns a base url for current project. 
 *
 * @return {string}
 * Base url for current project.
 */
BaseServer.prototype.getBaseUrl = function () {};

/**
 * Returns a URL for a given path
 * @param {string} path Absolute path to covert to a URL
 * @return {?string} Converts a path within the project root to a URL.
 *  Returns null if the path is not a descendant of the project root.
 */
BaseServer.prototype.pathToUrl = function (path) {};

/**
 * Convert a URL to a local full file path
 * @param {string} url
 * @return {?string} The absolute path for given URL or null if the path is
 *  not a descendant of the project.
 */
BaseServer.prototype.urlToPath = function (url) {};

/**
 * Called by LiveDevelopment before to prepare the server before navigating
 * to the project's base URL. The provider returns a jQuery promise.
 * The Live Development launch process waits until the promise
 * is resolved or rejected. If the promise is rejected, an error window
 * is shown and Live Development does not start..
 *
 * @return {jQuery.Promise} Promise that may be asynchronously resolved
 *  when the server is ready to handle HTTP requests.
 */
BaseServer.prototype.readyToServe = function () {};

/**
 * Determines if this server can serve local file. LiveDevServerManager
 * calls this method when determining if a server can serve a file.
 * @param {string} localPath A local path to file being served.
 * @return {boolean} true When the file can be served, otherwise false.
 */
BaseServer.prototype.canServe = function (localPath) {};

/**
 * Adds a live document to server
 * @param {Object} liveDocument
 */
BaseServer.prototype.add = function (liveDocument) {};

/**
 * Removes a live document from the server
 * @param {Object} liveDocument
 */
BaseServer.prototype.remove = function (liveDocument) {};

/**
 * Clears all live documents currently attached to the server
 */
BaseServer.prototype.clear = function () {};

/**
 * Start the server
 */
BaseServer.prototype.start = function () {};

/**
 * Stop the server
 */
BaseServer.prototype.stop = function () {};
```

#### Example Server Implementation

The snippet below is simplified from the [Theseus](https://github.com/adobe-research/theseus) JavaScript Debugger Extension. It shows a minimal implementation for creating and registering a live preview server.

The _interesting_ part of Theseus isn't shown here. Like our built-in `StaticServer`, Theseus' `ProxyServer` uses [Connect](http://www.senchalabs.org/connect/) middleware to intercept HTTP requests and serve instrumented content back to the browser.

```
var LiveDevelopment      = brackets.getModule("LiveDevelopment/LiveDevelopment"),
    LiveDevServerManager = brackets.getModule("LiveDevelopment/LiveDevServerManager"),
    BaseServer           = brackets.getModule("LiveDevelopment/Servers/BaseServer");

function ProxyServer(config) {
    BaseServer.call(this, config);
}

ProxyServer.prototype = Object.create(BaseServer.prototype);
ProxyServer.prototype.constructor = ProxyServer;

ProxyServer.prototype.canServe = function (localAbsoluteFilePath) {
    return myTestIfFileTypeCanBeServedAsAWebPage(localAbsoluteFilePath);
};

ProxyServer.prototype.readyToServe = function () {
    var deferred = new $.Deferred();
    // ... async initialization of server ...
    return deferred.promise();
};

function _createProxyServer() {
    var config = {
        pathResolver    : ProjectManager.makeProjectRelativeIfPossible,
        root            : ProjectManager.getProjectRoot().fullPath
    };
    
    return new ProxyServer(config);
}

// register server provider
LiveDevServerManager.registerProvider({ create: _createProxyServer }, 10);
```