Download Brackets
====

* Contributors - See [Linux Development For Contributors](https://github.com/adobe/brackets/wiki/Linux-Development-for-Contributors). This is your guide to setting up a development environment, navigating your way around the code base, finding Linux specific user stories to contribute to, and updates on our progress overall.
* Users - Visit [brackets.io](http://brackets.io) for 64-bit and 32-bit debian package downloads.

Feature Status
====

The latest Linux builds (as of Release 1.0), are **practically** at feature parity with the the Mac and Windows builds. However, there are notable features (e.g. Native Menus) that are in various stages of planning and development. To be clear though, Brackets features going forward that only rely on features in the browser can be developed in lock-step on all 3 platforms.

This table lists features that appear in the Linux build but do not function as expected. Please see the [Linux Development For Contributors](https://github.com/adobe/brackets/wiki/Linux-Development-for-Contributors) for progress on these features.

Known Issues
----

| Issue/Feature | Description | Workaround |
| ------------ | ------- | ----- |
| App Doesn't Quit on First Attempt | Brackets doesn't close the first time you click the window close button | Click on close again |
| Native Menus | Native OS menus | N/A. Brackets falls back to HTML menus on Linux |

FAQ
====

1. **I found a bug, where do I file it?** Here [Brackets issues](https://github.com/adobe/brackets/issues)
1. **I want to help, but I don't have any C++ or GTK experience. How can I help?** Start learning C++ and GTK! Otherwise, see the next question.
1. **I can't help on the native shell, but how else can I help?** You can help fix issues (Linux-specific or not) in the www code located in the brackets repository.
1. **Why isn't Linux distribution X supported?**
We're currently focused on Debian/Ubuntu since [Chromium Embedded Framework (CEF)](https://code.google.com/p/chromiumembedded/) is well tested there both in a development environment and in deployment. We're open to contributors helping us *test* and support more distributions.
1. **How do I debug the native shell?** As of August 29, 2013, CEF does not distribute a debug build. Once this is available, we'll post a new answer to this question.
1. **Why is Node.js required for brackets-shell?** With Node.js, we're able to leverage other open source packages to help us build features in Brackets like the HTTP server that we use for Live Preview to instrument HTML files.
1. **How do I use a browser other than Google Chrome?** You can't, for now. For all platforms, Brackets only supports Chrome and Chromium.
1. **Live Development doesn't work with the Chromium browser.  How can I fix it?**  If you want to use the Chromium browser for Live Development, you will need to add symlink the file `/usr/bin/google-chrome` to the chromium executable which is normally located at `/usr/bin/chromium-browser`. All it should take is `sudo ln -s /usr/bin/chromium-browser /usr/bin/google-chrome`.
1. **What IDE or tools should I use to work on brackets-shell?** Anything you want.
1. **I really want to use Brackets in my browser instead of the native shell. How do I do that?** This use case is unsupported currently. However, there is a (somewhat stale) [in-browser branch](https://github.com/adobe/brackets/tree/in-browser/) that implements required file system functionality. See this [Google Group thread](https://groups.google.com/d/msg/brackets-dev/HR4lwxEKt6M/WHj4fcstLwMJ) for an introduction.

Log
----
* 2014-01-05: Removed the known issue with Live Development and restarting Chrome. This was already resolved
* 2013-08-29: Removed Slow Startup, Extension Manager, HTML Highlighting and File rename and delete from known issues. Removed Grunt from FAQ. All fixed in sprint 30.
* 2013-08-09: Added download link
* 2013-06-24: Updated known issues and FAQ. [jasonsanjose](http://github.com/jasonsanjose)
* 2013-06-22: Created. Pointer to dev wiki page. Placeholder for end users until download and documentation plans are in place. [jasonsanjose](http://github.com/jasonsanjose)