<a id="howtoget"></a> Getting Brackets
-----------
**[Downloads Brackets here](http://brackets.io)** for Mac, Windows and Linux (Debian/Ubuntu). Brackets is built with HTML, CSS and JS, but currently runs as a desktop application in a thin native shell that can access your local files.

Updates are released about once a month.


The Basics
----------
Initially, Brackets opens a default "Getting Started" project. Follow the instructions in the HTML code for a quick walkthrough of Brackets features.

You can open a different folder in the file tree on the left using **File > Open Folder**. Brackets considers this folder your "project": it acts as the scope for various search operations, and some settings are tied to the folder you have open. You can easily switch back to previous projects by clicking on root folder name in the file tree. You can also drag a folder from the OS onto the Brackets window to open it in the file tree, and drag files onto the Brackets window to open them.

Unlike other editors that show open files in tabs, Brackets has a **"Working Files"** list, which is displayed above the file tree. Clicking a file in the file tree just views it, but doesn't add it to the Working Files list — so you can quickly browse through different files without cluttering the list. If you make an edit, the file is automatically added to Working Files. To add a file without editing it, double-click it in the file tree.

### Split View
Initially Brackets will show just one editor in the main view but you can split the main view so that 2 editors can be shown in whichever orientation you prefer (vertical or horizontal).  

You can do this by selecting **View > Horizontal Split** or **View > Vertical Split**. This splits the main view into 2 panes so that 2 files can be shown at the same time.  Splitting the main view also creates a second **"Working Files"** list, which shows which files are open in which pane.  You can drag a file between the two **"Working Files"** lists to move it to the opposite pane.

Brackets will remember the view layout for each project so switching to another project will show the layout you had chosen when the project was closed. And, if you'd prefer to go back to just a single view, you can select **View > No Split** to go back to a single view. Doing so does not close the files currently opened. Brackets will just merge the two **"Working Files"** lists and keep your changes in memory until you're ready to save them.

Brackets doesn't support opening a file in both panes but we are planning to ship that in a future release.

### Extensions
In addition to the core features built into Brackets, there is a large and growing community of developers building extensions that add all sorts of useful functionality. You can search for and install/uninstall extensions using **File > Extension Manager...** (or click the "plugin block" icon on the toolbar). You can even temporarily disable extensions and re-enable them later from the Extension Manager.

You can also [browse the available extensions online](https://brackets-registry.aboutweb.com) without installing Brackets first.

### Themes
You can change the color scheme of the editor by downloading a theme from via the Extension Manager (search for "theme" in the Extension Manager to find them). You can even [create your own custom theme](https://github.com/adobe/brackets/wiki/Creating+Themes) to get editor colors that suit your personal taste.


Brackets Highlights
-------------------
### <a id="inlines"></a>Quick Edit
Instead of cluttering up your coding environment with lots of panels and icons, the Quick Edit UI in Brackets puts context-specific code and tools inline.

You open Quick Edit's inline editors by pressing **Ctrl/Cmd-E** when your cursor is on certain pieces of code. For example:

* In an HTML file with the cursor inside a `class` or `id` attribute (name or value) or on the tag name — Quick Edit will show you all the CSS, SCSS and LESS rules in your project that match. You can edit these rules directly inline, without ever leaving the context of the HTML file.
    * When multiple rules match, navigate among them using the list on the right side (or use Alt-Up/Down).
    * To create a new CSS rule directly from the inline editor, click the **New Rule** button (or press Ctrl-Alt-N/Cmd-Opt-N).
* In any file with a hex color or rgb/rgba/hsl/hsla color — Quick Edit opens an inline color picker for previewing and adjusting the color. The color picker also lists the top most-used colors from other parts of the file for quick access.
* In a JavaScript file with the cursor on a function name — Quick Edit will show you the function's body (even if it's in a different file).
* In a CSS/LESS/SCSS file with the cursor on a `cubic-bezier()` or `steps()` transition timing function — Quick Edit opens a graphical transition curve editor. Pre-defined timing functions `ease`, `ease-in`, `ease-out`, `ease-in-out`, `step-start`, and `step-end` are also valid starting points.

**Quick Docs** is a related feature that displays relevant documentation inline. Use **Ctrl/Cmd-K** to open Quick Docs:

* In a CSS/LESS/SCSS file with the cursor on a CSS property/value — Quick Docs opens documentation from the [Web Platform Docs](http://docs.webplatform.org/) project.

You can open _multiple_ inline editors and docs viewers simultaneously. To close a single inline editor or docs viewer, click the "X" in the upper-left or press Escape while it has focus. To close _all_ inline editors & docs at once, place your cursor back in the main enclosing code editor and press Escape.


### <a id="livedev"></a>Live Preview
Brackets works directly with your browser to push code edits instantly, so your browser preview is always up to date while you're coding — no page reloads needed. In order to keep your current web browsing unaffected, Brackets Live Preview opens an *additional copy of Chrome* using a separate Chrome profile.

There are two different ways to use Live Preview:

**With no backend (i.e. server-side) logic** — Open an HTML file and select **File > Live Preview** (or click the "lightning bolt" icon). Brackets will launch Chrome and open your file in a new tab. The content is served statically from a built-in server that Brackets runs — it doesn't contain any of your app's backend logic.

This mode offers the full range of Live Preview functionality:
* Browser preview updates in real time as you type in HTML _and_ CSS files (without reloading)
* File extensions recognized for HTML files are: htm, html, and xhtml.
* If you edit any other type of file, the page is auto-reloaded when you save
* When you move the cursor around an HTML file, the corresponding element is highlighted in the browser
* When you move the cursor around a CSS/LESS/SCSS file, all elements matching the current rule are highlighted in the browser
* (All cursor-driven highlighting can be disabled by unchecking **View > Live Preview Highlight**)

All the CSS features above also work when you're in an inline Quick Edit CSS editor.

**Using your own backend**<a name="lp-custom-server"></a> — Make sure your local server is already running, serving files from the same folder Brackets is editing. Choose **File > Project Settings** and enter whatever URL corresponds to the _root_ folder that's open in Brackets (typically a localhost URL). Then open a file for one of your webpages (e.g. an HTML, PHP, or ASP file) and launch Live Preview. Brackets will launch Chrome with the correct URL to load that page from your local server.

_However_, Live Preview has the following limitations when using your own backend:

* The browser won't update immediately as you type in server-processed files (such as HTML or PHP) — only changes to CSS files will be reflected in real time. For server-processed files, Brackets will automatically reload the page on save to update the browser preview.
* "Live Highlighting" is disabled for server-processed files. It will still work when your cursor is in a CSS file, however.

> Why do these limitations exist? To enable HTML live editing, Brackets needs to inject some annotations into your HTML code before the browser loads it. Normally, the built-in Brackets server does this. When using your own server instead, Brackets can't inject those annotations. Without the annotations, Brackets can't map edits and cursor positions from your source file onto the corresponding DOM nodes in the browser.

_Note:_ The "Experimental Live Preview" mode (aka multi-browser Live Preview) does not support using a custom server. The reasons are similar to the explanation above (additional detail [here](https://github.com/adobe/brackets/issues/10374#issuecomment-78398635)).

##### Live Preview with SCSS/LESS

Live Preview won't update in real time as you type in LESS/SCSS files. However, if you use a third-party "file watcher" to automatically recompile your CSS on save, Live Preview will automatically update on save to reflect the changed CSS file (without reloading). You can also use a Brackets extension such as [Brackets SASS](https://github.com/jasonsanjose/brackets-sass) or [LESS AutoCompile](https://github.com/jdiehl/brackets-less-autocompile) for this. However — if you're using less.js to dynamically compile your LESS at runtime, Live Preview won't be able to update the page; you'll need to reload to see changes.

##### Other limitations

Live Preview currently has a few other important limitations:

* It only works with desktop Chrome as the target browser.
* Opening the Developer Tools in Chrome will close the live development connection.
* Files must be inside your "project" (the root folder you currently have open in Brackets).
* Only one HTML file can be previewed at a time. If you switch to a different HTML file in Brackets, the browser preview will switch to that new page as well.
* Updating pauses when the HTML is syntactically invalid (e.g. after you type '<' for a new tag but before you type the closing '>'). The line number and Live Preview icon turn red in this case. Brackets will resume pushing changes to the browser when syntax becomes valid again.

See **[Live Preview troubleshooting](https://github.com/adobe/brackets/wiki/Troubleshooting#livedev
)** for additional help.

##### Experimental Live Preview
 Experimental multi-browser live preview can be enabled/disabled Under **File > Enable Experimental Live Preview** menu. This addresses some of the limitations of current Live Preview. 
* Target multiple browsers and preview on them simultaniously.
* Developer Tools in chrome can be opened without interrupting the Live Preview workflow.

### <a id="quickview"></a>Quick View
Quick View makes it easy to visualize assets and colors in your code. Just hover your
mouse over a color, gradient, or image reference, and a popover will appear showing
a preview. You can disable this feature in the View menu.


Other Features
--------------

### Command Line Integration

Brackets can be launched from command line- using `brackets <file/folder/nothing>` on Windows and Mac. On Windows, you can even right-click on a file/folder from Windows Explorer and select "Open With Brackets"/"Open as Brackets Project" respectively to launch Brackets.

### Code Folding

With Brackets, you can expand/collapse blocks of code using indicators next to the line numbers, or via keyboard shortcuts.

### Multiple Selections

Brackets supports multiple cursors, multiple selections, and rectangular selections, as well as Undo Selection and useful commands like Add Next Match to Selection. See [[Working with Multiple Selections]] for more information.

### Settings
* **Indentation and tabs** — To change the default indentation for the editor, use the controls on the right end of the status bar at the bottom of the window. Click the word "Spaces" or "Tab Size" to switch whether you're using spaces or tabs, and change the indentation size by clicking on the number to the right. Note that Brackets uses "soft tabs", so even if spaces are inserted, the cursor moves as if tabs are present.
* **Editor font and colors** — There are no official preferences for these yet. However, there are unofficial ways to [[Customize your code font]], and several extensions add the ability to choose different themes.

See also the [Preferences](#preferences) section below.

### Quick Open
To quickly jump to a file, press **Ctrl/Cmd-Shift-O** and type part of the filename. You can type abbreviations or other non-contiguous parts of the name, and Quick Open will intelligently find the best matching file.

### Quick Find Definition
To quickly jump around _within_ a file, press **Ctrl/Cmd-T** to see an outline view — functions in a JS file, selectors in a CSS/LESS/SCSS file, etc. Similar to Quick Open, you can type parts of a name to filter the list.

### Instant Find in Files
The updated Find In Files feature- Instant Search shows results as you type. To start a search, press **Ctrl/Cmd + Shift + F** and start entering your query string. You see the matches almost instantly in the results panel as you type in the search bar. You can refine the search in files using case-sensitive and regex searches. You could even make the search in a specific folder(right click a folder in the file tree and select `Find in`) or exclude files based on exclusion sets(Press `ctrl/cmd + shift + f`and click on the exclusion set dropdown).

### <a id="codehints"></a>Code Hints
Code hints generally pop up automatically while you're typing, but you can also manually display them with **Ctrl-Space** (note that this shortcut uses Ctrl even on Mac).

Code hints are provided by default in a number of places:
* HTML — tag names, attribute names, attribute values, and `&` entities.
* CSS, LESS, SCSS — all property names, and enumerated property values (those where the value is a discrete list of keywords).
    * Code hints don't yet work for shorthand properties (e.g. `background`), only for individual properties (e.g. `background-repeat`).
* JS — variables and functions, using the [Tern](http://ternjs.net/) code intelligence engine.
    * Tern makes intelligent inferences about what properties and methods a given object contains, based on an analysis of your code. In addition to the current file, Brackets looks at other files in the same folder and any files referenced by a `require()` statement.
    * In cases where Brackets can't determine exactly what hints should be available, it will fall back to a list of heuristic guesses. These guesses are shown in italics.
    * JS code hints use smart matching — so you can type camel-case initials and other shorthand to filter the hint list more quickly (e.g. type "gsp" for `getScrollPos`).
    * You also get _argument hints_ — while you're typing arguments to a function, an indicator above the cursor lists the expected types of the arguments. Normally this appears automatically, but you can also display it manually by pressing **Ctrl-Shift-Space**. (Nothing is shown if Tern is unsure where the function is defined, however).

### ESLint

By default, Brackets runs [ESLint](https://eslint.org/) on JavaScript files when you initially open them and whenever you save changes. If ESLint finds problems, the results are shown in a panel at the bottom. If your file is clean, you'll see a green checkmark in the status bar instead.

See all the configuration options available at [https://eslint.org/docs/user-guide/configuring](https://eslint.org/docs/user-guide/configuring]. You can hide the ESLint results panel by clicking the close box at the top (the status bar icon will still indicate if ESLint has found problems with either a green checkmark or yellow warning symbol), or you can turn off ESLint completely by unchecking **View > Lint Files on Save**.

### Brackets Health Report

Anonymous data to help improve Brackets. You can preview the data that will be sent, or opt-out if desired. We've gone to great lengths to protect your privacy and maintain transparency - see **Help > Health Report** for details.

Keyboard Shortcut Cheat Sheet
-----------------------------
Here are some keyboard shortcuts that are worth knowing. Also see the
[Brackets Shortcut wiki page](https://github.com/adobe/brackets/wiki/Brackets-Shortcuts)
for a more complete list of shortcuts and the [User Key Bindings Wiki](https://github.com/adobe/brackets/wiki/User-Key-Bindings) to learn how to setup Brackets to use your preferred keyboard shortcuts.

| Windows | Mac | Description |
|---------|-----|-------------|
| Ctrl-E | Cmd-E | Open/close the inline editor (Quick Edit) |
| Alt-Up/Down | Alt-Up/Down | Switch between rules in the inline editor |
| Ctrl-K | Cmd-K | Open Quick Docs |
| Ctrl-Space | Ctrl-Space | Bring up code hints, if applicable |
| Ctrl-Shift-O | Cmd-Shift-O | Bring up the Quick Open prompt |
| Ctrl-G | Cmd-L | Go to a line in the current file |
| Ctrl-T | Cmd-T | Go to a method/selector in the current file (Quick Find) |
| Ctrl-Shift-H | Cmd-Shift-H | Show/hide the sidebar |
| Ctrl-Alt-P | Cmd-Alt-P | Live preview |


<a id="preferences"></a>Preferences
-----------

There is not yet a global user interface for all preferences (so the required "Preferences" menu item on Mac is disabled). You can change a few preferences from the Brackets user interface - such as word wrap or the current color theme. But many more preferences are available via JSON configuration files. To modify these preferences, first choose how globally you want to set the preference:

#### Scope of preferences

* **User-global preferences** - your default settings. To edit, choose **Debug > Open Preferences File**.
* **Project-level preferences** - override any number of preferences on a project-specific basis. Create a `.brackets.json` file in the root of your project.

Choosing **Debug > Open Preferences File** opens the global preferences file "brackets.json" along with the default preferences in left panel in split view. You can refer to the available settings in the left panel and change the preferences in the right. Code hints is available for the settings of this file.
Changing a preference using the Brackets UI modifies the project's `.brackets.json` file _if_ the preference is already set there; otherwise it modifies the user-global setting.

Within either file, there are three levels of specificity at which you can set a preference:

* default - global (user-level file) or project-global (project-level file)
* `"path"` layer - overrides in effect for files that match the given path/filename wildcard
* `"language"` layer - overrides in effect for files that Brackets detects as the given programming language (this is also filename/extension based, but it's easier to work with since Brackets already understands many file extensions out of the box, and additional languages supported by Brackets extensions can automatically be used here too).

See the sample .json file below for examples of how to use the path/language-specific preferences.

#### List of supported preferences

These are all the settings that are currently supported:

| Setting | Default | Description |
|---------|---------|-------------|
| `closeBrackets` | `false` | Automatically close braces, brackets and parentheses |
| `closeOthers.above`,<br>`closeOthers.below`,<br>`closeOthers.others` | all `true` | False to remove the Close Others, Close Others Above, Close Others Below items from the Working Files context menu |
| `closeTags` | `{"whenOpening": true, "whenClosing": true, "indentTags": []}` | Sets the tag closing options. See the [CodeMirror documentation](http://codemirror.net/addon/edit/closetag.js). |
| `code-folding`&#8203;`.alwaysUseIndentFold` | `false` | _Since 1.3._ Always generate collapsible section markers when indent level changes (for > `minFoldSize` lines), including in plain text files |
| `code-folding.enabled` | `true` | _Since 1.3._ True to show collapsible section indicators in the gutter. Disabling code folding also hides its View menu items |
| `code-folding`&#8203;`.hideUntilMouseover` | `false` | _Since 1.3_ Section-collapsing markers are only visible when you move the mouse over the gutter. May improve performance |
| `code-folding`&#8203;`.maxFoldLevel` | `2` | _Since 1.3._ Limits how many levels deep Collapse All applies |
| `code-folding.minFoldSize` | `2` | _Since 1.3._ Blocks must be > _minFoldSize_ lines long (including opening and closing lines) before a collapsible section icon appears |
| `code-folding`&#8203;`.saveFoldStates` | `true` | _Since 1.3._ Remember collapsed sections if you close and reopen a file or project |
| `codehint.AttrHints` | `true` | Enable/disable HTML attribute hints |
| `codehint.CssPropHints` | `true` | Enable/disable CSS/LESS/SCSS property hints |
| `codehint.JSHints` | `true` | Enable/disable JavaScript code hints |
| `codehint`&#8203;`.SpecialCharHints` | `true` | Enable/disable HTML entity hints |
| `codehint.SVGHints` | `true` | Enable/disable SVG code hints |
| `codehint.TagHints` | `true` | Enable/disable HTML tag hints |
| `codehint.UrlCodeHints` | `true` | Enable/disable URL hints in HTML & CSS/LESS/SCSS |
| `dragDropText` | `false` | Enable/disable Drag & Drop functionality |
| `healthData`&#8203;`.healthDataTracking` | `true` | _Since 1.3._ Enable the Health Data Tracking |
| `highlightMatches` | `false` | Enables automatic highlighting of matching strings throughout the document:<ul><li>`true` — highlight all strings that match the current selection (nothing is highlighted when no selection)</li><li>`{"showToken": true}` — highlight all strings that match the token the cursor is currently in (no selection needed)</li><li>`{"wordsOnly": true}` — highlight only when selection is a complete token</li></ul> |
| `insertHintOnTab` | `false` | True to insert the currently selected code hint on tab |
| `jscodehints.noHintsOnDot` | `false` | If true, do not automatically show JS code hints when `.` is typed. |
| `language.fileExtensions` | `undefined` | Additional mappings from file extension to language name (see [Language Support#Preferences](https://github.com/adobe/brackets/wiki/Language-Support#preferences)) |
| `language.fileNames` | `undefined` | Additional mappings from file name to language name (see [Language Support#Preferences](https://github.com/adobe/brackets/wiki/Language-Support#preferences)) |
| `linting.enabled` | `true` | Determines if Code Inspection is on |
| `linting.prefer` | `undefined` | *Since 1.1.* Array of linters (`CodeInspection` providers) to run first. For example: `["ESLint", "SassLint"]`. If provider on the list is not installed, it will be silently ignored. By default, all the linters will be run. |
| `livedev.enableReverseInspect` | `true` | False to disable live preview reverse inspect. |
| `livedev.multibrowser` | `false` | True to enable experimental Live Preview. |
| `linting.usePreferredOnly` | `false` | *Since 1.1.* If set to `true`, then only providers specified in `linting.prefer` will be run (if installed). |
| `maxCodeHints` | `50` | Maximum code hints displayed at once |
| `proxy` | `undefined` | The URL of the proxy server used for extension installation (general syntax: "http://username:password&#8203;@server:port/") |
| `scrollPastEnd` | `false` | True to be able to scroll beyond the end of the document |
| `showCodeHints` | `true` | If false, all code hints are disabled |
| `showCursorWhenSelecting` | `false` | Keeps the blinking cursor visible when you have a text selection |
| `showLineNumbers` | `true` | Show line numbers in a "gutter" to the left of the code |
| `smartIndent` | `true` | Automatically indent when creating a new block |
| `softTabs` | `true` | False to turn off soft tabs behaviour |
| `sortDirectoriesFirst` | `false` for Mac, `true` otherwise | True to sort the directories first in the project tree |
| `spaceUnits` | `4` | Number of spaces to use for space-based indentation |
| `staticserver.port` | | Port number that the built-in server should use for Live Preview |
| `styleActiveLine` | `false` | Highlight background color of the line the cursor is on |
| `tabSize` | `4` | Number of spaces to display for tabs |
| `useTabChar` | `false` | True to use tabs instead of spaces |
| `uppercaseColors` | `false` | Generate uppercase hex colors in Inline Color Editor |
| `wordWrap` | `true` | Wrap lines that exceed the viewport width |

#### A Sample preferences JSON file:-

```json
{
    "spaceUnits": 4,
    "closeTags": {
        "whenOpening": false,
        "whenClosing": true
    },
    "proxy": "http://username:password@proxy.com:8080",
    "path": {
        "src/thirdparty/someLibrary.js": {
            "useTabChar": true,
            "tabSize": 4,
            "linting.enabled": false
        },
        "**.js": {
            "insertHintOnTab": true,
            "scrollPastEnd": true
        }
    },
    "language": {
        "javascript": {
            "linting.prefer": ["ESLint"],
            "linting.usePreferredOnly": true
        }
    },
    "language.fileExtensions": {
        "foo": "javascript"
    },
    "language.fileNames": {
        "pavement": "python"
    }
}
```

With this .brackets.json file at the top of your project, your files will all default to 4 space indentation. However, src/thirdparty/someLibrary.js will be set to use tabs with 4 spaces for the tabs and linting will be turned off. Note that values for paths match fileglob rules. So for example, `*.js` will only match JS files in the root of the project whereas `**.js` will match files with the JS extension anywhere in the project.


What's Next?
------------

For more info on Brackets, check out these resources:
* [How to contribute to the Brackets project](https://github.com/adobe/brackets/blob/master/CONTRIBUTING.md), including filing bugs and editing the source code
* [How to write an extension](https://github.com/adobe/brackets/wiki/How-to-write-extensions)
* Join the community!
    * [@brackets on Twitter](https://twitter.com/brackets)
    * [#brackets on Freenode IRC](http://webchat.freenode.net/?channels=brackets)
    * [Discussion forum](http://groups.google.com/group/brackets-dev)
* [Brackets blog](http://blog.brackets.io/)
* [Troubleshooting for common issues](Troubleshooting)
* Watch our [GitHub activity stream](https://github.com/adobe/brackets/pulse) or [latest release notes](https://github.com/adobe/brackets/wiki/Release-Notes) to see what's in the works for the next update!