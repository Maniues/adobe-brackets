This checklist is for Brackets Committers to assist in reviewing pull requests to ensure that they are ready to merge. It has some elements in common with the [Pull Request Checklist](https://github.com/adobe/brackets/wiki/Pull-Request-Checklist) because committers have a chance to double check work before it goes in.

How to use the checklist:

* The purpose of the checklist is to make sure that we don't forget anything important
* Consider each item in the list, but only act on the items that you think are relevant to the change you're looking at

## Proposed Pull Request Review checklist

### Ensuring the change is ready to go in ###

* Has the user [signed the CLA](http://dev.brackets.io/cla/brackets/check.cfm)?
* Was there new third-party code? If so, what's the license? (ideally MIT or BSD)
* Does this change belong in core? Some features would be better as an extension - could it be done as an extension by separating out a more limited set of core changes (e.g. more generic APIs)?
* Any major architectural or UI changes have been discussed in the forum?
* All new APIs are documented in the [Release Notes] (https://github.com/adobe/brackets/wiki/Release-Notes)?

### Testing the change ###

* Download the code
    * `git checkout -b (username)/(branchname)`  (create a new branch to hold the changes)
    * `git pull (URL of contributor's git repo) (branchname)` (download their changes)
* Build on Travis-CI passes (you'll see the results in the pull request on GitHub)
    * This ensures that code passes JSHint
* Look for JSLint errors
    * New or obsolete globals, for example
* Check copyright date on file
* Code is syntactically valid (Brackets launches & no exceptions in the console)
* All unit, integration and extension tests pass
* [Smoke tests](Brackets-Smoke-Tests) and server smoke tests have been run (nontrivial changes specifically)
* Try out edge cases for the change
    * Clearly depends on what the change is
    * Look for problems at beginnings/ends of files, top/bottom of the editor window
    * Make sure inline editors work well, too
* UI is reasonably polished ?

### Other things to look for when reviewing the code ###

* Code follows our JS coding style guidelines (we probably need to clean those up)
* Edge cases not covered by the code
* Code has unit tests
* All UI strings externalized (we should have a how-to page for this).
* If there are string changes, it can't land at the very end of the sprint
* Native: should compile
* Native: Mac AND Win implementations

### At the end ###

* GitHub's merge button is the easiest way to merge the changes in
* Do a manual merge if you make any minor tweaks to the pull request
* `git branch -D (username)/(branchname)` to remove the test branch

### Avoid Common pitfalls

(make sure these have been thought about):

* Asynchronicity: look for possible race conditions and assumptions about the order of operations that might be invalid
* Text manipulation commands: should consider what happens when in an inline editor at boundaries
* Inline editors: does this collide with any other providers?
* Code hinting: does this collide with any other providers?