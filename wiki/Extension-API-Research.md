## Status

* This is a revision based on previous extension research and some [Extension Robustness](Extension-Robustness) thoughts
* Originally, created as part of the [Extension API research story](https://trello.com/c/rnN0XwK0/876-3-research-extension-api-design). See the "Proposed API changes" section for the start of this research.
* (See ["Extension API Evolution"](Extensions2) for earlier thoughts than this proposal, including more detail on the longer-term ideas such as restartless extensions & sandboxing)

# The Patterns of Brackets 1.0 Extensions

This section is by Kevin Dangoor from July 2014.

## Introduction

Brackets has grown tremendously since its initial release and the overall architecture has held up well with that growth. That said, there are some things that we've learned over time that could change some of the patterns used in extensions. This isn't about the [entire surface area of the Brackets API](http://brackets.io/docs/current/modules/brackets.html), which is something we improve incrementally. This is about improving patterns that appear in *every single extension*.

* `brackets.getModule` was always considered to be something of a hack to allow extensions to get at Brackets core code
* Extensions have no way of sharing services
* [Promises/A+](http://promisesaplus.com/) has caught on and those basic semantics are already shipping as a standard Promise object in Chrome and Firefox
* jQuery promises [don't quite follow this spec](https://github.com/kriskowal/q/wiki/Coming-from-jQuery)
* Brackets has error handling needs that are different from many client side projects because extension code is not maintained by the same people maintaining Brackets itself. Errors in extensions can significantly impact Brackets' operation.

The goal of this project is to make changes to patterns that spread throughout Brackets and extensions to improve our foundation for 1.0 and beyond:

* Module loading
* Promises
* Events

This document serves as a presentation of the basic idea with some strawman proposals. The final implementation will vary based on research and feedback.

## Module Loading

The changes we'd make to module loading are:

1. The same mechanism is used to load modules from Brackets as is used for loading extension modules
2. Brackets core modules will be under a "core" namespace ("core/ProjectManager", for example).
3. The door will be opened for extensions to use services provided by other extensions. This wouldn't be enabled at first, but this is the reason Brackets core features would be in the "core" namespace.
4. core modules would not have had to have been loaded prior to use in extensions as they do with brackets.getModule

Secondary features:

1. Easy stubbing/mocking of modules for unit testing
2. Ability to drop in modules installed via npm

We are at a crossroads in JavaScript modules right now. ECMAScript 6 (ES6, the next version of the JavaScript standard) will soon have a [native module system](http://jsmodules.io/) that draws heavily from the CommonJS module system. The module specs and transpilers have matured to a point at which we could reasonably switch to ES6 modules. There are even [module](https://github.com/systemjs/systemjs) [loaders](http://webpack.github.io/) that can handle AMD, CommonJS and ES6 modules. Looking purely to the future, ES6 modules would be the way to go. However, it would be reasonable to choose CommonJS modules today, because that is what Node supports natively and what the tens of thousands of packages in npm are designed to support. A small bit of research could likely show us whether ES6 module interop is good enough now to make the leap. AngularJS 2.0 [makes the leap to ES6 modules](https://github.com/angular/watchtower.js/blob/master/src/watch.js) as does [Ember 1.6](https://github.com/emberjs/ember.js/blob/92a844e65059a402c2435fc983033be01da9f83b/packages_es6/ember-views/lib/main.js) so it may not be an unreasonable choice.

The module format is less important than how extensions refer to other modules that they need to load.

There is some more detail on the [module loader and module format card](https://trello.com/c/Wtv5a74b/992-new-module-loader-module-format) in Trello.

## Promises

With the Promise object already appearing in [48% of browsers](http://caniuse.com/#search=promise) (with Safari coming soon), the standard is clear and does not behave as jQuery promises do. With a standard in hand, it is likely that more libraries will start using promises and that alone is a good benefit for switching. Beyond that, error handling is better in other promises implementations. Troubleshooting asynchronous behavior is difficult enough without errors getting transparently swallowed.

In addition to the non-standard error handling, jQuery also has an API that is different from the standard (different names, different chaining) and jQuery will synchronously call a handler whereas the standard ensures that all handlers are called asynchronously.

[Bluebird](https://github.com/petkaantonov/bluebird) and [Q](https://github.com/kriskowal/q) are both well regarded promises libraries that we could choose from. Both libraries can wrap a jQuery promise with their own to ensure consistent behavior. Both can provide [additional debugging information](https://github.com/kriskowal/q#long-stack-traces) during development that show where the promise was created. The most important aspects for us today are:

1. following the standard
2. providing a clear deprecation/upgrade path

Changing promises implementations is potentially the most difficult change, as far as backwards compatibility is concerned and there is a [research card to study the impact](https://trello.com/c/qJ0TgoVu/1361-s-research-promises-upgrade). An initial quick look at a few extensions made it appear promising that we can provide a nice upgrade path without breaking many, if any, extensions.

## Events

A misbehaved event handler can prevent other handlers from receiving events, which has the potential to make many parts of Brackets fail in ways that are hard to trace. This is not a problem that typical web applications have and capturing exceptions in handlers can potentially slow down notifications because v8 does not optimize functions with try/catch. For Brackets, including the try/catch in most event notifications will ensure that even if one handler fails, the rest get the message.

Additionally, we can reduce coupling by using a global event bus between subsystems and an EventEmitter pattern within subsystems or when dealing with specific, non-singleton objects. This will make testing easier and provide an easy place to hook in logging for debugging.

Also, we can potentially reduce the chances of typos in event names by warning about unregistered event names. (Check for undefined events after all extensions have loaded.)

AppInit could be replaced by a type of channel that fires once per subscriber and remembers that it has fired.

The first argument to event handlers can be an "envelope" that provides metadata about the event. This will ease backwards compatibility for handlers as jQuery passes an event object as the first argument to its handlers.

I have looked a number of EventEmitter and event bus implementations. None have the combination of features we need, particularly around error handling. The reason there are so many is that they're not hard to write, so we should just make our own.

## Backward compatibility

The good news is that we should be able to provide deprecation warnings and give a backwards-compatible transition to the new style. The biggest question mark around compatibility is with the promises upgrade.

## Strawman Examples

I used the main.js file from brackets-git-info as a sample. I stripped out most of the file to focus on the important bits.

### Current version

```javascript
// Adapted from brackets-git-info's main.js

/*global brackets, define, $, window, Mustache */
define(function (require, exports, module) {
    'use strict';
    
    var AppInit             = brackets.getModule("utils/AppInit"),
        CommandManager      = brackets.getModule("command/CommandManager"),
        Dialogs             = brackets.getModule("widgets/Dialogs"),
        ExtensionUtils      = brackets.getModule("utils/ExtensionUtils"),
        FileUtils           = brackets.getModule("file/FileUtils"),
        Menus               = brackets.getModule("command/Menus"),
        FileSystem          = brackets.getModule("filesystem/FileSystem"),
        LanguageManager     = brackets.getModule("language/LanguageManager"),

        NodeConnection      = brackets.getModule("utils/NodeConnection"),
        DocumentManager     = brackets.getModule("document/DocumentManager"),
        ProjectManager      = brackets.getModule("project/ProjectManager"),
        qunitRunner         = require("main_qunit"),
        jasmineRunner       = require("main_jasmine"),
        jasmineNodeRunner   = require("main_jasmine_node"),
        nodeRunner          = require("main_node"),
        yuiRunner           = require("main_yui"),
        MyStatusBar         = require("MyStatusBar");

    var moduledir           = FileUtils.getNativeModuleDirectoryPath(module),
        commands            = [],
        YUITEST_CMD         = "yuitest_cmd",
        JASMINETEST_CMD     = "jasminetest_cmd",
        QUNITTEST_CMD       = "qunit_cmd",
        SCRIPT_CMD          = "script_cmd",
        NODETEST_CMD        = "nodetest_cmd",
        GENERATE_JASMINE_CMD = "generate_jasmine_cmd",
        GENERATE_QUNIT_CMD  = "generate_qunit_cmd",
        GENERATE_YUI_CMD    = "generate_yui_cmd",
        VIEWHTML_CMD        = "viewhtml_cmd",
        projectMenu         = Menus.getContextMenu(Menus.ContextMenuIds.PROJECT_MENU),
        workingsetMenu      = Menus.getContextMenu(Menus.ContextMenuIds.WORKING_SET_MENU),
        nodeConnection      = new NodeConnection(),
        _windows            = {},
        enableHtml          = false;


    // opens an html file in a new window
    function viewHtml() {
        var entry = ProjectManager.getSelectedItem();
        if (entry === undefined) {
            entry = DocumentManager.getCurrentDocument().file;
        }
        var path = entry.fullPath;
        var w = window.open(path);
        w.focus();
    }


    // reads config.js to determine if brackets-xunit should be disabled for the current project
    function readConfig() {
        var result = new $.Deferred();
        var root = ProjectManager.getProjectRoot(),
            configFile = FileSystem.getFileForPath(root.fullPath + "config.js");
        FileUtils.readAsText(configFile).done(function (text) {
            try {
                var config = JSON.parse(text);
                if (config.hasOwnProperty('brackets-xunit') && config['brackets-xunit'] === 'disable') {
                    result.reject('disabled');
                }
            } catch (e) {
                console.log("[brackets-xunit] reading " + root.fullPath + "config.js Error " + e);
            } finally {
                return result.resolve('ok');
            }
        }).fail(function () {
            return result.resolve('ok');
        });
        return result.promise();
    }

    /*
     * cleanMenu - removes all brackets-xunit menu items from a menu
     * parameters: menu - the WorkingSetMenu or the ProjectMenu
     */
    function cleanMenu(menu) {
        var i;
        for (i = 0; i < commands.length; i++) {
            menu.removeMenuItem(commands[i]);
        }
    }
    // setup, connects to the node server loads node/JasmineDomain and node/ProcessDomain
    AppInit.appReady(function () {
        $(DocumentManager)
        .on("documentSaved.xunit", function (e, d) {
            runTestsOnSaveOrChange(d);
        });


        $(DocumentManager)
        .on("currentDocumentChange", function () {
            runTestsOnSaveOrChange(DocumentManager.getCurrentDocument());
        });

        MyStatusBar.initializePanel();

    });




    // Register commands as right click menu items
    commands = [ YUITEST_CMD, JASMINETEST_CMD, QUNITTEST_CMD, SCRIPT_CMD, NODETEST_CMD, GENERATE_JASMINE_CMD,
                GENERATE_QUNIT_CMD, GENERATE_YUI_CMD, VIEWHTML_CMD];
    CommandManager.register("Run YUI Unit Test", YUITEST_CMD, runYUI);
    CommandManager.register("Run Jasmine xUnit Test", JASMINETEST_CMD, runJasmine);
    CommandManager.register("Run QUnit xUnit Test", QUNITTEST_CMD, runQUnit);
    CommandManager.register("Run Script", SCRIPT_CMD, runScript);
    CommandManager.register("Run Jasmine-Node xUnit Test", NODETEST_CMD, runJasmineNode);
    CommandManager.register("Generate Jasmine xUnit Test", GENERATE_JASMINE_CMD, generateJasmineTest);
    CommandManager.register("Generate Qunit xUnit Test", GENERATE_QUNIT_CMD, generateQunitTest);
    CommandManager.register("Generate YUI xUnit Test", GENERATE_YUI_CMD, generateYuiTest);
    CommandManager.register("xUnit View html", VIEWHTML_CMD, viewHtml);

    // check if the extension should add a menu item to the project menu (under the project name, left panel)
    $(projectMenu).on("beforeContextMenuOpen", function () {
        var selectedEntry = ProjectManager.getSelectedItem(),
            text = '';
        if (selectedEntry && selectedEntry.fullPath && DocumentManager.getCurrentDocument() !== null && selectedEntry.fullPath === DocumentManager.getCurrentDocument().file.fullPath) {
            text = DocumentManager.getCurrentDocument().getText();
        }
        cleanMenu(projectMenu);
        readConfig().done(function () {
            checkFileTypes(projectMenu, selectedEntry, text);
        });
    });

    // check if the extension should add a menu item to the workingset menu (under Working Files, left panel)
    $(workingsetMenu).on("beforeContextMenuOpen", function () {
        var selectedEntry = DocumentManager.getCurrentDocument().file,
            text = DocumentManager.getCurrentDocument().getText();
        cleanMenu(workingsetMenu);
        readConfig().done(function () {
            checkFileTypes(workingsetMenu, selectedEntry, text);
        });
    });
    exports.formatTime = formatTime;
    exports.checkFileTypes = checkFileTypes;
    exports.determineFileType = determineFileType;
});
```

### Deprecation Warnings

* Replace `brackets.getModule` with `require("core/*")`
* Deprecation warning for `FileUtils.readAsText().done()` and `.fail()`. These should be `.then` and `.catch`.
* `AppInit.appReady` should be replaced with `EventBus.on("AppInit.appReady")`
* `$(DocumentManager).on` should be replaced with `EventBus.on("DocumentManager.")` See [DropletJS.PubSub's message syntax](https://www.npmjs.org/package/dropletjs.pubsub) for a way to think about event bus channels.
* `$(contextMenu).on("beforeContextMenuOpen")` should be replaced with `EventBus.on("Menus.contextMenu.beforeOpen")`

### Updated Version (Sticking with RequireJS)

```javascript
// Adapted from brackets-git-info's main.js

/*global brackets,$, window, Mustache */
define(function (require, exports, module) {
    'use strict';

    var EventBus            = require("core/EventBus"),
        CommandManager      = require("core/command/CommandManager"),
        Dialogs             = require("core/widgets/Dialogs"),
        ExtensionUtils      = require("core/utils/ExtensionUtils"),
        FileUtils           = require("core/file/FileUtils"),
        Menus               = require("core/command/Menus"),
        FileSystem          = require("core/filesystem/FileSystem"),
        LanguageManager     = require("core/language/LanguageManager"),
        NodeConnection      = require("core/utils/NodeConnection"),
        DocumentManager     = require("core/document/DocumentManager"),
        ProjectManager      = require("core/project/ProjectManager"),
        qunitRunner         = require("main_qunit"),
        jasmineRunner       = require("main_jasmine"),
        jasmineNodeRunner   = require("main_jasmine_node"),
        nodeRunner          = require("main_node"),
        yuiRunner           = require("main_yui"),
        MyStatusBar         = require("MyStatusBar");

    var moduledir           = FileUtils.getNativeModuleDirectoryPath(module),
        commands            = [],
        YUITEST_CMD         = "yuitest_cmd",
        JASMINETEST_CMD     = "jasminetest_cmd",
        QUNITTEST_CMD       = "qunit_cmd",
        SCRIPT_CMD          = "script_cmd",
        NODETEST_CMD        = "nodetest_cmd",
        GENERATE_JASMINE_CMD = "generate_jasmine_cmd",
        GENERATE_QUNIT_CMD  = "generate_qunit_cmd",
        GENERATE_YUI_CMD    = "generate_yui_cmd",
        VIEWHTML_CMD        = "viewhtml_cmd",
        projectMenu         = Menus.getContextMenu(Menus.ContextMenuIds.PROJECT_MENU),
        workingsetMenu      = Menus.getContextMenu(Menus.ContextMenuIds.WORKING_SET_MENU),
        nodeConnection      = new NodeConnection(),
        _windows            = {},
        enableHtml          = false;


    // opens an html file in a new window
    function viewHtml() {

        // MIGRATION NOTE: Some direct uses of ProjectManager and DocumentManager would not be required because
        // events will convey the information necessary. Unless CommandManager changes to pass in the current document
        // and currently selected file, these uses will still be required.
        var entry = ProjectManager.getSelectedItem();
        if (entry === undefined) {
            entry = DocumentManager.getCurrentDocument().file;
        }
        var path = entry.fullPath;
        var w = window.open(path);
        w.focus();
    }


    // reads config.js to determine if brackets-xunit should be disabled for the current project
    function readConfig() {
        // Extensions can still use jQuery promises if they want, though they'd be encouraged to switch to real promises.
        var result = new $.Deferred();
        var root = ProjectManager.getProjectRoot(),
            configFile = FileSystem.getFileForPath(root.fullPath + "config.js");
        FileUtils.readAsText(configFile).then(function (text) {
            try {
                var config = JSON.parse(text);
                if (config.hasOwnProperty('brackets-xunit') && config['brackets-xunit'] === 'disable') {
                    result.reject('disabled');
                }
            } catch (e) {
                console.log("[brackets-xunit] reading " + root.fullPath + "config.js Error " + e);
            } finally {
                return result.resolve('ok');
            }
        }).catch(function () {
            return result.resolve('ok');
        });
        return result.promise();
    }

    EventBus.on("xunit:AppInit.appReady", function () {
        EventBus.on("xunit:DocumentManager.document.saved", function (e, d) {
            runTestsOnSaveOrChange(d);
        });


        EventBus.on("xunit:DocumentManager.currentDocument.changed", function (e, d) {
            runTestsOnSaveOrChange(d);
        });

        MyStatusBar.initializePanel();

    });




    // Register commands as right click menu items
    commands = [ YUITEST_CMD, JASMINETEST_CMD, QUNITTEST_CMD, SCRIPT_CMD, NODETEST_CMD, GENERATE_JASMINE_CMD,
                GENERATE_QUNIT_CMD, GENERATE_YUI_CMD, VIEWHTML_CMD];
    CommandManager.register("Run YUI Unit Test", YUITEST_CMD, runYUI);
    CommandManager.register("Run Jasmine xUnit Test", JASMINETEST_CMD, runJasmine);
    CommandManager.register("Run QUnit xUnit Test", QUNITTEST_CMD, runQUnit);
    CommandManager.register("Run Script", SCRIPT_CMD, runScript);
    CommandManager.register("Run Jasmine-Node xUnit Test", NODETEST_CMD, runJasmineNode);
    CommandManager.register("Generate Jasmine xUnit Test", GENERATE_JASMINE_CMD, generateJasmineTest);
    CommandManager.register("Generate Qunit xUnit Test", GENERATE_QUNIT_CMD, generateQunitTest);
    CommandManager.register("Generate YUI xUnit Test", GENERATE_YUI_CMD, generateYuiTest);
    CommandManager.register("xUnit View html", VIEWHTML_CMD, viewHtml);

    // check if the extension should add a menu item to the project menu (under the project name, left panel)
    EventBus.on("xunit:Menus.projectMenu.beforeOpen", function (e, projectMenu, selectedEntry) {
        var text = '';
        if (selectedEntry && selectedEntry.fullPath && DocumentManager.getCurrentDocument() !== null && selectedEntry.fullPath === DocumentManager.getCurrentDocument().file.fullPath) {
            text = DocumentManager.getCurrentDocument().getText();
        }
        cleanMenu(projectMenu);
        readConfig().done(function () {
            checkFileTypes(projectMenu, selectedEntry, text);
        });
    });

    // check if the extension should add a menu item to the workingset menu (under Working Files, left panel)
    EventBus.on("xunit:Menus.workingSetMenu.beforeOpen", function (e, workingSetMenu) {
        var selectedEntry = DocumentManager.getCurrentDocument().file,
            text = DocumentManager.getCurrentDocument().getText();
        cleanMenu(workingSetMenu);
        readConfig().done(function () {
            checkFileTypes(workingsetMenu, selectedEntry, text);
        });
    });
    exports.formatTime = formatTime;
    exports.checkFileTypes = checkFileTypes;
    exports.determineFileType = determineFileType;
});
```

### Event Emitter example

In the section on Events, I talk about using an event bus between subsystems and event emitter within a subsystem. Most extensions would likely use the event bus rather than the event emitters, because extensions are across subsystem boundaries. An example of where we'd use an event emitter can be found in EditorManager:

Current:

```javascript
function _createEditorForDocument(doc, makeMasterEditor, container, range) {
    var editor = new Editor(doc, makeMasterEditor, container, range);

    $(editor).on("focus", function () {
        _notifyActiveEditorChanged(this);
    });

    return editor;
}
```

New: 

```javascript
function _createEditorForDocument(doc, makeMasterEditor, container, range) {
    var editor = new Editor(doc, makeMasterEditor, container, range);

    editor.on("focus", function () {
        _notifyActiveEditorChanged(this);
    });

    return editor;
}
```

The pattern changes only slightly from the current system (gets rid of jQuery use here, has the error handling that we want).

## Benefits

* Shrinks the difference between core code and extension code by putting core and extension code into a consistent namespace
* Modules provided by core do not need to be loaded before an extension can request them
* First step in enabling extensions to share services
* Robust and easier to debug error handling for promises and events
* Event bus provides looser coupling between subsystems which can make testing easier, reduce circular dependencies
* Event bus is also a mechanism through which extensions could communicate some events today

## Implementation Plan

### Phase 1: Research and Infrastructure

The goal of this phase is to quickly answer any questions, identify potential problems and lay the groundwork for rapid implementation of the real changes. We can do these parts in parallel.

For the purposes of testing, "popular extensions" refers to:

* Brackets Git
* Code Folding
* Emmet
* PHP Syntax Hint
* Beautify
* Extensions Rating
* Brackets File Icons
* WordHint

#### Promises

Though we'll ultimately want to use a more full-featured promises implementation, both Q and Bluebird use a `done` method with a different meaning than the one offered by jQuery. To make the transition easier, we can stick with ES6 promises and choose a library with more features once we're ready to remove the deprecated wrapping.

Note that extensions can still use jQuery promises if they wish because extensions don't send promises into core code (as far as I know).

* [Use the es6-promise](https://github.com/jakearchibald/es6-promise) shim until we've updated to a Chromium version that includes the native Promise object
* Augment or wrap promises so that they include `done` and `fail` that issue deprecation warnings but otherwise behave like jQuery promises (returning the same promise).
* Update our `utils/Async` to use new promises
* Replace all instances of `new $.Deferred` with a use of standard promises
* Test popular extensions to make sure they still work and display expected deprecation warnings

See [the card in Trello](https://trello.com/c/qJ0TgoVu/1361-s-research-promises-upgrade).

#### Modules

We should be able to provide 100% backwards compatibility with new module loading while also setting the stage for cross-extension dependencies.

* Update to latest cajon
* Configure `core` package to load Brackets core modules (see [James Burke's comments](https://github.com/adobe/brackets/issues/4986) for hints)
* Make deprecated version of `brackets.getModule` that uses global `require` context but adds "core/" to the module.
* Change extension loading to no longer create a new require context
* Verify that JavaScript Code Hints still works (it loads require into a Web Worker)
* Make sure minified builds work
* Test popular extensions

See [the card in Trello](https://trello.com/c/Qk7uqIw8/991-research-extension-loader-implementation)

#### Events

See [the Events section](https://github.com/adobe/brackets/wiki/Extension-API-Research#events) of the extension API research. I have been unable to find a library that does precisely what we want. We want an EventEmitter and global Event Bus (which is basically a singleton EventEmitter) with these features:

* Channel/Event names follow a "dotted.name.format"
* Events should be registered. An options object can be used for future expansion (should a channel's messages not be wrapped in try/catch for performance reasons, for example). A description of the purpose of the event and information about the data sent to the handler should be included.
* A method will return information about the registered events (imagine an eventual UI that tells extension developers all of the events available in the system)
* A method is available to verify that the events listeners are listening for have been registered
* Listeners can specify a name for easy unregistration in the format: "listenername:dotted.event.name"
* Wildcards can be used to match a segment of the channel name. `dotted.*.name` will match "dotted.event.name" and "dotted.foo.name". A `*` only matches a whole segment.
* If the published message goes to a channel with more segments in the name than a listener specified, that is still considered a match. For example, if a listener subscribes to "dotted", that will match any channel that starts with dotted, including "dotted.event.name".
* Handlers are called in a try/catch block
* The first argument to a handler is an envelope with metadata. The only required metadata in the first version is the full channel name that the event is sent to.
* `on` is used to listen
* `off` is used to stop listening
* `trigger` is used to publish a message
* Consider having the EventEmitters roll up to the global bus in some fashion to assist in debugging

To make the initial implementation easier, we could eliminate the "wildcards" bullet point while retaining the next point about shorter matches. This would still allow listening to all DocumentManager messages, for example, by setting up a subscriber for simply `DocumentManager`. All messages could be subscribed to by listening to "".

Here are the tasks to prepare for rollout:

* Implement EventEmitter
* Implement EventBus
* Create a wrapper for `$(object).on()` and `$(object).off()` to issue deprecation warnings for events that have been changed over. Specifically check that `object` is a Brackets object to avoid false matches.
* Migrate DocumentManager to the new style
* Test popular extensions

See [the card in Trello](https://trello.com/c/ogaZoRHJ/1377-events-infrastructure-implementation)

### Phase 2: Fast rollout

With potential problems resolved and infastructure set up, I think that the least disruptive way to roll this out is all at once with a guide for extension developers that explains how to transition.

* Change Promises throughout the core code
* Integrate module loading change
* Merge in new event system and start changing commonly used events
* Write migration guide for extension developers (note that all changes should be accompanied by deprecation warnings)

# Previously Proposed API changes

This section is the broader research done by Peter Flynn in the previous phase of which the pieces above are a part.

In priority / implementation order:

#### 1. Better module loader
* Easy shimming/mocking for unit tests
* Extensions authors get code hints for core APIs
* Less verbose per-module boilerplate, matching Node module format more closely
* **[Sample code here...](https://gist.github.com/peterflynn/6323175)**
* [Implementation research notes here...](https://github.com/adobe/brackets/wiki/ModuleLoader)
* **Open question:** What do we do for worker-thread code? E.g. see _tern-worker.js_ in JS code hints &ndash; it looks like an unusual edge case for module loading... _(Added to [implementation research story](https://trello.com/c/Qk7uqIw8/991-research-extension-loader-implementation))._

#### 2. Cross-extension dependencies
* Built atop new module loader: APIs pulled in the same way as core APIs. Factored into the dependency-driven load order.
* Extensions are treated as singleton "service" providers rather than NPM-style dependencies
* Extension Manager automatically pulls down dependency extensions & identifies conflicts
* Cross-extension dependencies _must_ be declared in package.json (to allow Extension Manager to properly manage the dependency). If not declared, any `require()` calls referencing another extension will fail.

#### 3. Transparently expose APIs from Node-side modules
* _Restrictions:_ all APIs are async; all APIs can only receive & return JSON values (no complex objects)
* Makes it easier to do our current arrangement, where more business logic is in shell-V8 code but certain APIs are implemented on Node side
* Makes it easier for core and extensions to reuse useful utilities via NPM packages
* A few NPM packages won't really be usable this way, without loading directly into the shell-side -- e.g. a collections libary

#### 4. Clean up core APIs to simplify extension authoring
* Done incrementally, driven by pain points we see extension authors hitting
* Old APIs would continue to work for a while, with deprecation warnings

#### 5. Allow a generic Node utility module (installed via NPM) to be loaded shell-side
* Easier to reuse some of our own headless utility code
* Enables reusing NPM utility modules, if they're headless (no dependencies on Node-only APIs)

#### Tabled - not in near future
* Track API calls so extensions can be made restartless semi-automagically &ndash; Wait & see how extension auto-updating feels with simpler mitigations (e.g. auto-restart, batched update notifications).
* Run extensions in a sandbox and/or web worker thread &ndash; Wait & see how much extensions affect performance/stability as ecosystem develops. Could _potentially_ be done without changing APIs by mirroring model objects across sandboxes.
* Call Brackets APIs from Node-side code / enable code that uses Brackets APIs to be agnostic as to whether it runs Node-side vs. shell-side &ndash; Doesn't seem like a strong need.

## Open questions

* (See inline above)

#### Resolved questions

* The shared-services architecture (item #1 above) feels akin to writing a whole new module loader (we have to deal with load order, cycle breaking, etc.).
    * _Resolved:_ We think that's the right thing to do. Service dependencies have much in common with module dependencies, but just enough not in common to make reusing something like Require not viable.
    * _Update:_ Using a new module loader is now the _centerpiece_ of the latest proposal.
* Do we require extensions to explicitly declare every core service they depend on?  Should we hide services that are undeclared, to prevent subtle bugs from slipping in? (e.g. extension depends on a service that's implemented in, or later migrated to, a core/default extension; without a declaration, we can't guarantee the load order will be right)
    * _Resolved:_ You must declare your service dependencies _only if_ you are in an extension, and the service is implemented by a different extension in the same "pool" as you (default vs. non-default). We’ll guarantee that all core modules load first, then all default extensions, then all user extensions (the latter part is not true today).  But within a pool, we'll need to use dependencies to determine a "safe" load order. -- This should reduce the burden on extension authors, where very few dependencies will be on other non-default extensions.
    * _Update:_ With a new module loader, this would be _automatically_ taken care of, the same way e.g. Require detects dependencies between core modules.
* Need a nicer pattern for storing off references to services after `load()`. It's too hard to remove all refs from non-init code (see below), and it's ugly to pass `services` as an arg to every other function in the module.
    * _Resolved:_ In new proposal, we continue to use the existing `require()` pattern with references both declared & assigned at top of module.
* Some Brackets core APIs feel weird to be called "services," e.g. StringUtils. Would we permanently leave some modules behind `getModule()`?
    * _Resolved:_ In new proposal, things aren't explicitly labeled as "services."
* Do we still want to define general API principles (to be used by core APIs when we port them over later) in Sprint 29's research story? Seems like what the APIs look like depends a lot on our restartless thinking, which we've deferred a bit (e.g. restartless might imply moving to pub-sub for everything).
    * _Resolved:_ Deferred to a later story that will specifically cover API cleanup. Restartless is lower on the priority list now, so we shouldn't block API cleanups on that anymore (and ideally we'd try to do restartless without imposing big new constraints on API design anyway).
* Do we enforce that the dependency is declared in package.json?
    * _Resolved:_ Yes. It's essential for Extension Manager to know about dependencies, so we want to force authors to always declare them. Module loader _will not_ enforce API version constraints, though - only Extension Manager deal with version issues.
* Should extension references to core modules be behind a single root namespace, to disambiguate the first "path segment" in case of collisions between core folder names & user extension names?
    * _Resolved:_ Not needed -- core modules take precedence. A collision could still occur if core code adds a new folder name conflicting with an existing poorly-named extension that was already providing an API to other extensions, breaking its ability to be referenced as a dependency. But that will be rare, and our docs for cross-extension dependencies can amplify the need to use "."s in extension names.
* Should we deprecate `require()` paths that lack a "./" prefix? This is more consistent with Node and the module loaders we might use for Brackets. We could do it without breaking extensions since the deprecated `brackets.getModule()` API could continue to work the old way.
    * _Resolved:_ Only extension-to-core module references will need a "./" prefix, but it will indeed be required there.
* If we wait too long to implement "Allow a generic Node utility module (installed via NPM) to be loaded shell-side," does it encourage people to use the cross-extension dependency mechanism to share utility code instead? (Which is not really the right model - see discussion of services vs. libraries).
    * _Resolved:_ We think most people will just copy utility libs into each extension since that's the path of least resistance. The alternative cross-extension dependency approach is unlikely to be (ab)used often.

## Prototyping notes

#### Notes from Peter
* I converted four simple extensions over to an approximation of the services architecture: Markdown Preview, Everyscrub, Goto Last Edit, and File Navigation Shortcuts.
    * Caveat: none of these used Node-side code
    * Caveat: I didn't actually get them running against Kevin's branch; the ported code is only hypothetical
    * Caveat: I made up a bunch of API redesigns on the fly. But I think they are all fairly sane.
        *  (kevin) That's not so much a caveat. That's a great exercise that I was hoping we'd do some of during this research
* **Verbosity** - Extensions stayed about the same size (on average only +2 lines), but the splitting of service reference declaration & assignment did feel annoying.
    *  (kevin) I'll note that it actually felt really good to me to remove the `define(function...` stuff at the top of the modules
* **Storing off service references** - 3 of 4 extensions required some retained references (even after a bunch of light refactoring). I would want to store off separate refs for individual services, rather than just storing the whole `services` object, to avoid having to use fully-qualified names on every ref.
* **Restartless**
    * Already run into a _large set of APIs that would need to be auto-restartless_: commands, menu items, bare keybindings, keybinding patching, QuickOpen, EditorManager/DocumentManager listeners, PanelManager listeners, DOM injection (panels, toolbar icons, etc.), stylesheet loading
    * 3 of 4 extensions required _manual unload code_: per-Editor/Document listeners, caches hung off of Editor/Document objects (or could just leave these around), global DOM listeners. Need to play with some more extensions to see if these are the biggest sticking points -- could be mitigated if so:
        * Could offer singleton proxies for 'currentEditor', 'currentDocument', etc. to get around many use cases for per-Editor/Document listeners (w/o hitting the tricky per-object listener cleanup issues below).
        * Could offer a per-file cache/storage service to avoid hanging per-extension data directly off of core objects (though it's so "JavaScriptey" that some authors may keep doing it anyway?)
    * _Getting restartless right isn't easy_ -- in 2 of 4 extensions, I erroneously thought I was done making it restartless, then later realized I was wrong after a more careful review.


## Restartless questions & discussion

* Peter: Realistically, how many extensions would work without any manual work in the extension?  Would we trust extension authors to correctly identify what cleanup (if any) needs to happen manually?  (Not even sure I trust _myself _- see above). How bad would the experience be when a not-actually restartless extension fails to fully clean itself up?
    * We should definitely default to non-restartless unless an extension explicitly declares that it can handle it.
* Peter: Demand from users hasn't been that high yet.  Demand will rise with extension update notifications, but we could do some things to mitigate that - for example:
    * Automate a full restart instead of quit & manual-relaunch
        * (kevin) We talked about this one and it *seems* like the native menu problem should be tractable allowing for simply "refresh" of Brackets rather than relaunch. I will note that we'd also need to make sure that the Node side is restarted properly.
    * Throttle / batch extension update notifications
    * Hold notifications until next launch (or quit, or idle time - it might be a preference kind of thing, since Kevin pointed out he hates notify-on-launch while Peter prefers it)
    * Notify & download updates whenever, but don't force a restart until later (updates take effect whenever restart occurs)
* Need an extremely clear way to document / make predictable _which_ APIs are automatically reversible and which aren't...
* _Listeners_ are a very tricky area: hard for extensions to remember to detach all of them, but hard to make them auto-detach too. Auto-detach seems like it would have to mean:
    * a) Disallow listeners on non-singletons -- move to a pub-sub like architecture
    * b) All event-dispatching objects listen for extension unload to auto-remove listeners. But this implies all event-dispatching objects must be explicitly disposed (can't just be GC'ed). True for Editor and Document already, but this still feels like a nasty requirement...
    * c) Event-dispatching objects lazily check for & remove 'dead' listeners each time they trigger an event. Avoids the above limitations, but might be slow.
    * (kevin) d) if the objects exposed through the ServiceRegistry are actually wrappers that know which extension is accessing them, they can automatically track listeners.
        * (peter) At first I thought this would have the same problem as (b), but I _think_ it could duck that. E.g. all wrapper APIs return wrapper objects, and whenever a wrapper object is generated it's added to an extension-specific wrapper registry. When an extension unloads, we'd traverse its registry and dispose all wrapper objects (letting them unlisten from the core objects they're mirroring), and then throw out the registry itself to let the wrappers get GC'ed.

## Discussion of new proposal

(peter: the top sections above now reflect this proposal)

(kevin)

There are a whole bunch of different ways to approach our extension API and a path forward for it.  In my prototype, I tried to think about a way to make a whole bunch of things possible:

1. Restartlessness
2. Sharing of services between extensions
3. Code hinting (that even works as you dynamically update extensions)
4. Easier access to Node-side code
5. Ability to use Node modules (ideally installed via npm) in Brackets client side code (in addition to in Brackets Node-side code)
6. A possible route to allowing things to run in a Web Worker or other sandbox
7. Better unit testing of core code by handing the core code the other services that it needs (and mock services when testing)

After reviewing my prototype, Peter talked about how making the ServiceRegistry work is similar to writing a module loader. Along the way, I have had similar thoughts. In fact, I am even proposing that we write a module loader so that Brackets can seamlessly load modules from node_modules directories.

Peter did a great job of breaking out the various features so that we can consider their priorities independently. But, at the same time, we need to take a longer term, holistic view to know where we want to end up. Over time, I've become more convinced that restartlessness doesn't *have* to be an important feature, as long as the user experience for updates in unobtrusive.

On the other hand, I've started thinking that perhaps Web Workers/Sandboxes are more important than we've been thinking. The more extensions that a user has, the more likely that those extensions will start impacting the core performance of the editor. It happened with Sublime Text (which now sandboxes extensions) and it happened with Firefox (which has taken a variety of measures over time to deal with problematic extensions). Of course, sandboxing becomes tricky when we want to also provide UI flexibility for extensions (something that Sublime doesn't have to contend with).

In the end, I built a prototype that, at least in prototype quality, provides all of the features that I listed above. But, looking at the extensions that Peter ported to an imaginary ServiceRegistry-based API made me think that perhaps there's another, better path forward.

Our inability today to share modules between extensions and to conveniently share code between the Node side and the client side actually stems from our use of RequireJS. *Part* of my proposal has been to make a new module loader. It occurs to me now, depending on exactly what our priorities are, maybe building a new module loader should *be* the proposal, or at least the very first step.

Our own module loader would be able to load modules in whatever way we deem fit. And, for testing purposes, we can have simple enough control over our module loader to allow us to load in mocks for the services.

A new module loader would also represent a much subtler shift for extension authors. They wouldn't *have* to make any change, but they could start by simply:

* Removing the `define()` call at the beginning of their modules
* Changing from `brackets.getModule()` to `require()`

I think we'd still want to provide a higher-level interface to Node code for extensions, but I think that should be straightforward. Here's an example:

* an extension defines a module called "node.js"
* That module makes functions available on its `exports`
* client side code can `require('extensionName/node')`
* that module on the client side will automatically proxy to the Node side

That's actually a little more convenient than the API my prototype provides (though my prototype provided seamless communication in both directions).

### What about Restartlessness and Sandboxes?

If these *aren't* priorities, then we really are in a position to just do step-by-step incremental improvements to our API over time.

If we *do* decide to go restartless later, we still have some options. In this new scenario, we control the module loader. So, we would have the power to make `require('commands/CommandManager')` return a proxy object that *knows* what extension is using it so that added commands are automatically registered to the given extension.

For sandboxing, the hardest part is anything that manipulates the UI. For anything else, we could create a system of proxy objects that allow the extension author to manipulate the document object, etc. and the changes propagate from a worker to the main thread.

### What about code hinting?

This one should be a priority, I think. The prototype has nice code hinting. It gives the user an idea of common extension points.

We can still do that. If we can recognize that a user is working in the context of an extension, we can make `require()` calls provide hints for modules that are most likely to be of interest to them.


### So, what's this "top-level question"?

I guess there are two, one for Adam and one for the engineers.

Adam: what is the priority of restartlessness and sandboxing, given that they are likely expensive?

Team: what do you think about taking control of our module loading rather than building a services API?

## API Cleanups

I think we should still review extensions that are out there and incrementally update the extension APIs to be more convenient. For example, Markdown Preview adds a document change handler during which it turns off its document listener and turns on a new one. This kind of thing can be made more declarative ("listen for changes to documents with these extensions"). These also seem like the kind of events that could be pub/sub channels if we decided to go that route.