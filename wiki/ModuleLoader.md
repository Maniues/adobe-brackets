# Module Loader Details

Features that we want (in priority order):

1. ability to load in core modules
2. ability to load in modules from other extensions
3. ability to shim browser modules (like jslint)
4. Support for node\_modules loading
5. Possibility of customizing the object that is returned (to support passing extension context along for automatic unloading)

I have taken a look at three CommonJS module loaders:

* [Cajon](https://github.com/requirejs/cajon) (RequireJS with some added support for CommonJS modules)
* [Inject](https://github.com/linkedin/inject)
* [Link.js](https://github.com/calyptus/link.js) (also used by Inject)

A useful note: Link.js provides more robust parsing for dependencies (and also exports function names and such from its parse). This may be useful. Of note, the addition of Link.js to Inject fixed a bug in which a minifier renamed "require" to "a". Link.js can handle that. We may be able to use exported functions to feed JavaScriptCodeHints.

Cajon/RequireJS and Inject are both extensible.  They are both not perfect for our needs. Neither one:

* would properly handle node\_modules resolution. For example, consider the following scenario with files in node\_modules:
    * blarg/index.js
    * blarg/node\_modules/forble/index.js
    * forble/index.js
    * If an extension does `require("blarg"); require("forble");`, I could not see a way to modify Cajon or Inject to load the correct forble for the module
* In both cases, when they're taking a module ID and figuring out what that maps to, they do it synchronously. We need to be able to do this asynchronously because we need to poke around on disk for things (today, at least). We *could* get around this by generating a list of Brackets and extension files that we can synchronously search for matches. That may even be good for performance.

Inject has a further problem:

* It does not look like `require("otherModuleInExtension")` will work. It appears that that `require` would go back to the root
* `require("./otherModuleInExtension")` would be the needed syntax. This is backwards incompatible for us, but is better for node compatibility.
* Inject is a good deal larger, but part of that is Link.js which looks to be a win anyhow.

Inject's extensibility API *almost* allows us to manipulate module resolution as we would need to. If it passed in the "relative to" value, we could use that to figure out which module is appropriate.

## Resolution Order

Searching in the current directory first is not very compatible with the Node behavior. It's cleaner to state explicitly that you want a local module with "./". Perhaps we deprecate the current directory behavior?

1. Current directory (as in require)
2. Core modules
3. If it begins with "./", "/" or "../", use the same mechanism as node
4. Look in node_modules for the extension
5. Look for other extension packages
6. Look in other node_modules directories (in Brackets itself?)

## Examples

## From a Brackets core module

* require("command/CommandHandler")
    * Look for src/command/CommandHandler in core modules
    * Look for src/command/CommandHandler.js
* require("underscore")
    *  assume underscore is installed in src/node\_modules
    *  Look for src/underscore
    *  Look for src/underscore.js
    *  Look for src/underscore directory
    *  Look for src/node_modules/underscore
    *  Look for src/node_modules/underscore.js
    *  Look for src/node_modules/underscore/package.json
    *  Look for main in package.json
    *  It's there, so load underscore.js per package.json
* require("JavaScriptCodeHints") // an extension, default in this case
    * Look for src/JavaScriptCodeHints
    * Look for src/JavaScriptCodeHints.js
    * Look for src/node\_modules/JavaScriptCodeHints
    * Look for src/node\_modules/JavaScriptCodeHints.js
    * give up
* require("shim:jQuery!thirdparty/jquery.min.js")
    * Look for src/thirdparty/jquery.min.js
    * Adds a shim for jQuery (magic? how does jQuery.noConflict get called? See if link.js offers assistance here. I think Inject has support for this)

## From a Brackets extension

Assume a jstools extension that looks like this:

* main.js
* jsutils.js
* jslint.js

* require("command/CommandHandler")
    * Look for jstools/command (this is the one we should deprecate)
    * Look for src/command
    * Look for src/command/CommandHandler
    * Look for src/command/CommandHandler.js
* require("jsutils")
    * Look for jstools/jsutils
    * Look for jstools/jsutils.js
* require("shim:JSLINT!jslint")
    * Look for jstools/jslint
    * Look for jstools/jslint.js
    * Look for src/jslint
    * Look for src/jslint.js
    * Add shim that exports JSLINT (this one is not so magical)

# Additional Notes

* Look into the [Google module server](https://github.com/google/module-server)