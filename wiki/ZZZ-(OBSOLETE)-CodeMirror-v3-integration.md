### NOTE: This page is now obsolete, as CodeMirror v3 has landed in Brackets master. See [Notes on CodeMirror](https://github.com/adobe/brackets/wiki/Notes-on-CodeMirror) for more info on how we're managing our CodeMirror fork going forward. ###

We've started the official integration of CodeMirror v3 into Brackets. You can get this by pulling the [`cmv3`](https://github.com/adobe/brackets/tree/cmv3) branch from Brackets, then doing `git submodule update`, which will switch the CodeMirror2 submodule to use the [`upstream-master`](https://github.com/adobe/CodeMirror2/tree/upstream-master) branch (now that v3 has been officially merged into CodeMirror's master).

We will be submitting various small changes from the `v3-brackets` branch into the upstream CodeMirror repo. We are currently tracking these as separate feature branches in our repo. Currently, the feature branches are:

* [`v3-dirty-bit-final`](https://github.com/adobe/CodeMirror2/tree/v3-dirty-bit-final) -- **Already merged upstream.** Adds the ability to track a "dirty state" for a document that can be reset with "markClean()". This enables the presentation of a dirty dot in the UI that works appropriately with respect to save, undo, and redo. 

## Current issues with the integration

**General bugs:**
These are now being tracked in the Brackets issue tracker under the [CodeMirror v3 label](https://github.com/adobe/brackets/issues?labels=codemirror+v3&page=1&state=open).

**Missing functionality:**
* No way to turn off fixed gutter

**Inline editor bugs:**
* Cmd-E from within an inline editor (to close it) doesn't work
* Deleting a line doesn't close its attached inline editor
    * I put in code to handle the "delete" event on the line, but it doesn't seem to be working. Not clear if this is our bug or a CodeMirror bug.
* Can get two inline editors on same line (CM doesn't enforce the same rule we did--we'll need to specifically check for this)
* With the new markText() API for line hiding, empty lines above and below visible lines are shown
* Touch-scrolling over inline editor doesn't work (I think this worked in earlier v3 versions)
* Horizontal scroll area is huge when inline editor is open
    * We were setting the min-width of the inline editor to match the overall width of CodeMirror's linespace, but this doesn't work anymore because the inline editor is actually inside the linespace (and the total width of the editor is wider than the min-width, since we set padding to account for the rule list width). We need to rethink how we're managing the width of the inline editor.
* Scrolling past the right edge of the inline editor with cursor movement doesn't scroll the whole doc (because scrollIntoView() isn't exposed--could we write our own using scrollTo and getScrollInfo?)
* addLineWidget() doesn't have scrollIntoView property
* Scrolling of rule list lags behind widget
    * The issue is that the browser is now handling scrolling itself, so the display will always update immediately, while the rule list is moving asynchronously on the scroll event. (In our old flicker fix scheme, both the display update and the rule list moving happened on the scroll event.) I'm not sure of a good way to fix this; we can't just make the rule list be a child of the scroller, because we want it to stay stuck to the right edge of the editor window (which is why it's position: fixed, and we reposition it in JS). We would need to spend a couple of days trying to find an alternative solution.
* Motion of rule list when horizontally resizing window lags
    * This seems less noticeable in the latest v3.
* Widget loses focus when scrolled far off the screen
    * The CodeMirror implementation appears to remove widgets from the display list when they scroll off the screen. We'll need to ask Marijn if he'd be willing to add an option to leave certain widgets on the display list (maybe turning off their visibility). If not, we could conceivably live with this since it's kind of an edge case.

**Stuff to test:**
* Middle mouse button

**New things we can take advantage of:**
* setSize() to explicitly set width/height
* getHistory()/setHistory() to get history info
* Multiplexing of event listeners no longer necessary for most events (but still needed for onKeyEvent)

## Porting changes from our fork of CodeMirror

The brackets fork of CodeMirror has a bunch of changes that were _not_ integrated into the upstream branch. Most of the changes were dealing with inline widgets, but there were several other small features and performance improvements. 

The inline widget changes need to be re-implemented on top of the new line widget API added to CodeMirror 3. A first pass at this has been done in the `nj/cmv3` branch, but it has the issues noted above.

Here are notes on the other changes that were made.

### Functionality that does NOT need to be ported

#### Handle hidden lines in coordsChar() - ([commit 11f204](https://github.com/adobe/CodeMirror2/commit/11f2041529d798f9a916be2f146edf165c2b5434))

This fixed performance problems when the contents of an inline editor were near the top of a very large file. For example, the `body` selector in bootstrap.css. The v3 branch of CodeMirror does not have the same performance problems, so this change doesn't need to be ported.

#### Use document fragments in patchDisplay() - ([commit cf24f7](https://github.com/adobe/CodeMirror2/commit/cf24f7240b3bf7edb63cf9f4a625f5c16dff7620))
The v3 branch contains a more comprehensive fix for using document fragments, obviating our changes.

#### Mouse handling for drag selecting and autoscrolling across inline editors
The v3 branch seems to handle this pretty well.

#### Changes to updateGutter()
These are no longer necessary with the new inline editor implementation. They were only required because we were injecting dummy nodes into the gutter.

#### Right-click fix to `onMouseDown()`
It turns out we didn't need to port this. We've done some testing around our context menus and they seem to work fine without it.

### Functionality that needs to be ported

Most of these changes should be trivial to port, as long as Marijn is okay accepting them upstream.

#### Dirty bit handling (`isDirty()`, `markClean()`)
This has already been ported and merged upstream. The API was changed to have `isClean()` instead of `isDirty()`.

#### Exposing CodeMirror methods
We exposed the CodeMirror methods `selectWordAt()` and `scrollIntoView()` for use in Brackets. 
* For `selectWordAt()`, we decided to pull the implementation into Brackets since we might want to modify it in future, perhaps in a mode-specific way. This has already been merged into Brackets master (not just cmv3).
* v3 appears to expose `scrollIntoView(pos)`; we'll need to see if we can implement the functionality we need purely using that, or if we need to be able to pass rects as we are today.

#### `totalHeight()`
We added this function in order to determine how tall an inline editor should be to show all of its content. We might be able to just get this by letting CodeMirror lay itself out and then measuring its height, but I believe there were reasons why that didn't work before. If we can't get that to work, then we'll need to port this, which should be easy.

#### Checking `isInWidget()` in various cases and returning early
We will likely need to port these so that CodeMirror doesn't handle things like context menus or double-click in widgets.

#### `preserveScrollPosOnSelChange`
This makes it so that when you do Cmd/Ctrl-A to select all, the editor doesn't scroll. This should be easy to port.

### Changes to htmlmixed mode
If we encounter a `<script>` tag in HTML that seems to be a Mustache template, we treat it as HTML, and if we don't recognize it, we leave it uncolored instead of attempting to color it as javascript. We should submit this upstream.

## Forking strategy

Currently, the `master` branch in adobe/CodeMirror2 is completely out of sync with the upstream CodeMirror master, because we actually checked all of our changes directly into our master. In hindsight, that wasn't a good idea :)

For now, since all the v3 work is in a separate branch in the CodeMirror repo, we don't have to worry about our master; we can simply point the submodule SHA in Brackets to the v3 branch. However, once Marijn merges v3 into CodeMirror master, we'll need to bring our master back into sync.

Our proposal is to hard reset our master to the upstream CodeMirror master at that point. Our assumption is that there are no (or very few) people working on the adobe/CodeMirror2 fork, so this shouldn't disrupt anything. Once we've done that, we will never make commits directly to master in our fork; we'll either submit patches directly to Marijn and wait for them to be merged upstream, or we'll create a separate Brackets-specific branch and point our submodule SHA to that branch.

Here's the proposal in more detail:

### Before v3 is merged into upstream master

1. We will pull the `v3` branch from upstream into adobe/CodeMirror2 and keep it in sync with upstream.
2. We will have a separate branch in adobe/CodeMirror2 that contains any local changes we need for now as we're investigating the integration. (Currently, this branch is `nj/cmv3-brackets`).
3. If we finish our integration before v3 officially gets merged into upstream master, we'll keep this separate branch in adobe/CodeMirror2, and point the main Brackets submodule SHA at this branch.

### After v3 is merged into upstream master

1. We will rename adobe/CodeMirror2 to adobe/CodeMirror (and update the submodule info in Brackets) to stay in sync with Marijn's repo name.
2. On our master, we'll create a `brackets-old` branch based on the current state of our master.
3. We'll then do `git reset --hard upstream/master` in order to point our master back at upstream. We will never commit directly to our master after this.
4. Anyone else who has actually made unsubmitted changes in their fork of adobe/CodeMirror2 will similarly need to reset their master branch to ours, and deal with merging any changes they have into a branch based on the new master.
5. Since there will be some functionality in our old CodeMirror branch that will need to be ported over to v3, we'll create a separate `brackets` branch in adobe/CodeMirror to hold this ported functionality, and point the Brackets submodule at that branch.
6. We'll keep the `brackets` branch as clean as possible, rebasing as necessary to make it so each commit can be submitted directly to CodeMirror upstream as a pull request.
7. Over time, we will aim to get everything from the `brackets` branch into CodeMirror upstream, and then just point Brackets at the master branch of adobe/CodeMirror. We'll only revive the separate branch if we need to implement functionality that will take awhile to get accepted upstream, but we'll always try to get it pushed upstream as soon as possible.