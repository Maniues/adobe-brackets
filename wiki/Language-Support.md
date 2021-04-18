Starting with release 0.39, file extensions and file names can be mapped to languages via preferences. See [Preferences](#preferences) below.

Extensions can add basic language support &ndash; like syntax highlighting and comment toggling &ndash; via the [LanguageManager](https://github.com/adobe/brackets/blob/master/src/language/LanguageManager.js) API. This page explains how to use LanguageManager, and documents how language support in Brackets is currently implemented.

Some languages are built into core Brackets by default (for a list, see [languages.json](https://github.com/adobe/brackets/blob/master/src/language/languages.json)). A small set of these (HTML, JS, CSS) support richer editing features such as Quick Edit, Quick Find Definition, code hints, and Live Preview. _Some_ of those capabilities are extensible for new languages already (see [Extending specific Brackets features](https://github.com/adobe/brackets/wiki/How%20to%20write%20extensions#extending-specific-brackets-features)); others are not ready to support additional languages yet ([Language Support Changes](Language Support Changes) lists proposals for making the remaining rich functionality extensible).

## Preferences

If you have a file that is in a language that Brackets already has support for, either in core or in an extension, you can map additional file extensions and file names to that language. For example, imagine that you have a file called `pavement` that is a Python file and any file with a `foo` extension is a JavaScript file in your project. You can create a **.brackets.json** file in the root of your project that looks like this:

```json
{
    "language.fileExtensions": {
        "foo": "javascript"
    },
    "language.fileNames": {
        "pavement": "python"
    }
}
```

With that file in your project, or those preferences in your user-level brackets.json file, any time you open `something.foo`, Brackets will treat the file as JavaScript. If you open `pavement`, Brackets will treat the file as Python.

For these preferences, you map from the file name or extension to a language name as defined in [languages.json](https://github.com/adobe/brackets/blob/master/src/language/languages.json) or in an extension that provides support for the language you're interested in.

## Defining a new language

In an extension, if a language has an [existing CodeMirror mode](http://codemirror.net/doc/modes.html), you can declare the new language in a simple JSON object:

```javascript
var LanguageManager = brackets.getModule("language/LanguageManager");

LanguageManager.defineLanguage("haskell", {
    name: "Haskell",
    mode: "haskell",
    fileExtensions: ["hs"],
    blockComment: ["{-", "-}"],
    lineComment: ["--"]
});
```

#### Custom CodeMirror modes

If your language is not already supported by CodeMirror (see list above), you'll need to [write a new CodeMirror mode](http://codemirror.net/doc/manual.html#modeapi). To use your custom CodeMirror mode, load it using `require()`, register it with CodeMirror using [``CodeMirror.defineMode()``](http://codemirror.net/doc/manual.html#modeapi), then call ``LanguageManager.defineLanguage()`` as above. You can also use [``CodeMirror.defineSimpleMode()``](http://codemirror.net/demo/simplemode.html), which does not fit as many languages, but is a whole lot easier to write in the first place.


## Refining an existing language

An extension can also modify/enhance an existing registered language. Retrieve a Language object by its ID, then use setter methods to change it:

```javascript
var LanguageManager = brackets.getModule("language/LanguageManager");

var language = LanguageManager.getLanguage("coffeescript");

language.addFileExtension("cf");
language.addFileName("Cakefile");
language.setLineCommentSyntax(["#"]);
language.setBlockCommentSyntax("###", "###");
```

For further details, please refer to the comments in [LanguageManager.js](https://github.com/adobe/brackets/blob/master/src/language/LanguageManager.js).


## Using an existing language

### Retrieving a Language object

Each supported language is represented by exactly one Language object that can be retrieved in various ways:

* When defining a language: (returns the newly created Language object)
  `LanguageManager.defineLanguage(id, definition).done(function (language) { ... });`
* Using a language ID:
  `var language = LanguageManager.getLanguage("javascript");`
* Using a file path:
  `var language = LanguageManager.getLanguageForPath("/path/to/file.js");`
* Using a Document instance:
  `var language = doc.getLanguage();`
* Using an Editor instance*:
  `var language = editor.getLanguageForSelection();`
* Using another Language instance and a CodeMirror mode:
  `var language = otherLanguage.getLanguageForMode("xml");`

*The language in a selection may not be the same as the language for the document. For example, the ``html`` language supports ``css`` and ``javascript`` content.

Except for `LanguageManager.getLanguage`, these methods always return a language object. To determine whether this is the fallback language, use `if (language.getId() === "unknown") { ... }`.

### Using the Language object

A ``Language`` object contains model data for a language. The following methods are available:

* ``getId()`` returns the ID of a language (e.g. "cpp", "cs"). Use this to trigger language-specific behavior.
* ``getName()`` returns the human-readable name of a language (e.g. "C++", "C#"). Used by the status bar.
* ``getMode()`` returns the CodeMirror mode for a language. Used by EditorManager, should only be used in combination with CodeMirror specific code. Use ``getId()`` to identify a language.
* ``getFileExtensions()`` returns an array of file extensions for a language (e.g. "svg", "html.erb").
* ``getFileNames()`` returns an array of file names for a language (e.g. "Makefile", ".profile").
* ``hasLineCommentSyntax()/getLineCommentSyntax()/setLineCommentSyntax(prefix)`` returns line comment info. Used by the toggle line comment editor command.
* ``hasBlockCommentSyntax()/getBlockCommentPrefix()/getBlockCommentSuffix()/setLineCommentSyntax(prefix)`` returns block comment info. Used by the toggle block comment editor command.
* ``getLanguageForMode()`` returns either a language associated with the mode or the fallback language. Used to disambiguate modes used by multiple languages.



# Notes on "language" support (as of Sprint 21)

The remainder of this page documents what parts of language support are hardcoded and need to be refactored to allow extensibility, or require new capabilities before they can be refactored. See also [[Language Support Changes]].

These notes are based on the [LESS Refactoring](https://github.com/adobe/brackets/pull/2844) work.


### No changes required

These are okay the way they are.

* document/DocumentManager.js
    * `Document.getLanguage` uses the LanguageManager to determine the language based on the file extension.
* language/CSSUtils.js
    * Method `extractAllSelectors` extracts CSS selectors from a string. Internally uses CodeMirror's css mode as a parser, but that could be swapped out.
* language/JSLintUtils.js (now extensions/default/JSLint/main.js)
    * Uses JSLint internally which could be swapped out.
* language/JSUtils.js
    * Method `findAllMatchingFunctionsInText` to find all instances of a function name in a string of code. Internally uses CodeMirror's javascript mode as a parser, but that could be swapped out.
* language/LanguageManager.js
    * Defines the Language class used to represent a given language
    * Loads default languages from `language/languages.json`
    * Method ``defineLanguage(id, definition)`` to define and register a language. Returns a promise object that will be resolved with the Language object.
    * Method ``getLanguage(id)`` resolves a language ID to a Language object.
    * Method ``getLanguageForPath(path)`` resolves a file path to a Language object.
    * Used by extension "LESSSupport" to add basic support for LESS
* utils/StringUtils.js
    * Method `htmlEscape` escapes characters with special meaning in HTML. However, this function is necessary since the Brackets UI is written in HTML, and has nothing to do with language support for the users.


### Straightforward refactoring

These need to be changed to use existing functionality.

* __Done:__ brackets.js __requires language/JSLintUtils.js__. This can be refactored into an extension without introducing new APIs. See [issue #3094](https://github.com/adobe/brackets/issues/3094) and [pull request #3143](https://github.com/adobe/brackets/pull/3143).
* brackets.js __requires editor/CSSInlineEditor.js__. It should first call __require("editor/MultiRangeInlineEditor")__ (loaded by CSSInlineEditor.js), since this defines shortcuts for inline editor navigation. Then the CSSInlineEditor could be moved to an extension.
* editor/CodeHintManager.js
    * _In progress:_ Method `registerHintProvider` **registers hint providers by mode**. This can simply be changed to check for language IDs since currently all modes this function is being called with (either by Brackets or the known extensions) belong to a language with an equal ID ("css", "html", "javascript"). See [issue #3085](https://github.com/adobe/brackets/issues/3085) and [pull request #3270](https://github.com/adobe/brackets/pull/3270).
* __Done:__ editor/CSSInlineEditor.js
    * Inline editor provider `htmlToCSSProvider` **decides to open based on the editor mode**. This can simply be changed to check for the language ID to be "html" (via `editor.getLanguageForSelection().getId()`).
* project/FileIndexManager.js. See [pull request #3301](https://github.com/adobe/brackets/pull/3301).
    * Maintains an index called "css" using only files ending with ".css", i.e. **uses file extensions**. The call to add this index should be moved to CSSUtils (the only place this index is used at the moment). In addition, the filter function can be changed to use the language API: `return LanguageManager.getLanguageForPath(entry.name).getId() === "css";`
* __Done:__ language/JSLintUtils.js
    * Method `run` to run JSLint on the current document. Checks if the extension is .js, therefore **uses file extensions**. See [issue #3094](https://github.com/adobe/brackets/issues/3094) and [pull request #3143](https://github.com/adobe/brackets/pull/3143).
* language/JSUtils.js
    * Method `findMatchingFunctions` finds all functions with a specified name within a set of files. Filters these files by checking that the file extension is ".js", i.e. **uses file extensions**. This should use the language API instead (determine the language for the file and check whether that language has the ID "javascript").
* __Done:__ search/QuickOpen
    * Method `addQuickOpenPlugin` **uses file extensions** to register plugins. It should use language IDs instead. It is currently only used with file extensions "css", "js" and "html". For "css" and "html", the calling code can remain unchanged, transparently changing the meaning of the string from file extension to language ID. For "js", the calling code needs to use "javascript" instead. Currently `extensions/default/QuickOpenJavaScript/main.js` is the only place in either Brackets core or the extensions that uses this file extension. See [pull request #3301](https://github.com/adobe/brackets/pull/3301).


### Issues that should be addressed as part of other planned work

These are places that affect areas we already have plans to work on, and where issues are best addressed as part of that work.

* document/DocumentCommandHandlers.js
    * Method `_handleNewItemInProject` hardcodes ".js"/"Untitled.js" as the default file extension/name for new files. Changing this to work as proposed in [card #291](https://trello.com/c/WUdhIRlh) would remove this issue.
* language/{CSSUtils|HTMLUtils|JSUtils}.js should be provided by default extensions. For this to work, all other parts that depend on them (ideally only other extensions) need to be able to access these extensions. Supporting this is part of the [ongoing extensions research](https://github.com/adobe/brackets/wiki/Extensions2).
    * brackets.js __loads JSUtils.js__. This is only necessary so extensions can load it synchronously via `JSUtils = brackets.getModule("language/JSUtils")` instead of asynchronously via `brackets.getModule(["language/JSUtils"], function (JSUtils) { ... })`. This can be removed once JSUtils can be loaded as an extension.
    * brackets.js __exports CSSUtils and JSUtils for tests__. Tests should instead load these modules from extensions, but this depends on the point above.
    * editor/CSSInlineEditor.js __relies on HTMLUtils and CSSUtils__.
    * LiveDevelopment/Agents/DOMHelpers.js contains multiple methods that encapsulate knowledge about HTML, **should potentially be moved to HTMLUtils**
    * LiveDevelopment/Agents/DOMNode.js contains DOMNode.prototype.toString contains basic knowledge about HTML, **should potentially be HTMLUtils**
* utils/ExtensionUtils
    * Method `loadStyleSheet` **uses file extensions** to support LESS files. Once we have a compiler infrastructure in place, any path could be mapped to a language, and if there's a compiler to CSS for that language, it should be used. Note that is only relevant for extension developers.
* utils/TokenUtils
    * Method `getModeAt` **has a hardcoded special case for XML**. Once the other places are no longer based on this mode, but on the language, this can be removed to just report "xml". For HTML documents, the language manager maps the "xml" mode to the HTML language. XML documents are not affected by this. See [issue #2965](https://github.com/adobe/brackets/issues/2965) for a related discussion.


### Code that relies on the current editor state

These places currently access CodeMirror's state directly and are therefore not usable without an active editor. They might benefit from doing their own parsing, possibly using CodeMirror modes as parsers. CodeMirror's editor state could still optionally be used for optimization, but nothing else.

* editor/EditorCommandHandlers.js
    * Functions `_findCommentStart`, `_findCommentEnd` and `_findNextBlockComment` **use tokens provided by CodeMirror** to search for comment boundaries. While the strings they search are provided by a language definition, this prevents us from defining arbitrary comment symbols. One example is "//~" as the prefix for line comments (as [SciTE](http://www.scintilla.org/SciTE.html) does). Adding a comment this way is possible, but removing it does not work because "//~" is not a prefix of the CodeMirror token for "//".
    * Methods `blockCommentPrefixSuffix` and `lineCommentPrefixSuffix` have similar constraints as they navigate by tokens instead of characters. In addition they check whether a token's `className` is different from `"comment"`. Therefore they **use tokens provided by CodeMirror**.
* language/CSSUtils.js
    * Method `findMatchingRules` to find CSS rules matching a selector. Searches an HTML document via language/HTMLUtils __if it is the current full editor's document__.
    * Method `findSelectorAtDocumentPos` to find the selector(s) of a CSS block, directly **uses tokens provided by CodeMirror**
    * Method `getInfoAtPos` to provide a context info object for the given cursor position, directly **uses tokens provided by CodeMirror**
* language/HTMLUtils.js
    * Method `findStyleBlocks` to gather info about all `<style>` blocks in an HTML document, directly **uses tokens provided by CodeMirror** 


### Providing code semantics

This relates to places that require information about the semantics of code beyond what is provided by CodeMirror.

* editor/Editor.js
    * Method `_checkElectricChars` adjusts the current line's indentation when blocks are ended. **The characters to detect block boundaries are hard-coded - `]`, `{`, `}` and `)`.** The function is supposed to replace CodeMirror's own implementation, citing [bugs](https://github.com/adobe/brackets/pull/250). In contrast to `_checkElectricChars`, CodeMirror's own implementation does not re-indent the line after typing "]" in a JavaScript file, so for now we cannot remove our own implementation without removing existing functionality.


### Starting Live Preview

These areas are concerned with what files live preview can be started with. This also affects Brackets' behavior when switching between files while live preview is active.

* file/FileUtils.js
    * Methods `isStaticHtmlFileExt` and `isServerHtmlFileExt` __use hardcoded lists of file extensions__ to determine which file extensions are okay to open statically and which require a base URL to work. Switching to a file matching these criteria causes live preview to show that file instead unless it is included in the currently displayed file.
* LiveDevelopment/LiveDevelopment.js
    * Method `open` **reduces LiveDevelopment support to HTML files based on file extensions**
    * Function `_onDocumentChange` **closes LiveDevelopment when switching to a different, not included HTML file**


### Updating Live Preview

These areas are concerned with making sure that the live preview is up to date. This is needed if the main document or included files are changed.

* LiveDevelopment/LiveDevelopment.js
    * **Requires hardcoded list of special documents**, namely {CSS,HTML,JS}Document. These function as updaters and highlighters.
    * Function `_classForDocument` **uses a hardcoded mapping of file extensions to document types**
    * Function `_openDocument` **only loads related CSS documents** (no JavaScript, not extensible)
    * Function `_onLoad` **excludes CSSDocument from being marked as out of sync** (not extensible)
    * Function `_onDocumentChange` **excludes CSSDocument from being marked as out of sync** (not extensible)
    * Function `_onDocumentSaved` **only reloads the page if the document is not a CSSDocument** (not extensible)
    * Function `_onDirtyFlagChange` **only updates the LiveDevelopment status if the dirty file is not a CSSDocument** (not extensible)


### Showing the context in Live Preview

These areas are concerned with showing the context of the current cursor position by highlighting affected areas in the live preview and opening files related to elements in the page (GotoAgent).

* LiveDevelopment/LiveDevelopment.js
    * **Requires hardcoded list of special documents**, namely {CSS,HTML,JS}Document. These function as updaters and highlighters.
    * Function `_classForDocument` **uses a hardcoded mapping of file extensions to document types**
    * Function `_openDocument` **only loads related CSS documents** (no JavaScript, not extensible)
    * Method `showHighlight` **only calls doc.updateHighlight() for CSSDocuments**
* LiveDevelopment/Agents/GotoAgent.js
    * **Has hardcoded support for HTML, CSS and JavaScript**
* LiveDevelopment/Agents/HighlightAgent
    * **Has hardcoded support for HTML and CSS**
* LiveDevelopment/Agents/RemoteFunctions.js
    * Function `_typeColor` has **hardcoded distinctions between html, css, js and others**