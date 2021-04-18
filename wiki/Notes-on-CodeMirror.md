The core text editor in Brackets is [CodeMirror](http://github.com/marijnh/CodeMirror). The Brackets
team has submitted or contributed to a number of features and bug fixes in CodeMirror, and has also
given input into the design of features like line widgets, document/view separation and multiple selections.

Currently, Brackets uses a [fork of CodeMirror](http://github.com/adobe/CodeMirror2) as a submodule. 
Originally, we created our own fork to hold a number of experimental features that Brackets implemented (including a prototype implementation of inline widgets) that we hadn't yet contributed upstream. Now that the
upstream CodeMirror repo has versions of these features, we generally keep Brackets in sync with the upstream 
version.

In general, the `master` branch in our fork ([adobe/CodeMirror2](http://github.com/adobe/CodeMirror2)) is updated to match upstream/master at the beginning of each release, and we don't do any work in master ourselves. Usually we keep the Brackets submodule pointed at master, but in cases where there are
last-minute upstream changes that we don't want to pull into a given sprint, and we need other fixes,
we sometimes make short-lived sprint branches.

For bugfixes we want to make in CM, we generally submit pull requests directly to the 
[main CodeMirror repo](http://github.com/marijnh/CodeMirror), then pull them down once they've been merged upstream. Similarly, if you have fixes you need in CodeMirror, please submit them upstream (you can file an issue with us to let us know that it's important for us to merge it soon, or refer to your upstream fix from a dependent pull request). We generally won't accept pull requests directly into our CodeMirror fork.

Note that we have not renamed the submodule or the repo to reflect the current naming of the upstream
repo--it's still "CodeMirror2", since that was the old name of the upstream repo when we started Brackets. This renaming is a bit tricky and isn't urgent, so we probably won't bother. See [this Google Group thread](https://groups.google.com/forum/?fromgroups=#!topic/brackets-dev/D_rezwjyXM0)
for more info.
