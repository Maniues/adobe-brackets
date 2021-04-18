## Tern.JS Integration 
The JavaScript Code Hints feature is implemented as a Brackets extension. For hinting, it relies on the [TernJS Library](http://www.ternjs.net).  

The extension manages an instance of the Tern server, which runs in a web-worker, and makes requests to it whenever it needs information to calculate hints.  The information requested includes function types, types of the base of a member expression, and potential property or identifier names.  The extension caches most of the information it requests so that it does not need to continuously requery tern during the same code hinting session.

When the extension starts up, it reads in some .json files that provide type information for the environment.  Currently it reads in ecma5.json, which contains type info for the javascript builtins, and browser.json, which provides type information for the methods that the browser exposes.  These files are only parsed once, as they should never change.  When the tern server is initialized, the environment built from those files is passed to tern to initialize its environment.  To support different modes of development (e.g. editing a node.js file vs. a browser based file) we could supply tern with different environments to get the most appropriate hints.

The hints are calculated based on the files in the current directory, and on the calls to requirejs seen in the file.

## API Proposals and Guesses
Hints are displayed as the user is typing or explicitly by the Ctrl+space key. Three different kinds of hints are available: property hint, identifier hint, and function hint. If the cursor is following a "." then property hints are displayed. If the cursor is inside function parentheses, then the function's argument types and return type are displayed. Otherwise all known identifiers, classes and top-level functions are hinted.

**Property hints:**
* When the type of the object is known, the properties for that type are displayed.
* When the type of the object is unknown or there are no more matching entries in the objects's properties, guesses are shown. Guesses are shown in italics.

**Identifier hints:**

Identifier hints contains the list of identifiers following by literals, and keywords.

* Identifiers - Known identifiers, classnames, and functions. Sorted by depth – current scope depth shown in green, the next outer scope in yellow, then blue, and the rest in black.
* Literals - "true", "false", "null", "undefined".
* Keywords – sorted alphabetically, displayed in monospaced font.

**Function hint:**

Displays the functions parameter types and the return type.

**String Literals:**

Currently we do not hint string literals.

**Interaction:**

As the user types, the lists of choices in the hints pop-up is narrowed. All the character matches are case-insensitive. Character matching and sorting is handled by the [StringMatcher](https://github.com/adobe/brackets/blob/master/src/utils/StringMatch.js) utility. It has an advanced character matching algorithm and sorts the best matches to the top.

The code that filters and sorts the hints is in the getHints() method in [Session.js](https://github.com/adobe/brackets/blob/master/src/extensions/default/JavaScriptCodeHints/Session.js). The results are returned to the getHints() in [main.js](https://github.com/adobe/brackets/blob/master/src/extensions/default/JavaScriptCodeHints/main.js) where they are formatted. Formatting is done by adding CSS class names to hint objects. The CSS classes are defined in [brackets-js-hints.css](https://github.com/adobe/brackets/blob/master/src/extensions/default/JavaScriptCodeHints/styles/brackets-js-hints.css)

## Jump-To-Definition (of current identifier)
Jump-To-Definition allows the user to have the cursor move to the definition of the variable or function to which the cursor currently points.  This works for definitions within the current file and within all other open files.  Definitions are selected/highlighted when found, with the cursor positioned at the end of the selection.

**Interaction:**

This feature is controlled by the Navigation Menu -> Jump to Definition (control-J).

##Initializing a hinting context on file open

If the opened file had already been added to Tern then no new files need to be read in. If the previous file has been modified (does not need to be saved) then the contents of the new file will be updated in Tern.

When a JavaScript or HTML file is opened in Brackets the JS Code Hinter reads additional files and adds those to Tern to be analyzed. All of the JS files in the directory of the opened file are added to Tern. More files may be added depending on if the opened file uses requirejs. This directory of the initial file is considered the root directory of the JS Code Hinter.

After the files in the current directory are added to Tern, the opened file is analyzed. If the opened file uses requirejs to create dependencies, all of the dependent files will be added to Tern.

If the opened file does not have dependencies on other files, it is assumed that more files will be needed to provide a better context for hinting. First the JS files in the subdirectories of the opened file will be added to Tern to provide more context. If the opened file is in a subdirectory of the Bracket's project root, then JS files from the project's root directory will be added to Tern as well. 

The JS Code Hinter limits the number files that can be added to Tern at 100.

## Configuration

Very large and complicated JavaScript files can cause performance issues. A JS Code Hints configuration file provides the ability to work around these issues. When Brackets opens a new project root, if a configuration file is found, the settings are used to control loading files for code hinting. Configuration files are only updated when changing the project root. Opening new files will not cause a search for a new configuration file. The configuration file is a JSON formatted file named “.jscodehints”.

The following properties are supported:

* **excluded-directories**
An array of strings or regular expressions that match directories relative to the project root. Matching directories will be excluded from analysis. Directories may be excluded if they contain automated tests that aren’t relevant for code hinting. The default value is an empty array. Filter values are matched against the _project-relative_ path of the directory, excluding trailing slash. Two types of filter values are supported:
    * A simple string, which may contain the wildcards `*` and `?`. (Note: unlike [search file exclusions](Using File Filters), `*` here matches all characters _including_ path separators). This string must match the _entire_ project-relative path in order to exclude the directory, so you may need to add leading/trailing `*`s.
    * A regular expression literal embedded in a string (wrapped in `/`s), e.g. `"/thirdparty/mylibrary-1\\.\\d/"` (note the double-escaping due to the regexp being inside a JSON string literal). This need not match the entire project-relative path, unless you manually add `^` and `$` to it.

* **excluded-files**  
An array of strings or regular expressions that match files that will be excluded from analysis. Files are typically excluded because their API is in a JSON file or they are known to cause problems with either stability or performance. Brackets always excludes ["require.js", "jquery*.js", "less*.min.js", "ember*.js", ".*"]. Any settings you apply will be _in addition to_ these defaults.
    * A simple string, which may contain the wildcards `*` and `?`. This string must match the _entire_ filename including extension.
    * A regular expression literal embedded in a string (wrapped in `/`s), e.g. `"/mylibrary-1\\.\\d\\.js/"` (note the double-escaping due to the regexp being inside a JSON string literal). This need not match the entire filename, unless you manually add `^` and `$` to it.

* **max-file-count**   
Limits the total number of files that can be processed for hints. This limit only applies when an opened file does not use "require" and files under the project root are being added to the hinting context. The default value is 100.

* **max-file-size**  	
Files larger than this number of bytes will not be parsed. The default value is 524,288 bytes.

The strings in "excluded-directories" or "excluded-files" will be treated as a regular expression if the first and last characters of the string are the '/' character. Note the '\' character in a regular expression needs to be escaped to be valid in a JSON formatted file. For example "/[\d]/" becomes "/[\\\\d]/".

**Example file:**

    {               
         "excluded-directories" : ["excluded-dir", "/tests-.*files/"],  
         "excluded-files" : ["require.js", "jquery*.js", "/lib-1.[12].js/"],  
         "max-file-count": 100,   
         "max-file-size": 524288  
    }

## Advanced Hinting
JS Function Parameter and Documentation Hints

In addition to the current hints, the JS code hints will provide independent windows for function parameter hints and documentation hints. The look and feel of the three windows will be similar to the Eclipse implementation. The function parameter hints will be displayed above the current line; the main hints below the current line, and documentation on a highlighted hint will pop out to the side of the main hint window.

 

The function parameter hints will be displayed after the user chooses a function from the hints popup, inserting the function text and dismissing the main hint window. Function parameter hints can be re-displayed anytime the cursor is inside a function by pressing Shift+Ctrl+Space. The parameters will be displayed in a format similar to closure annotation but without enclosing brackets around the type annotation. Optional parameters will be enclosed in brackets.

The main hints themselves will be unchanged from the current format. 

Documentation on a function or variable will pop out to the side of the main hint window when a hint is highlighted. A small delay between highlighting a hint and displaying the documentation will prevent flash in the case where the user is quickly moving through the hints. The documentation for functions will contain the function signature, origin, description, parameters, and return type. The documentation window for variables and properties will show the type, origin, and description. The figure below shows a documentation hint for a highlighted function.

## JavaScript File Inference Problem

In Release 0.41, Brackets upgraded to Tern 0.62 and turned on the timeout feature.
This allows Brackets to stop runaway file processing that would cause Brackets to hang or crash.
After files hit the timeout, they are stopped and not tried again for current
project session.

In Release 0.42, Brackets now displays a modal dialog warning when this is detected
and adds the file to the `jscodehints.detectedExclusions` array in `.brackets.json`
project preferences file in the project root folder. If the current project does
not already have a project preferences file, then one is created. Files are no longer
processed for hint information until they are removed from this array.

You can try pasting the code into the [Tern demo page](http://ternjs.net/doc/demo.html) to see if it passes there. Note that the version of Tern running in that demo may be newer than the version of Tern running in Brackets.

The number of milliseconds of the timeout can be set using the `jscodehints.inferenceTimeout`
preference. <del>The default timeout is 10 seconds (10000).</del> Starting with Release 1.0, The default timeout is 30 seconds (30000).

# Refactoring Project (spring 2014)

Contact: dangoor

Two significant features with an impact on JS Code Hints shipped in Brackets 36: file watchers and preferences. As a result, we've chosen this time to [refactor the JS code hints code](https://trello.com/c/BMVw25vU/75-research-js-code-hints-cleanup) in order to improve the overall performance of code hints.

## Breakdown of Code Hints issues

### Crashes

* [Brackets grey screens and crashes when editing JS files](https://github.com/adobe/brackets/issues/7514)
* [Mac OSX - Freeze - Grey Screen](https://github.com/adobe/brackets/issues/7308)
* [Entire window goes blank](https://github.com/adobe/brackets/issues/7262)
* [Brackets crashes](https://github.com/adobe/brackets/issues/7025)
* [CPU stays at 100% with specific project](https://github.com/adobe/brackets/issues/7245)

### Hints not behaving as expected

* [Suboptimal behaviour of Quick Edit](https://github.com/adobe/brackets/issues/7003)
* [Missing intellisense on too big file](https://github.com/adobe/brackets/issues/6986)
* [No code hints for "this" when superclass is unknown](https://github.com/adobe/brackets/issues/6929)
* [Requires don't work if extension missing in filename (node)](https://github.com/adobe/brackets/issues/5993)
* [Display redefined variables hints as guesses](https://github.com/adobe/brackets/issues/5729)
* [JS code hints aren't picking up new items added to exports](https://github.com/adobe/brackets/issues/4991)
* [code hints are affected by a javascript loading a module with require](https://github.com/adobe/brackets/issues/4192)
* [jump to definition inside quick editor on a function not working](https://github.com/adobe/brackets/issues/3951)
* [quick editor not open when function name has non ascii chars](https://github.com/adobe/brackets/issues/3941)
* [Support require() calls not in an AMD wrapper](https://github.com/adobe/brackets/issues/3801)
* [Incorrect jQuery hints when switching to a $. from $(something)](https://github.com/adobe/brackets/issues/3685)
* [Should not show hints after typing a dot with no function call before it](https://github.com/adobe/brackets/issues/3682)
* [include JSLint-defined globals story](https://trello.com/c/AOEjxuDl/1011-js-code-hints-should-include-jslint-defined-globals)
* [Code hints for require() path strings story](https://trello.com/c/0HWEDXGE/997-code-hints-for-require-path-strings)
* [Improved code hints for extension authors story](https://trello.com/c/xBlcCYju/990-improved-code-hints-for-extension-authors)
* [Better formatting for optional arguments story](https://trello.com/c/jm6sMvdz/955-js-code-hints-better-formatting-for-optional-arguments)

**Problems in Tern**

* [Inconsistent JS hints depending on whitespace](https://github.com/adobe/brackets/issues/5263)
* [Object .toString / toJson / toValue](https://github.com/adobe/brackets/issues/4183)
* [no code hint for jQuery.Event in on( events, handler(eventObject))](https://github.com/adobe/brackets/issues/4181)
* [parameter type later becomes variable name when passing object as param to a function](https://github.com/adobe/brackets/issues/3838)
* [Incorrect function return type for jQuery setter functions](https://github.com/adobe/brackets/issues/3684)
* [code hints in inherited class by Class.create not showing, later use guess hint causes existing hint changed](https://github.com/adobe/brackets/issues/3660) – this appears to be Prototype library specific. I would probably call this "no priority" or even just close it.
* [console.assert() missing from code hints](https://github.com/adobe/brackets/issues/3655)

### Performance

* [make jump to definition response faster (on large JS framework file)](https://github.com/adobe/brackets/issues/4066)
* [Apparent performance test regression in JS Quick Edit when Tern doesn't find results](https://github.com/adobe/brackets/issues/3961)
* [JS Code Hints File Handling Revamp story](https://trello.com/c/VjAiuH31/1070-js-code-hints-file-handling-revamp)

### StringMatch

* [Code hints should filter out unlikely matches](https://github.com/adobe/brackets/issues/5993)
* [Prefer results that match case of the query string](https://github.com/adobe/brackets/issues/3971)

### Code Cleanup

* [ScopeManager is inconsistent about path endings](https://github.com/adobe/brackets/issues/5529)

### Unit tests

* [Unit tests for 6931, referencing members of a class](https://github.com/adobe/brackets/issues/7152)
* [Console errors when running unit tests](https://github.com/adobe/brackets/issues/6937)
* [Intermittent test failure](https://github.com/adobe/brackets/issues/7646)
* [Automated testing for JS code hinting with various frameworks/projects](https://trello.com/c/gqdpxfM2/937-automated-testing-for-js-code-hinting-with-various-frameworks-projects)

### Preferences Handling

* [JS code hint exclusions ignored sometimes](https://github.com/adobe/brackets/issues/7342)
* [Ability to turn off code hinting](https://github.com/adobe/brackets/issues/4716)
* [max-file-count takes less files to be processed for hints](https://github.com/adobe/brackets/issues/4195)
* [excluded-files preference does not support directory names in file path](https://github.com/adobe/brackets/issues/4191)
* [when open an excluded file is opened methods and properties not excluded](https://github.com/adobe/brackets/issues/4190)
* [Understand require configuration story](https://trello.com/c/mq7dZnlv/1049-js-code-hints-understand-require-configuration)
* [JS Code Hint preferences story](https://trello.com/c/BOgGIzWW/1046-js-code-hint-preferences)

### Other

* [JavaScript Jump To Definition Ctrl+J/Cmd+J not mentioned in Right-Click Menu](https://github.com/adobe/brackets/issues/3860)

# The Rework

These were the goals I had going in to the rework investigation, with most important on top:

1. Better stability
2. Better performance
3. Easier testability with more predictability
4. Improved results
5. Start integrating into a reusable, extensible project model
6. Less code

The results are on the [JavaScript Code Hints Cleanup](https://trello.com/c/cNVSkHPV/1235-javascript-code-hints-cleanup) card.

The lowest priority two considerations have mostly fallen out of scope. After reviewing the code that is in the extension now, I don't think it's carrying much in the way of "excess baggage", beyond the preferences system. So, "less code" ends up largely being about removing its preferences system. The reusable, extensible project model requires some new infrastructure and this does not strike me as the project during which we should start that infrastructure. Some of the other needs are just too urgent and are not directly helped by the new model.

* Better stability will come from automatic detection of files that make Tern spin out of control.
* Better performance will come from trying to avoid killing the worker every 30 hints (partly enabled by file watchers) and trying to avoid reading/parsing/inferring the same JS file more than once
* Better testability will come from refactoring ScopeManager to clarify the line between it and Session. This should make tests much faster. Making it possible to run Tern on the main thread can make certain problems easier to track down.
* Improved results will come from attempting to remove limits (the arbitrary 2,000 line limit for files) that may have just been there because of the edge cases that were problematic for Tern

Finally, once all of the main work is done, we should do a bug squash in which we go back through the list of bugs do a final round of fixes that will likely be aided by the improved testability.