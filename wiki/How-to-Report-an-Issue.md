Whether you've found an issue with Brackets, the Brackets Registry or the Brackets Shell, here's how to report the problem...

First, check our **[Troubleshooting page](https://github.com/adobe/brackets/wiki/Troubleshooting)** for solutions to common problems.

### How to file a bug

1. Go to our **[issue tracker on GitHub](https://github.com/adobe/brackets/issues)**
2. Search for existing issues using the search field at the top of the page
3. File a new issue _including the info listed below_
4. Thanks a ton for helping make Brackets higher quality!

_**When filing a new bug, please include:**_

* **Descriptive title** - use keywords so others can find your bug (avoiding duplicates)
* **Steps** to trigger the problem that are specific, and repeatable
* **What happens** when you follow the steps, and **what you expected to happen** instead.<br>Include the exact text of any error messages if applicable (or upload screenshots).
* **Brackets version** (or if you're pulling directly from Git, your current commit SHA - use `git rev-parse HEAD`)
* **Did this work in a previous version?** If so, also provide the version that it worked in.
* **OS version**
* **Extensions?** Confirm that you've tested with _Debug > Reload Without Extensions_ first (see below).
* **Any errors logged** in _Debug > Show Developer Tools_ - Console view

### If you've installed extensions

Bugs can be caused by Brackets extensions you've added. Before you file an issue, use **Debug > Reload Without Extensions** to see if the problem still occurs without any extensions.

If that fixes the problem, then please file an issue in the extension's repo instead, so it can be addressed by the extension author.  See [Troubleshooting extension bugs](https://github.com/adobe/brackets/wiki/Troubleshooting#wiki-disable-all-extensions) for tips on identifying which extension is causing the problem.

### Requesting a feature

Please first check our [feature backlog on Trello](http://bit.ly/BracketsBacklog) to see if it's already there. You can vote on features in the backlog to help us prioritize them.

Feel free to file new feature requests as an issue on GitHub, just like a bug. We tag these issues "move to backlog" and periodically migrate them onto the feature backlog for you.


# What happens after a bug is filed?

### Bug lifecycle

1. New bug is filed; awaiting review
2. Triaged in bug review -- see below (_'last reviewed'_ tag
3. Developer begins working on it -- bug is tagged _'fix in progress'_
4. Developer opens pull request with a fix, which must be reviewed -- a link to the pull request appears in the bug's activity stream
5. Pull request is merged, and the bug's filer is pinged to verify that it's fixed -- bug is tagged _'fixed but not closed'_ ("FBNC")
6. Filer agrees that it's fixed -- bug is closed, and its milestone is set to the release the fix landed in

### Bug review

We review _all_ new issues on a regular basis. Several things typically happen as part of review:

* Ping the filer for clarification if needed.
* If the issue is a feature request, we'll tag it [_'move to backlog'_](https://github.com/adobe/brackets/issues?labels=move+to+backlog&state=open); the issue will be migrated to our [feature backlog](http://bit.ly/BracketsBacklog) at a later date.
* Add priority labels (high, medium, low, or none - [read more below](#wiki-bug-priority)).
* Add release milestone - typically only if a bug is a "release blocker" or related to features still in progress.
* Add feature area label, and possibly other category labels like _'starter bug'_ or _'Mac only.'_ See ["Understanding issue labels" below](#understanding-issue-labels).

Depending on priority, milestone, and other workload, a developer may or may not begin working on the bug soon.

Some bugs may be closed without fixing - see ["Hey! My bug wasn't fixed!" below](#hey-my-bug-wasnt-fixed).

### Hey! My bug wasn't fixed!

Yeah, what's up with that? There are a number of reasons an issue might get closed without being fixed:

* Tagged 'move to backlog' - It's not forgotten! Look for a link in the comment thread to our [feature backlog](http://bit.ly/BracketsBacklog). Please vote on the story to help us prioritize it!
* Tagged 'fixed but not closed' - _We_ think it's fixed. Make sure you have a Brackets build containing the fix (check the milestone assigned to the bug). If you're still seeing it, let us know.
* Duplicate - There's already a bug for this.
* Unable to reproduce - We're unable to reproduce the result described in the bug report. If you're still seeing it, please reply with more detailed steps to trigger the bug.
* Out of scope / extension idea - This change probably doesn't belong in Brackets core... but it could still make for a great extension!
* Not a bug / fact of life - This is the intended behavior. If it feels wrong, we should discuss how to improve the usability of the feature.

If you disagree with a bug being closed, feel free to post a comment asking for clarification or re-evaluation. The more new/updated info you can provide, the better.


# Can I help fix a bug?

Yes please! But first...

* Make sure no one else is already working on it -- if the bug has a milestone assigned or is tagged _'fix in progress'_, then it's already under way. Otherwise, post a comment on the bug to let others know you're starting to work on it.
* Read the [guidelines for contributing code](https://github.com/adobe/brackets/blob/master/CONTRIBUTING.md#contributing-code), [pull request guidelines](Pull Request Checklist), and [coding conventions](Brackets Coding Conventions).


# Understanding issue labels

We use labels/tags for a number of purposes:

<table width=90% border="5" cellpadding="1" bordercolor="#000033">
  <tr>
    <th width=15% scope="col"></th>
    <th width=15% scope="col">label</th>
    <th width=60% scope="col">meaning</th>
  </tr>
  <tr>
    <td rowspan="4">Process </td>
    <td>fix in progress</td>
    <td>Someone has started work on a fix (or the fix is ready but still undergoing code review - not merged yet).</td>
  </tr>
  <tr>
    <td>fixed but not closed</td>
    <td>Fix has been merged. Waiting for the original bug reporter to verify that it's fixed.</td>
  </tr>
  <tr>
    <td>last reviewed</td>
    <td>Last reviewed/triaged bug. See <a href="#bug-review">"Bug review" above</a>.</td>
  </tr>
  <tr>
    <td>move to backlog</td>
    <td>Feature/enhancement request rather than a bug. Will be moved to the <a href="http://bit.ly/BracketsBacklog">feature backlog</a>.</td>
  </tr>
  <tr>
    <td rowspan="4">Priority <a name="bug-priority"></a></td>
    <td>high priority</td>
    <td>High impact bug many users will hit (e.g. crash/data loss). We aim for zero high-priority bugs before each release.</td>
  </tr>
  <tr>
    <td>medium priority</td>
    <td>At least somewhat severe and a significant number of users will hit.</td>
  </tr>
  <tr>
    <td>low priority</td>
    <td>Low severity (e.g. small cosmetic issue) and/or few users will hit.</td>
  </tr>
  <tr>
    <td>no priority</td>
    <td>We don't plan to  spend time fixing the bug - but we would accept a fix if someone offers a pull request.</td>
  </tr>
  <tr>
    <td>Feature area </td>
    <td>F ...</td>
    <td>Labels starting with "F" categorize bugs by feature area.</td>
  </tr>
  <tr>
    <td rowspan="2">Suggestions </td>
    <td>starter bug</td>
    <td>Recommended for <a href="https://github.com/adobe/brackets/blob/master/CONTRIBUTING.md">new contributors</a> as a good intro to the Brackets source code.</td>
  </tr>
  <tr>
    <td>Extension Idea</td>
    <td><em>May</em> be out of scope for Brackets core, but is a great idea for a Brackets extension.</td>
  </tr>
  <tr>
    <td rowspan="3">External tracking </td>
    <td>codemirror</td>
    <td>Needs <a href="https://github.com/marijnh/CodeMirror">CodeMirror</a> changes.</td>
  </tr>
  <tr>
    <td>cef<br>webkit</td>
    <td>Needs <a href="https://code.google.com/p/chromiumembedded/">Chromium Embedded Framework</a> or WebKit/Chromium changes.</td>
  </tr>
  <tr>
    <td>tracking</td>
    <td>Specific bug has been filed in external project - waiting on a fix.</td>
  </tr>
  <tr>
    <td rowspan="5">Architecturally-focused</td>
    <td>architecture</td>
    <td>Requires significant architectural changes - needs longer discussion.</td>
  </tr>
  <tr>
    <td>performance</td>
    <td>Perceived or measurable performance issue.</td>
  </tr>
  <tr>
    <td>code cleanup</td>
    <td>The fix improves code maintainability without changing Brackets's functionality.</td>
  </tr>
  <tr>
    <td>async</td>
    <td>Bug caused by asynchronous execution / race condition</td>
  </tr>
  <tr>
    <td>native shell</td>
    <td>Needs code changes in <a href="https://github.com/adobe/brackets-shell">brackets-shell</a>, our desktop native wrapper</td>
  </tr>
</table>

### Other bug-related terminology

Acronyms we use frequently in bug comments:

* _FBNC_ = "fixed but not closed": See bug lifecycle
* _UTR_ = "unable to reproduce": Someone tried to follow the steps in the bug, but everything seemed to work fine.
* _NAB_ = "not a bug": The behavior described in the bug is actually how it's supposed to work. This may indicate a usability or documentation problem.
* _FOL_ = "fact of life": The bug is virtually impossible to fix due to constraints of the OS, runtime, etc.