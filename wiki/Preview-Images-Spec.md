This document describes API changes needed to enable viewing image files within Brackets - see the [_Preview Images_ user story](https://trello.com/c/l9AcILkC/24-8-preview-images).

Brackets currently has notions of "current document" and "active editor." The `Document` and `Editor` objects are closely tied to _text_ content with a CodeMirror widget managing the state and view. Something in that picture will have to change to enable selecting image files for display.

To minimize changes to these existing APIs, "current document" and "active editor" _will both be **null**_ when an image is displayed. Previously, they could only be null when all files were closed.

We think this change can achieve all of the following goals:

* avoid breaking extensions (if we do, we'd better have a good reason)
* build core code that is maintainable
* build core code that is extensible
* keep API intuitively understandable

---
## Have extensions?
If you have written extensions, here's what you need to know...

You can use this branch to test your extensions with the new API behavior:
https://github.com/adobe/brackets/tree/couzteau/preview-images


### EditorManager getters
`getFocusedEditor()`, `getActiveEditor()` and `getCurrentFullEditor()` all return null when an image is displayed. (Note: it's _already_ possible for any of these to return null if all files are closed; and `getFocusedEditor()` is also null if keyboard focus lies outside the Editor). Your code should contain null checks such as this:

~~~~
var editor = EditorManager.getActiveEditor();
if (editor) { /* always check editor for null! ... */ }
~~~~


### DocumentManager getters
`getCurrentDocument()` returns null when an image is displayed. Note: it's _already_ possible for it to return null if all files are closed. Use null checks similar to the above.

Because of this, `getCurrentDocument() === null` no longer guarantees that `getWorkingSet().length === 0`.

### Events from EditorManager & DocumentManager

When opening an image file while a text document is open, the `"activeEditorChange"` and `"currentDocumentChange"` events are triggered. The second `"activeEditorChange"` argument will be null, matching the getter API value. (It's _already_ possible to receive such events transitioning to null, when the last open file is closed). Listeners should always check for null, like this:

~~~~
function _onCurrentDocumentChange() {
   var doc = DocumentManager.getCurrentDocument();
   if (doc) { // make sure to check for null!
     ...
   }
   // even better use example above 
}
$(DocumentManager).on("currentDocumentChange", _onCurrentDocumentChange);
~~~~

When switching from an image file back to a text document, both these events are triggered again. The third `"activeEditorChange"` argument will be null, matching the previous getter API value. (It's _already_ possible to receive such events transitioning from null, when the first file is opened).

When switching between two image files, _neither_ of these events are triggered (since the values are remaining null).


---

## Spec

The following design has been picked after investigating a number of architectures comparing trade offs between making non-disruptive or less disruptive API changes at the cost of conceptual clarity of Editor and Document classes.

This design does seems to strike a good balance between the goals stated above

Behavior details
* Single click on image in project display image
* Images are not added to working set
* FileOpen opens an image if a single image file is selected in the open dialog
* Drag & Drop of single image displays image
* Drag & Drop of multiple images does not display any image
* No _Save As_ for images
* File Rename, Delete, Show in File Tree, Show in OS work on image files the same way as for other files.
* Find, Replace, QuickOpen UI and the likes do not show UI when an Image is displayed
* Copy does not modify the clipboard contents
* Cut, Paste do nothing on an image file.
* File->close, CMD-W closes image

Implementation details
* DocumentManager.getCurrentDocument returns null while an image is displayed, so that code / extensions that modify documents do not have to be updated.
* EditorManager.getFocusedEditor always returns null when image is displayed
* EditorManager.getActiveEditor always returns null when image is displayed
* EditorManager.getCurrentFullEditor always returns null when image is displayed
* A div-tag containing an img-tag is appended to the code mirror wrapper element to display the image. The image itself is not loaded at all. It is loaded by the browser / CEF  to render the view.
* A new mode / language will be added such that LanguageManager.getLanguageForPath() on an image can be called to identify wether fileEntry points to an image file, by checking mode.getName(), i.e.
`var mode = LanguageManager.getLanguageForPath(fullPath);
if(mode.getName() === "image"){...}`
* Extensions that are prepared for getCurrentDocument to return NULL will be fine.
* DocumentManager.getDocumentForPath is modified so that no document will be created for paths to image files.
DocumentCommandHandlers.doOpen will own the responsibility to determine wether an image file or a text file or something else is attempted to be opened.

### Screen mockups
![1](https://trello-attachments.s3.amazonaws.com/4f90a6d98f77505d7940ce88/4f91ec23c0e7c29036c1e92f/93e6ecaa4aec4fe4d174427ca515bd7a/Brackets_ImageView_005.png)
![2](https://trello-attachments.s3.amazonaws.com/4f90a6d98f77505d7940ce88/4f91ec23c0e7c29036c1e92f/e3a0f30e8308702b98469e910d0c7a5f/Brackets_ImageView_004.png)
![3](https://trello-attachments.s3.amazonaws.com/4f90a6d98f77505d7940ce88/4f91ec23c0e7c29036c1e92f/468e703d2a78ee5f0b7f9774331483ca/Brackets_ImageView_003.png)

[Further discussion on this story](https://github.com/adobe/brackets/wiki/Preview-Images-Research----old-drafts)