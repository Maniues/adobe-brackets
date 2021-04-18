(submitted by Pam Fox)
## Answered Questions on Brackets

(Might help you in figuring out what other people would ask)
* **What would be the difference between embedding Brackets and embedding CodeMirror?**
    * CodeMirror: just editing,
    * Brackets: file management, server APIs (github), extensions+inline

* **How does the editor work?**
    * CodeMirror, pushing changes back into CodeMirror.
* **Are there apps for iPad/Android?**
    * Not now, will wrap in PhoneGap in the future
* **Is it at all possible to run it on web right now, or has noone written that?**
    * They tried it once a while ago. Tricky because dependent on Chromium version, might not work with other browsers or look good. Also file API.
* **Is there a video that shows the cool demo’d features (inline CSS/HTML/JS)?** 
    * Not yet.
* **Are there any plans to use right-click/context menu?**
    * Eventually.
* **How would it work with Less/Sass? Spriting?**
* **What about server-side code? Templates?**
* **Could write extensions. (repl.it?)**


## Feedback/Ideas on Brackets

* Maybe can use the JS evaluators from repl.it for other languages/extensions.
* For people on ChromeBook or iPad or Android tablet, they have no ability to use a desktop editor - that’s where a cloud version can be very valuable.
* It could be interesting to provide a virtual keyboard for people on shitty keyboards (iPad, iPhone, non-English keyboards)
* For embedding, can you separate the brackets with extensions from the file API stuff? (for single-file editing).
* Use jshint, the next version lets you do style stuff.

## Running Feedback/Questions while using Brackets:

* **Q: What library is $ from?**
    * jQuery
* **Q: Where do you put a new HTML file when you’re testing brackets changes?**
    * Make a new window, new folder, reload
* **Q: How to save as?**
    * Not yet
* **Q: Can you drag+drop files around? (how to put file in other folder?)**
    * Not yet
* **_Bug_: When you add a folder from Finder, the UI doesn’t show it, have to reload**
* **_Feature_: Change title of multiple windows to show folder name or some way of differentiating.**
* **_Bug_: When you put a breakpoint while debugging and close Developer Tools before removing the breakpoint, Bracket dies.**
* **Q: How do you change indenting/spacing?** 
    * Edit CodeMirror, no UI for it yet.
    * (But if you do it, change white:true for your own code)

```js
this._codeMirror = new CodeMirror(container, {
    electricChars: false,   // we use our own impl of this to avoid CodeMirror bugs; see _checkElectricChars()
    indentUnit: 2,
    indentWithTabs: false,
    lineNumbers: true,
    matchBrackets: true,
    extraKeys: codeMirrorKeyMap
});
```

## Feedback while making extensions:

* **_Feature_: It’d be good to have a general library for extensions for common tasks, like:**
    * get current word
    * get current selection (currently have EditorManager.getFocusedEditor().getSelectedText() and a pull request for EditorManager.getFocusedEditor().replaceSelection(text); by mesh)
    * iframe/image/HTML viewers (with styling options)
* **Q: How to debug extension?**
    * Use console.log
    * Use breakpoints
    * Use magnifying glass
* **_Bug_: right-click and inspect element in Brackets inline editor doesn’t work.**
* **_Docs_: Need better documentation on how to trigger extensions (UI hooks), via keyboard commands, Inline command, menu, quick open etc.**
* **Q: For extension like InlineImageViewer, how do you detect live changes to the image src to reload the image viewer?**
* **_Feature_: The inline editor could benefit from a close button**
* **_Feature_: Extensions could be placed on a toolbar (see my MDN extension), so users don’t have to remember tons of keyboard commands**
* **_Feature_: Extensions could define UI hooks in config.js that would be user-editable, so they could turn different UI hooks on/off, similar to what SublimeText does.**

## Extension ideas:
* HTML escaper
* Look up on MDN/caniuse/etc
* CSSLint
* Font Previewer