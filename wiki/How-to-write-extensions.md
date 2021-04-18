If you'd like to develop a theme, take a look at [Creating Themes](https://github.com/adobe/brackets/wiki/Creating+Themes)

There are three stages to developing an extension:

1. **[Set up](How-to-write-extensions#creating-an-extension)** the basic scaffolding.
2. **[Develop](How-to-write-extensions#common-how-tos)** your extension and **[debug](How-to-write-extensions#testingdebugging-workflow)** it.
3. **[Package and publish](How-to-write-extensions#publishing-extensions)** your extension for others to use.

Follow these links to the sections below for details!

Looking for inspiration? Check out the **[Extension Ideas list](https://github.com/adobe/brackets/issues?q=label%3A%22Extension+Idea%22)**.

## Creating an Extension

* Open your extensions folder by selecting "Help > Show Extensions Folder" in Brackets
* Inside the `user` folder(*), create a new "yourExtensionName" folder, and inside that create a `main.js` file.
* For a quick start, you can paste in the [[Simple "Hello World" extension]] or the code from an [existing extension](https://brackets-registry.aboutweb.com/) that is similar to what you want to do.
* If you're working on anything big we recommend you post to the [brackets-dev Google group](http://groups.google.com/group/brackets-dev) or the [#brackets IRC channel on freenode](http://freenode.net) early on so you can get feedback (there may be others working on similar ideas!).

> \* Note: Because Extension Manager lets you delete extensions from this location, in the long run it's _**safer** to develop inside the `src/extensions/dev`_ folder. The best way to do that is to [clone the Brackets source](https://github.com/adobe/brackets/wiki/How-to-Hack-on-Brackets) and run from that copy. This also makes it easy to test your extensions with upcoming Brackets changes before they're released.

## Testing/Debugging Workflow

* Edit your main.js file.
* Save the file and restart Brackets via "Debug > Reload With Extensions" to see your changes.
* To debug problems, use "Debug > Show Developer Tools". You can use console.log(), set breakpoints, etc.
    * _The first time you open Developer Tools, you must [disable caching](https://groups.google.com/forum/?fromgroups=#!topic/brackets-dev/E5iqcD8VqD4)_ - otherwise using Reload while dev tools are open will not reflect changes to your extension.

See **[[Debugging Brackets]]** for a more robust two-window workflow.

You can also [write unit tests for your extension](Extension Unit Tests).


## Publishing Extensions

1. Add a **[package.json file](https://github.com/adobe/brackets/wiki/Extension-package-format#packagejson-format)** next to your main.js
2. ZIP up your entire extension folder (the GitHub "Download ZIP" button is handy for this) or use the command `git archive --format zip -o yourextension.zip master` to generate a zip file.
    * Note: we've had difficulty with ZIP files created from Finder on the Mac. If you get an error when uploading your ZIP file, try creating it from the command line instead.
    * If your extension utilizes git submodules, they must be wrapped in the ZIP. For a solution, refer to [this blog](http://www.topbug.net/blog/2012/03/31/archive-a-git-superproject-and-its-submodules) and use [git-archive-all](https://github.com/Kentzo/git-archive-all).
3. Publish your extension by uploading the ZIP to the **[Brackets Extension Registry](https://brackets-registry.aboutweb.com/)**

For more, see [[Extension Registry Help]].


## Common How-Tos

**API docs** are available [online](http://brackets.io/docs/current) or as JSDoc comments inline in the [Brackets source code](https://github.com/adobe/brackets/tree/master/src).

### Using modules

* To load modules from your extension's folder tree, use `require()` with a path relative to your extension's root folder.
* To load modules from Brackets core, use `brackets.getModule()` with a path relative to the Brackets src root.
* You cannot load modules from _other_ extensions (yet).

You can also use other files packaged inside your extension - for example, see "Load a CSS file" below.

### <a name="uihooks"></a>Adding menu items & keyboard shortcuts

_See [[Simple "Hello World" extension]] for a code sample._

For any new behavior, first register a Command that implements your behavior, via ```CommandManager.register()```. This just maps a Command id (string) to your handler function. Use package-style naming for your Command id (e.g. ```"myorg.myextension.mycommand"```) to avoid collisions with other extensions. (See also: [higher-level overview of command architecture](Brackets Development How Tos#wiki-commands)).

**Add a menu item:** Get a top-level menu by calling ```Menus.getMenu()``` with one of the ```AppMenuBar``` constants (currently ```FILE_MENU```, ```EDIT_MENU```, ```VIEW_MENU```, ```NAVIGATE_MENU``` or ```HELP_MENU```).  Then add a menu item via ```theMenu.addMenuItem()```, linking it to your Command id. The menu item's label will be the string name you gave the Command when it was created.

As a convenience, ```addMenuItem()``` also lets you create a keyboard shortcut for your Command at the same time.

**Add a context menu item:** Get a context menu by calling ```Menus.getContextMenu()``` with one of the ```ContextMenuIds``` constants (currently ```EDITOR_MENU```, ```INLINE_EDITOR_MENU```, ```PROJECT_MENU``` or ```WORKING_SET_MENU```).  Then add a menu item via ```theContextMenu.addMenuItem()```, linking it to your Command id exactly like a top level menu item. 

**Add a menu divider** Get a top level or context menu as explained above.  Then add a menu dividers via ```theMenu.addMenuDivider()```. It will default to the last position currently in the menu.  You have the option of placing it with the position parameter ```first``` and ```last```, which will place the divider accordingly. Additionally, you can set position parameter to ```before``` and ```after```, pass in a Command ID, and place the divider accordingly.  

**Add a keyboard shortcut:** To add a keyboard shortcut without any related menu item, call ```KeyBindingManager.addBinding()``` directly, linking a shortcut to your Command id. Be sure to use the [Brackets Shortcuts](https://github.com/adobe/brackets/wiki/Brackets-Shortcuts) page to see which shortcuts are available and to add the shortcuts that you use to the list.

To decline a keyboard event and allow other parts of Brackets to handle it, make your Command handler return a `$.Promise` that is _already_ rejected at the time you return it. (This is useful if you want to override editing keys like Enter only when the cursor lies in certain places, and allow the default behavior in other cases; or always override a key in the code editor but allow default behavior in simpler textfields). _(Note: requires Sprint 18 or later)_


### <a name="newui"></a>Adding new UI elements

**<a name="addpanel"></a>Add a panel below the editor:** Use the CSS class `.bottom-panel`; see the JSLint bottom-panel.html for an example. Add your panel _above_ the status bar using `PanelManager.createBottomPanel("yourExtension.name", $(panelHtml))`. You may see `Resizer.makeResizable()` and manual DOM insertion of panels in some extensions but this practice is being phased out since the introduction of PanelManager.

_**Unofficial techniques**_ - adding UI elements directly through the DOM works, but puts you on shaky ground. Code that does this _will_ break as Brackets updates evolve the UI. Use these code snippets as best practices that behave as nicely as possible given the risks:

> **Add a toolbar icon:** _(unofficial)_ Use `$myIcon.appendTo($("#main-toolbar .buttons"))`.

> **Add a top panel/toolbar:** _(unofficial)_ Use `$myPanel.insertBefore("#editor-holder")`.

**UI design:** Be sure to follow the [[Extension UI Guidelines]].

**Load a CSS file:** use `ExtensionUtils.loadStyleSheet()`. It returns a Promise you can use to track when the CSS is done loading.
<br>_To avoid accidentally breaking core Brackets UI_, place a CSS class on the root of your UI and make sure _all_ your CSS rules include a descendant selector. E.g. instead of `li { ... }` use `.myExtension li { ... }`.

### <a name="featurehooks"></a>Extending specific Brackets features

For each API listed, see [documentation](http://brackets.io/docs/current/) for more details.

**Quick Edit (inline editors):** To create an extension that responds on Ctrl+E (like the inline color picker), use `EditorManager.registerInlineEditProvider()`. If multiple "providers" all want to respond in a given context, however, the first one wins - there's no notion of priority or cycling through providers yet. 

**Quick Docs:** Similar to Quick Edit, but register your provider with `EditorManager.registerInlineDocsProvider()` instead.

**Quick Find Definition:** To provide quick symbol navigation for a new language, use `QuickOpen.addQuickOpenPlugin()`. Register for a specific language id and only return true from `match()` when an "@" prefix is present (see [CSS support](https://github.com/adobe/brackets/blob/master/src/extensions/default/QuickOpenCSS/main.js) for a simple example).

**Quick Open:** To add a new _global_ search feature (like Quick Open), use `QuickOpen.addQuickOpenPlugin()` with an empty `languageIds` array. Pick a new, unique prefix for `match()` to respond to, and register a new command that invokes `QuickOpen.beginSearch()` with your custom prefix. (See the [File Navigation Shortcuts](https://github.com/peterflynn/brackets-editor-nav/blob/master/main.js#L128) extension for a simple example).

**Code Hints:** To create an extension that shows a code hint popup, use `CodeHintManager.registerHintProvider()`. Unlike Quick Edit, these providers can have varying priority to resolve conflicts; more specific providers take precedence - but similar to Quick Edit, only the "winning" provider is shown.

**Jump to Definition:** To create an extension that responds on Ctrl-J (Jump to Definition), use `EditorManager.registerJumpToDefProvider()`. Similar to Quick Edit, the first provider to respond for a given cursor position wins.

**Syntax Coloring:** Extensions can add new code-coloring "modes" via `LanguageManager.defineLanguage()`. See [[Language Support]] for details.

**Linting:** Use `CodeInspection.register()` to provide linting/inspection for a given Language. Just like the built-in JSLint functionality, the provider is invoked whenever a file is opened or saved, and its results are displayed in a panel below the editor (providers may be run more frequently in the future, however). Currently, only one provider is accepted per language, although extensions _can_ replace the default JSLint provider for JavaScript.

**File Tree:** Take a look at the documentation for the [project/ProjectManager](http://brackets.io/docs/current/modules/project/ProjectManager.html) module. Starting with Brackets 0.44, there are functions you can call (`addIconProvider` and `addClassesProvider`) to decorate the tree.

### <a name="tourl"></a>Accessing resources (e.g. images) in your extension

Extensions can get installed at (semi-)arbitrary paths. For example, you might develop your extension in the ```brackets/src/extensions/dev/foo``` directory, but a user might install it in ```/Users/<user>/Library/Application Settings/Brackets/extensions/user/bar```.

Thankfully, the ```require``` context that's passed in to your extension's ```main.js``` file can help you resolve paths. Just call ```require.toUrl``` with the relative (to your module) path you'd like to make relative to the site root. **IMPORTANT:** Make sure you're using the ```require``` object that was passed to your module, _not_ the global ```require``` object.

For example, if you have ```awesome.jpg``` in your extension's top-level ```foo``` folder, you can do ```require.toUrl('./awesome.jpg')```, and it will return something like ```/extensions/dev/foo/awesome.jpg``` when you call it and ```/Users/<user>/Library/Application Settings/Brackets/extensions/user/bar/awesome.jpg``` when your user calls it. The path you give ```toUrl``` should be relative to your extension's top-level folder (yes, subdirectories work), and the URL you get back will be relative to the site root (i.e. it will begin with "/").

### Working with Preferences

Your extension can access Brackets' preferences and define preferences of its own. For preferences specific to your extension, you should make sure that all of the preferences have a prefix so that they don't clobber any other preferences.

For details, see the full **[Preferences System documentation](Preferences System)**. But here's an example that covers the main parts of the API you'll need to know:

```javascript

var PreferencesManager = brackets.getModule("preferences/PreferencesManager"),
    prefs = PreferencesManager.getExtensionPrefs("myextensionname");

// First, we define our preference so that Brackets knows about it.
// Eventually there may be some automatic UI for this.
// Name of preference, type and the default value are the main things to define.
// This is actually going to create a preference called "myextensionname.enabled".
prefs.definePreference("enabled", "boolean", true);

// Set up a listener that is called whenever the preference changes
// You don't need to listen for changes if you can just look up the current value of
// the pref when you're performing some operation.
prefs.on("change", function () {
    // This gets the current value of "enabled" where current means for the
    // file being edited right now.
    doSomethingWith(prefs.get("enabled"));
});

// This will set the "enabled" pref in the same spot in which the user has set it.
// Generally, this will be in the user's brackets.json file in their app info directory.

prefs.set("enabled", false);

// Then save the change
prefs.save();

```

### Accessing Node APIs

Brackets includes a built-in [Node.js](http://nodejs.org/) server that runs as a side process.  Your extension can include code that runs in Node &ndash; accessing useful Node APIs and pulling in helpful NPM libraries.  [Read more on running code in Brackets' Node instance...](https://github.com/adobe/brackets/wiki/Brackets-Node-Process:-Overview-for-Developers#usage-example)

### Further reading

For more on Brackets APIs and architecture, see [[Brackets Development How Tos]].

If you're interested in contributing to the _core_ Brackets codebase, see [[How to Hack on Brackets]].