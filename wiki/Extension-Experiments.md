## THIS PAGE IS OUT OF DATE AND KEPT FOR HISTORICAL REASONS ONLY

##Goal

The goal of this exercise was to write a feature as an extension, without modifying core Brackets code, and document the issues and pain points. The extension needs to be functional, but not necessarily "release ready". The main objective was to hit on all of the extensibility points required to write a robust, full-featured extension.

The original goal was to write a single extension: an inline editor for JavaScript functions. In the end, three extensions were written.

Additional notes were added via a thought experiment: analyzing what it would take to recast specific baked-in features as separate extensions.

## <a id='inline-js'/>Inline Editor for JavaScript functions

When in a JavaScript file, put the cursor (or selection) on a function name and press Cmd/Ctrl-E. If the function can be found it is displayed in an inline editor. If multiple functions with the same name are found, they can be navigated in the same way that multiple CSS rules can be navigated.

This extension searches all JavaScript files in your project, regardless of whether they are used or not.

This extension was possible without any changes to core Brackets code. It is checked in to brackets/src/extensions/disabled.

###Issues:

* CSSInlineEditor class is really just a generic inline editor that supports multiple text ranges. This class should be moved from the CSSInlineEditor module and given a generic name. Done: https://github.com/adobe/brackets/issues/768

* We need file watchers! This plugin scans all JavaScript files in your project and extracts all functions that are defined in the file. This function list is cached on a per-file basis in the FileIndexManager structures. In order to see if the cached data is still valid, the timestamp of the cached data is compared against the timestamp of the file on disk <sup><small>1</small></sup>. Simply checking the timestamp is virtually the same speed as reloading and rescanning the entire file. File watchers would significantly improve performance. This item is already in our backlog: https://trello.com/c/zldzEXmk
<br><small>[1] If the file is in memory and dirty, it is always rescanned</small>

* Needed to access the "private" `_codeMirror` property of Editor to determine the mode of the editor (`getOption("mode")`) and to call the `getTokenAt()` function. We need to decide if we want to simply expose the CodeMirror instance or if we want to add more methods to the Editor class. Filed: https://github.com/adobe/brackets/issues/804

* There is no priority for inline editor providers. Let's say I wanted to add an inline editor that supports editing a particular css selector, but fall back to the default inline CSS editor for all other selectors. There is no way to say my editor takes precedence over the default editor. This probably isn't a big deal for now, but will become an issue if lots of inline editors are written. This should be added to the backlog, but I don't consider it a 1.0 requirement.

###Other notes:

* Per-file information was cached on the FileIndexManager fileInfo structure, which is really handy. We should document best practices around this (each plugin should store all of its data on a single object and make sure it has a unique name).


## <a id='inline-image'/>Inline Image Viewer

If the cursor (or selection) is in a string that is an image filename (basically any string that ends with .jpg, .jpeg, .png, .gif or .svg), the image is displayed in an inline widget.

This extension was possible without any changes to core Brackets code. It is checked in to brackets/src/extensions/disabled.

###Issues

* Uses the `getTokenAt()` method on `_codeMirror` (same as previous extension)

* This inline editor has a file header section that should match the appearance of the file header in MultiRangeInlineEditor. There wasn't a common base class, so I had to duplicate some code. I'm not sure how much abstraction we can really get here since I only wanted the file name, but MultiRangeInlineEditor also has line number. I recommend not doing anything here for now, but let's keep an eye on it to see if more inline editors are running into the same issue.

## <a id='code-theme'/>Ambiance Theme for Code Editor

This extension adds a menu item to the "Experimental:" section of the Debug menu. When selected, a dark theme is applied to the main code editor and the menu item is checked. 

This extensions was *not* fully possible without changing core Brackets code. The menu item could be added (in a very hacky way), but we have no way to add keyboard shortcuts.

This extension is much more experimental in nature, so I checked it in to my personal repo here: https://github.com/gruehle/AmbianceTheme 

###Issues:

* Need a way to add a keyboard shortcut - both the handler and display in menu (if the command is in a menu)
* Need to have a documented, supported place to put extension menu items.

We probably want theme support built into our core Editor classes. If not, we need the following:
* Need a way to get notified whenever an Editor is created. 
* Need access to all editor instances (there is a private `_instances` var, but no public access)

Built-in theme support added to backlog: https://trello.com/c/y5ed9WKY

## <a id='comment-uncomment'/>Comment/Uncomment Command

Not written as an extension - but this analysis considers what it would take to do so. Comment/Uncomment adds or removes "//"-style comments on the currently selected code.

###Issues:

* Need API to add menu item, command, and key binding. Adding a menu/shortcut without modifying core Brackets files requires fragile hacks (which probably get even more fragile if used by more than one extension).
* Need public APIs equivalent to CodeMirror.replaceRange() and CodeMirror.operation(). This will be an issue for any extensions that makes text edits.
* This feature's behavior varies depending on language/mode. Need a way to add support for new languages to this command. There are multiple ways we might achieve this:
    * Way for this extension to expose its own extension point in turn, where sub-extensions can add support for new languages.
    * Way to add language-scoped command handlers (sort of analogous to how we allow OS-scoped key bindings). But then we'd need to offer some language-agnostic utility methods so that the various handlers didn't have tons of code duplication.
    * Require extensions that add a new language/mode to implement a specific interface, which provides either (a) a language-specific comment/uncomment handler, or (b) info on which tokens (if any) are used for line comments, so that we could have a single language-agnostic implementation of the command.
* Is this _specific_ feature something we'd actually want to package as an extension? We've talked about trying to make as many Brackets features be separable extensions as possible. But there are lots of little text-edit gestures like this that come standard with most editors. How much overhead would there be if the core Brackets install relied on dozens of plugins for its basic functionality? Do plugins load slower than other modules? Do many small modules load slower than a few big modules?

## <a id='unittests'/>Unit Test Issues

In Sprint 9, we started looking at how to run Unit Tests on Extensions. Here's the initial proposal:

* Define a Unit Test file to be run from SpecRunner.html. For now, we are loading "unittests.js" from the main extension folder. When package.json support is added, the Unit Test loading point can be defined by extension author. 

* Unit Tests are optional. Obviously, these are not required.

* Need to provide a convenient way to lookup path to extension root folder for specifying test files, etc. ([Issue #831](https://github.com/adobe/brackets/issues/831)).

## <a id='general'/>General Extensibility Issues

* We currently load the "main.js" module for all extensions. We should support package.json files ([http://requirejs.org/docs/api.html#packages](details)) for defining extension loading points -- so that extensions don't all contain identically-named files. When working with multiple extensions, the monotonous file names are hard to distinguish in the working set, Quick Open, etc. Added to backlog: https://trello.com/c/cyKQeDzd
* How do we unit-test extensions? Presumably extensions we ship with the default install of Brackets should have unit test coverage just like our other standard features.
* What if extensions want to share code? How would one extension refer to another's module(s)? What happens if the required dependency extension isn't present? What about optional dependencies? (e.g. extension A says, _if_ extension B is present I'll plug into it to provide enhanced functionality, but if it's not present that's fine too).
* User should be allowed to specify sub-folders in the extensions folder structure to allow for some organization of extensions. Currently, Brackets only scans the first level of folders.
