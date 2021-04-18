This document explores the ramifications of four ideas, ranked by disruptiveness:
* [Modal dialog with image](https://github.com/adobe/brackets/wiki/Preview-Images-Research#show-modal-dialog-with-image)
* Non modal image viewer in place of the text editor
  * backed by a [standard document](https://github.com/adobe/brackets/wiki/Preview-Images-Research#non-modal-image-viewer-in-place-of-the-text-editor-backed-by-a-standard-document)
  * backed by a [custom document](https://github.com/adobe/brackets/wiki/Preview-Images-Research#non-modal-image-viewer-in-place-of-the-text-editor-backed-by-a-custom-document)
* [backed by a generic image document, here Document and Editor have base classes](https://github.com/adobe/brackets/wiki/Preview-Images-Research#generic-model-where-document-and-editor-have-base-classes)

###  Show modal dialog with image
* add logic to DocumentCommandHandlers.doOpen() to detect file type to open modal dialog.
* click on image in project tree is handles like click on folder
* prevent situation where a binary file is opened as garbage text into the editor

Advantage: 
* smallest change, neither Document API nor Editor API need to change.
* least disruptive, No extensions will break

Disadvantage
* user experience - some say it's not the best.
* no "Save as...".
* inconsistencies regarding how selection in the file tree is handled (thanks Raymond)
* extra work to prevent showing garbage text in editor (thanks Randy)


Open question:
* when would the modal dialog open? Single click, double-click, hover
* _[rlim] You need to mention the selection model in the project tree and working set. What happens with a right-click on an image file? Do we add the image file into the working set? Do we show selection on the image file when the modal image dialog is showing?_
    * _[pf] Currently it sounds like the proposal is that clicking in the tree does nothing, like clicking a folder... but that seems confusing to me. Maybe clicking should show the image, and only right-click should do nothing._
* _[randy] All of the other solutions imply that the problem where editor shows garbage or shows a modal error dialog is fixed, but that needs to be fixed with this solution, so it should explicitly be listed._


### Non modal image viewer in place of the text editor
The following options share the following details:

Behavior:
* Double click on image adds to working set.
* FileOpen adds to working set.
* Save as creates copy of file, updates working set
* cut & paste do nothing when an image is displayed, copy does not update contents of clipboard

Implementation:
* DocumentManager.doOpen checks the documents type by means of the inferred mode and does not read the file contents, instead it create a new empty document, with immutable cm editor instance.
* add new mode / language: API clients like extensions can check the language / mode
`doc.language.id !== “image”`
    * _[jh] I have just looked into this. We could also add 3 new modes: GIF, PNG, JPEG._
* getFocusedEditor returns null
    * _[nj] I'm not clear why this has to be the case in Glenn's proposal. If `getActiveEditor()` can return an "immutable" editor, it seems like `getFocusedEditor()` could return the same._
    * _[pf] It might be nicer though: that way well-written extensions might disable some of their behavior. Any extension that blows up when `getFocusedEditor()` is null would be pretty buggy already anyway..._
    * _[jh] This is a take awy from our initial discussion (jh + nj)_


Common disadvantage: 
* getFocusedEditor returns null - this will break some extensions
* Save as needs extra work
* Find, Replace, QuickOpen UI needs to be disabled, unless we feel ok with the clutter.

#### Non modal image viewer in place of the text editor backed by immutable and empty document
_Also known as Glenn's proposal_
* An HTML document with the image is appended as a div-tag to the code mirror wrapper element.
* A document for an image is an instance of Document, whose text is always empty. The image file's content won't ever be loaded into the editor, it will be loaded for rendering by the browser. It is made immutable by making it's cm editor instance immutable.
* All calls to text-based APIs (get text, cursor, selection) should remain unchanged and respond as they would on an empty document.
* Implement support immutable documents, APIs that modify text would have to be tweaked to check for mutability. When called on an immutable document any of these would silently do nothing.
    * _[pf] I'd suggest this should actually be a subclass of Document with stubbed-out mutators, rather than cluttering all the existing Document methods with read only checks. (Which makes this more similar to the next proposal below, except that there'd still be an Editor)._
* EditorManager.focusEditor() returns focus to last element shown in main editor space, i.e. image if that had focus or last editor otherwise
* _[nj] The spec doesn't explicitly describe what happens to `getActiveEditor()` or `getCurrentFullEditor()` in this case._
    * _My understanding of Glenn's proposal is that `getActiveEditor()` and `getCurrentFullEditor()` would return an ordinary Editor whose `_codeMirror` instance would still exist (behind the image div), but would be read-only. Is that still true? If so, we should explain that, and mention that our intent would be to make it so that instance basically never takes user input._
    * _Also, are we going to bother to stub out APIs on the CodeMirror instance that programmatically modify it, or are we just not going to care if an extension tries to write into it?_
        * _[pf] I think Glenn's proposal was to set the CodeMirror instance to readonly too, which should internally block write attempts._

Advantage: 
* since EditorManager and DocumentManager APIs are unchanged, fewer extensions will break.
* Since documents are added to the working set, the editor state persists across sessions.
* Rename, Delete, Show in File Tree, Show in OS still work

Disadvantage: 
* extra work to enable immutable documents    
    * _[pthiess] I do agree there might be some hidden work or potential to break extensions in case the extension tries to alter the immutable document._
* extra work to maintain EditorManager.focusEditor
* Documents class gains in complexity, no separation of concerns for text and image documents, seems harder to maintain if new editors are added, i.e. image editors, hex editor, ...
* Not clean enough to enable editable binary content, i.e. to enable an image editor

####  Non modal image viewer in place of the text editor backed by a custom document
_Also known as the original proposal_ - based on the discussion and outline [here](https://github.com/adobe/brackets/pull/4492) 
* A HTML document with the image is placed in place of the editor. A document for an image is a new type of immutable document. 
* ImageDocument asserts / throws errors when any text-related APIs are called.
* DocumentManager.getDocumentForPath(fullPath) on path to image returns ImageDocument.
* DocumentCommandHandlers.doOpen() generates a special image document
* ImageDocument is immutable
* ImageDocument has no editor, but a view
* DocumentManager.getCurrentDocument()  - returns either standard document or image document
* Image document does not have any of the methods related to text.
* EditorManager.getCurrentFullEditor - returns NULL when ImageDocument is displayed
* EditorManager.getFocusedEditor - returns NULL when ImageDocument is displayed
* EditorManager.getActiveEditor - returns NULL when ImageDocument is displayed
* Listeners to DocumentManager event currentDocumentChange or EditorManager event activeEditorChange may break. Event activeEditorChange will have NULL as argument in place of new Editor.
* EditorManager.focusEditor() gives focus to ImageViewer

Advantage: 
* conceptual clarity through separation of concerns. 
* Opens a path to add more types of documents

Disadvantage: 
* compared to the previous proposal more extensions break, because there are more API changes.

Random and incomplete  collection of extensions will likely break:

code folding, delete-line-start-end, superclipboard.js, case-converter, brackets-special-html-chars, brackets-minifier, brackets-indent-guides, spell-check, brackets-xunit, brackets-beautify, camden.w3cvalidation, cezarwojcik.cleaner, dehats.annotate, dehats.prefixr, dehats.togist, fontParser, enturn.quick-search, fontface.brackets-vimderbar, mikaeljorhult.brackets-autoprefixer, ...:
* _[pf] I think list is a bit over-aggressive. I looked at the camden.w3cvalidation, brackets-minifier, and brackets-indent-guides extensions and I'm pretty sure they would all work fine._

Note on the meaning of _breaking_: In many cases errors will only occur if i.e. a command provided by an extension is invoked while an image is displayed. The extension may continue to work fine on regular text documents. But some extensions will be rendered completely broken, i.e, those that make calls to changed APIs on startup.

####  Generic model where Document and Editor have base classes
seems to be the least likely choice, being the most disruptive.
* DocumentManager.getCurrentDocument()  -  returns BaseDocument. Any consumer of this API needs to check whether it can call text or cursor related APIs
* EditorManager.getCurrentFullEditor - returns BaseEditor. Any consumer of this API needs to check whether it can call code mirror /text editor related APIs
* EditorManager.getFocusedEditor - returns BaseEditor. Any consumer of this API needs to check whether it can call code mirror /text editor related APIs
* EditorManager.getActiveEditor - returns BaseEditor. Any consumer of this API needs to check whether it can call code mirror /text editor related APIs

This is not fully fleshed out as it doesn't seem to be the obvious choice.

Disadvantage: 
* Some new code to write for Base classes, plenty of core code to change. The amount of changes to the core code make this proposal the most uncertain, thus the most risky.
* extensions break - expected about the same amount as in previous option _original proposal_

### Summary
I suggest to implement _Glenn's proposal_ for the following reasons
* better user experience than Modal dialog with image
* few extensions break, fixes should be straightforward
* requires a reasonably small change to core code, which introduces minimizes the risk regarding unexpected challenges that may have been missed writing up this document.
* the simplicity outweighs a lack conceptual clarity. 

While this relatively small change enables a solution with a good user experience  it seems small enough to revert, i.e in case of the need for more flavors of viewers and editors. During feature discussion the need for displaying additional types of documents was rated unlikely. 