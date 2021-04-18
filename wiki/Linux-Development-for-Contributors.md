Introduction
====

Brackets officially supports Mac and Windows. However, we're working on adding Linux support with the help of our open source community. If you're an end user that just wants to try out Brackets on Linux, please be aware that many features are missing or partially implemented.

To be clear, if you are NOT an extension developer and NOT planning to contribute to brackets-shell, please visit [download.brackets.io](http://download.brackets.io) to download a preview build of Brackets. Please review the release notes for known issues.

Development Environment Setup
====

Required Setup for brackets-shell and brackets
----

These instructions will download the Git repositories for [brackets-shell](https://github.com/adobe/brackets-shell) and [brackets](https://github.com/adobe/brackets), download required dependencies, compile the native shell, create and install a debian package, then run Brackets ( `/usr/bin/brackets`).

1. [Fork brackets](https://github.com/adobe/brackets/fork) and [fork brackets-shell](https://github.com/adobe/brackets-shell/fork) 
2. Create a top level folder to contain the Brackets git repositories
3. In a terminal window, ``cd`` to the folder from the previous step and run the following command as root
```shell
wget https://gist.githubusercontent.com/jasonsanjose/5514813/raw/6a522e292f37256b42b697b6da5015e43a6fc2e6/setup.sh; chmod +x setup.sh; bash setup.sh; rm setup.sh
```
4. You'll be prompted for your GitHub user name to clone your fork of the repositories
5. Respond to ``sudo`` password prompts when requested

*ATTENTION* This setup script will point your git ``origin`` to your own fork, then add a remote ``upstream``and point to the ``upstream/master`` branch in the main git repository. Assuming you're here to contribute to Brackets, you will want to point to your fork. This setup script automates the fork and upstream [setup instructions here](https://github.com/adobe/brackets/wiki/How-to-Hack-on-Brackets#setting-up-your-dev-environment).

Cache Location
----

```
# cache root
~/.Brackets

# browser cache, local storage, etc.
~/.Brackets/cef_data

# extensions
~/.Brackets/extensions/user
```

Extension Development
----

The extension development workflow is the same as Mac and Windows. Please refer to [How to hack on Brackets](https://github.com/adobe/brackets/wiki/How-to-Hack-on-Brackets) and [How to write extensions](https://github.com/adobe/brackets/wiki/How%20to%20write%20extensions). Please note that the extension location on Linux is ``~/.Brackets/extensions/user``.

Tested Distributions
----

Contributors: If you are willing to test other Linux distributions please add your results here

| Distribution | Version | Issues |
| ------------ | ------- | ----- |
| Ubuntu | 13.04 64-bit | libudev workaround https://github.com/adobe/brackets/issues/4720 (Fix arrives in Sprint 30) |
| Ubuntu | 12.04 32-bit | None |
| Ubuntu | 12.04 64-bit | libudev workaround https://github.com/adobe/brackets/issues/4720 (Fix arrives in Sprint 30) |
| Mint 15 | 64-bit | libudev workaround https://github.com/adobe/brackets/issues/4720 (Fix arrives in Sprint 30) |
| Mint 16 | 64-bit | No issues besides the close issue. Still Testing |
|LMDE(*) | Debian Testing 32 bit |None|
| Arch | 32/64-bit | Not tested / None - (Package: https://aur.archlinux.org/packages/brackets-git) |
| Arch | 32/64-bit | No issues besides the close issue. Still working around all the features. (Package: https://aur.archlinux.org/packages/brackets-bin/)
| Slackware | 32 & 64 | None (http://slackbuilds.org/apps/brackets/) |

(*) Linux Mint Debian Edition

Building brackets-shell
----

After the script runs to completion, you'll need to get the CEF components. To do this, run:

```
grunt cef
```

To create the `/path/to/brackets-shell/Makefile` run:

```
gyp --depth .
```

If you're only making changes to C++ code, just run `grunt build` and the binaries will be in `/path/to/brackets-shell/out/Release`. 

When adding new files or changing the build configuration, you'll need to make modifications to the GYP configuration files (either `appshell.gyp.txt` or `common.gypi`). After making changes, you'll need to generate a new `Makefile` with gyp (as above).

Packaging Brackets
----

Brackets currently supports 2 options for installation: (A) a Debian .deb package and (B) a portable .tar.gz archive.

### Debian Package

To build a Debian package for installation on Debian/Ubuntu distributions, use the `grunt installer` build task. This tasks copies over both the binaries and the www source and creates a Debian package, e.g. installer/linux/Brackets Sprint 30.deb.

Note that Debian packaging is based on the Chromium browser installer build scripts (CEF 3.1547.1354, Chromium 29.0.1547.41) and shares some 3rd party code. Chromium also provides an RPM script that we could use in the future to implement installers for Fedora/CentOS.

References:
* [Debian Binary Package Building HOWTO](http://tldp.org/HOWTO/html_single/Debian-Binary-Package-Building-HOWTO/)
* [Pull Request](https://github.com/adobe/brackets-shell/pull/297)
* [Chromium trunk installer scripts](http://src.chromium.org/viewvc/chrome/trunk/src/chrome/installer/linux/)
* [Chromium 29 installer scripts](http://src.chromium.org/viewvc/chrome/trunk/src/chrome/installer/linux/?pathrev=214890)
* [Chromium trunk RPM packaging scripts](http://src.chromium.org/viewvc/chrome/trunk/src/chrome/installer/linux/rpm/)

### Archive

To build a portable archive, use the `grunt build-linux-archive` task. This task generates a .tar.gz based off the current Linux build. Use `install.sh` and `uninstall.sh` from the `installer/linux/archive/` directory to install/uninstall desktop and command-line entries for Brackets.

Reference: [Pull Request](https://github.com/adobe/brackets-shell/pull/433)

User Stories
====

There are several user stories (feature work) to complete in brackets-shell before the Linux version reaches feature parity with Mac and Windows. These stories are listed below in priority order

| User Story | Status | Affected Features | Contact |
| ---------- | ------ | ----------------- | ------- |
| [Ubuntu Installer/Packaging](https://trello.com/c/ZoCPy6mD) | In Progress | Install experience | [Jason San Jose](http://github.com/jasonsanjose) |
| [Native Menus](https://trello.com/c/WMB6vtwO) | Not Started | Menus (HTML menus are an interim, but completely functional substitute) | [Ingo Richter](http://github.com/ingorichter)|

Open Issues
====

We tag Linux-specific bugs in our [GitHub issues](https://github.com/adobe/brackets/issues?labels=Linux+only&page=1&state=open).

Development Log
====
* 2013-08-29: Updated for Sprint 30
* 2013-08-20: Update user story status, remove TODOs [jasonsanjose](http://github.com/jasonsanjose)
* 2013-08-09: Edit inline sprint 28 comments [jasonsanjose](http://github.com/jasonsanjose)
* 2013-07-30: Inline draft comments for sprint 28 build [jasonsanjose](http://github.com/jasonsanjose)
* 2013-06-22: Updated ``brackets-shell/linux`` branch with SVG app icon [jasonsanjose](http://github.com/jasonsanjose)
* 2013-06-21: Linux branches land in master in ``brackets-shell`` and ``brackets``. Includes: CEF parity with Mac and Win, stubbed methods for incomplete native shell stories (Node, File I/O, etc.), debian packaging. [jasonsanjose](http://github.com/jasonsanjose)