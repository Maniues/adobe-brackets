High quality code and a top-notch user experience are very important in Brackets, and we carefully review pull requests to keep it that way. Here's what we expect of a good quality pull request — be sure to follow all these items for a smooth landing!

Note: you must sign the **[Brackets Contributor License Agreement](http://dev.brackets.io/brackets-contributor-license-agreement.html)** before we can merge your first pull request.


###Pull Request Checklist

1. Discuss any major architectural or UI changes in the [brackets-dev newsgroup](http://groups.google.com/group/brackets-dev)
2. Does this change belong in Brackets core? Some features would be better as an extension — which may require factoring out a generic set of core API changes to enable writing the extension. When in doubt discuss in the newsgroup
3. Code follows our [JS coding style guidelines](https://github.com/adobe/brackets/wiki/Brackets-Coding-Conventions)
4. Code is well documented, including Closure-style [type annotations](https://developers.google.com/closure/compiler/docs/js-for-compiler#types)
5. Code passes JSLint
6. Testing
    * Code has been tested — in your pull request, describe the cases you tested
    * No known bugs
    * All [unit tests pass](https://github.com/adobe/brackets/wiki/Running-Brackets-Unit-Tests) ... (<i>Debug > Run Tests</i>)
    * [Smoke tests](Brackets Smoke Tests) pass (for larger, cross-cutting changes)
7. Include unit tests for new functionality
8. Avoid breaking API changes — existing public APIs are not strictly frozen, but you'll need a good reason for breaking backwards compatibility. The more commonly-used the API, the stronger the reason needed
9. All user-visible strings are [externalized](Localization)
    * Note: there's a "string freeze" near the end of each sprint. Pending pull requests with string changes must wait until the start of the next sprint.
10. UI is reasonably polished
11. After merging, all new & changed APIs should be documented in the [Release Notes](https://github.com/adobe/brackets/wiki/Release-Notes)

(See also the [Pull Request _Review_ Checklist](Pull-Request-Review-Checklist) that committers follow when doing the code review)

###Common Pitfalls

To avoid problems, consider whether any of these apply to your pull request:

* Text editing commands: what happens at the edges of an inline editor? (not just the edges of the overall document)
* Inline editors: does this collide with any other Quick Edit providers?
* Code hinting: does this collide with any other hint providers?
* Live Preview: ensure [Server smoke tests](Brackets Server Smoke Tests) pass
* Native code (brackets-shell): must include implementations for Mac, Windows _and_ Linux