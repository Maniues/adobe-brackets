APIs and information below **may change** in the future. Breaking API changes are listed in the [release notes](Release Notes).

## API Docs

* Formatted docs at: **http://brackets.io/docs/current**
* All documentation is also inline in the [Brackets source code](https://github.com/adobe/brackets/tree/master/src), as JSDoc comments


## Application Init

There are 2 primary milestones you can wait for during startup, defined in ``utils/AppInit``:
* ``htmlReady`` signals when the initial DOM content of Brackets is loaded (a template that is rendered by Mustache to include translated strings). Once this event fires, it is safe to query for static DOM elements. _Extensions do not need to wait for this event_ - they are always loaded after it. But core modules must wait for this event before querying the DOM.
* ``appReady`` signals when extensions have finished loading, the initially open project is loaded, and the last-open editor/document is loaded. _Waiting for this event is [not usually needed](https://groups.google.com/d/msg/brackets-dev/8sFGe6bhqaM/kx6B5yrmggoJ)_ for extensions or core.

Do not rely on other events such as ``$(document).ready`` or ``window.onload``.

**Under the hood:** the overall Brackets startup sequence unfolds as follows: 

1. Load global third-party libraries (jQuery, LESS, Mustache)
2. RequireJS loads the main.js bootstrapper, which sets up certain global shims (e.g. Compatibility.js module)
3. Load core Brackets modules (and CodeMirror, Lodash), culminating in the _main startup entry point: brackets.js_
4. View-state preferences loaded (bottom of brackets.js)
5. Main static HTML structure created (`_beforeHTMLReady()` in brackets.js)
6. Application-wide DOM event handlers are installed
7. ``htmlReady`` event fires (bottom of brackets.js, after `_beforeHTMLReady()` returns)
8. CodeMirror modes loaded & global preferences loaded (`_onReady()` in brackets.js)
9. Extensions are loaded: built-in first, then user/dev extensions (`ExtensionLoader.init()` call)
10. Initial project is loaded: file tree is populated, working set is restored then active document is opened (`ProjectManager.openProject()` call, followed by `"projectOpen"` handler in MainViewManager)
11. `appReady` event fires

If a new version of Brackets is available, the update notification dialog may appear at any time after step 7.


## Asynchronous APIs ##

Many operations in Brackets return their results asynchronously -- for example, because they involve file I/O. These APIs return a [jQuery Promise](http://api.jquery.com/Types/#Promise) object that you can use to listen for success/failure and retrieve the result.

For working with a sequence of asynchronous operations (in parallel or in serial), the Async utils module may be helpful.

## Core Classes ##

When dealing with files the user is editing, there are three important classes to understand:

* `Editor` represents the view (it wraps a CodeMirror widget) -- either a full-size editor _or_ an inline editor. An Editor can have focus. Use an Editor object to get/set the cursor position, selection, or scroll position. Every Editor is attached to a Document (accessible via `editor.document`).
* `Document` represents the model (the text content of the file). Use the Document object to get or modify the text, or to listen for changes to the text. There may be multiple Editors attached to a single Document (for example, a full-size editor plus an inline editor). Every Document is associated with a file on disk (accessible via `document.file`).
* `File` represents a file on disk. It's almost as lightweight as a plain string path: you can get a `File` object synchronously via `FileSystem.getFileForPath()`, without having to actually read or locate the file on disk yet.

## <a name="doc"></a>Working with Documents ##

`Document` is an object that represents an editable file on disk. Documents perform several important functions: they are the backing model for Editors; they provide APIs for reading and modifying the text content; and they emit events whenever the text is edited.

### How to get a Document ###

* If you're operating on an Editor, use `Editor.document`.<br>**How to get an Editor:**
    * `EditorManager.getFocusedEditor()` returns the editor that has focus; may be an inline editor. Will be null if focus is elsewhere, such as the search bar.
    * `EditorManager.getActiveEditor()` is similar, but if focus is somewhere other than an editor it returns whichever Editor _last_ had focus (i.e. the Editor that focus will return to when the search bar / dialog / etc. closes).
    * `EditorManager.getCurrentFullEditor()` returns the currently visible full-size editor (even focus currently lies in an inline editor within it, or if focus lies in other UI such as the search bar). Synonymous with using `DocumentManager.getCurrentDocument()`.
* To get a Document for any file, use `DocumentManager.getDocumentForPath()`. This returns asynchronously because it may need to read the file's content from disk.
* If you're sure a file is already open, you can use the synchronous `DocumentManager.getOpenDocumentForPath()` instead -- but it will return null if you're wrong.

### Proper Document usage ###

Documents are globally tracked, and thus _**must be ref counted**_ under certain circumstances.

> _Aside: Why is ref counting needed?_ We want all parts of Brackets to have a consistent view of the (potentially unsaved) contents of a file being edited.  Yet we also want to throw out that state when all parts of Brackets stop caring about a file, instead of keeping it in memory forever.  If JavaScript supported weak references they would be an ideal solution to these twin constraints, but alas -- we're stuck with ref counting instead.


**If you're only holding onto a Document for the duration of a function call** then you're in the clear and don't have to worry about this.  The text editing commands in EditorCommandHandlers are good examples of this.

If, on the other hand, you're...
* Storing a Document object in a property that you'll access later
* Getting a Document, doing something asynchronous, and then accessing the same Document pointer again when it's done
* Attaching event listeners to a Document

...then you'll need to be more careful. In all these cases, you must call `addRef()` on the Document after you fetch it, and then later call `releaseRef()` when you're done using it (e.g. when you null out / overwrite the property pointing to it, or detach your listeners).

The Document and its full text content will be kept in memory until you call `releaseRef()`, so it's better to avoid holding onto Documents if possible. In many cases, you can just store the file's path and re-fetch the Document each time you need it.

**If you're attaching event listeners to Document**:
* The Document "change" event fires on virtually every keystroke. For performance reasons, consider deferring your processing until later (for example, using the DocumentManager "documentSaved" event instead).
* If you're listening for Document changes, you probably also care about Document deletion -- so be sure to listen for the Document "deleted" event as well.

### Making edits ###

To modify a Document's text content, use `Document.replaceRange()`. If you're going to call it multiple times as the result of a single user action, wrap all your calls in `Document.batchOperation()` to ensure they're all batched into a single Undo/Redo entry.

In many cases, if you're implementing a new kind of edit, you'll want to handle multiple selections. See [Changes to Editor selection APIs](https://github.com/adobe/brackets/wiki/Brackets-CodeMirror-v4-Migration-Guide#changes-to-editor-selection-apis) and [Performing edits on multiple selections](https://github.com/adobe/brackets/wiki/Brackets-CodeMirror-v4-Migration-Guide#performing-edits-on-multiple-selections) for more information.

## <a name="commands"></a>Menus and Keyboard Shortcuts ##

Every menu item and shortcut invokes a `Command` -- basically just an object containing a handler function,  a label string, and some state such as enabled/disabled. `CommandManager` has a mapping from string "command ID" to Command object. `Menus` has a mapping from menu item to command ID, and `KeyBindingManager` has a mapping from key binding to command ID. All command IDs in core Brackets are listed as constants in `Commands`.

To **add a menu item or keyboard shortcut**, see [How to write extensions](How to write extensions#wiki-uihooks).

## Writing a New Inline Editor (Quick Edit) ##

See [How to write extensions](How to write extensions#wiki-featurehooks).

## <a name="fileio"></a>Read/Write Files ##

Normally, you'll want to use Document (see above) to read the contents of a file that the user might edit. If you modify a file via Document, it will automatically be recorded as an unsaved change for the user to track.

There are some cases where you may want to simply load a configuration file without treating it as a user document. For this, Brackets provides a [FileSystem API](http://brackets.io/docs/current/modules/filesystem/FileSystem.html) for direct access to local files. Here's how to simply read a file:
```javascript
// On Windows, paths look like "C:/foo/bar.txt"
// On Mac, paths look like "/foo/bar.txt"
var file = FileSystem.getFileForPath(localPath);

var promise = FileUtils.readAsText(file);  // completes asynchronously
promise.done(function (text) {
    console.log("The contents of the file are:\n" + text);
})
.fail(function (errorCode) {
    console.log("Error: " + errorCode);  // one of the FileSystemError constants
});
```

Note that paths in Brackets _always_ use "/" separators, regardless of OS. [Read more on path format](https://github.com/adobe/brackets/blob/master/src/filesystem/FileSystem.js#L43-L52).

## Accessing Node Modules from Brackets

See [Brackets Node Process: Overview for Developers](https://github.com/adobe/brackets/wiki/Brackets-Node-Process:-Overview-for-Developers).