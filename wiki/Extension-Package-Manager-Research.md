This is a look at existing extension and package managers to get an idea of the state of the art and whether there is any reusable infrastructure.

## Brackets Extension Manager Extension ##

[jdiehl and DennisKehrig made a Brackets extension](https://github.com/jdiehl/brackets-extension-manager) for extension management.

![Brackets Extension Manager extension](https://www.evernote.com/shard/s24/sh/138a073e-4e70-4e6e-881b-b352fdf9a537/b8c5cab92834571e7aacbde2211a0989/res/37459e7a-a03f-4ebe-92b0-3d989680eb7a/skitch.png?resizeSmall&width=832)

The Extension Manager extension provides a straightforward UI for installing, updating and uninstalling extensions. It requires Node to be installed (it appears that this is largely for file access, though I think the Node server is also responsible for downloading the extension database). The information about the available extensions comes from a static JSON file on jdiehl's github account.

## Package Control ##

The [third-party package manager for Sublime Text](http://wbond.net/sublime_packages/package_control).

Package Control has a minimal user interface:

![Package Control UI](http://wbond.net/sublime_packages/img/package_control/install_package.png)

It leverages Sublime's "Goto Anything" UI to search packages. There are no ratings, reviews or information beyond just the couple of lines that appear there.

No restart is required for any package operations. Packages can include a text file that is displayed in a Sublime buffer when the package is installed or updated (specified in a [messages.json file](https://github.com/wbond/sublime_package_control/blob/master/example-messages.json) in the package repository).

Packages are hosted on GitHub or Bitbucket. In the documentation for [package developers](http://wbond.net/sublime_packages/package_control/package_developers) it is noted that the process for getting a new package in is basically to add your package location to the main repositories json file and issue a pull request.

Package Control automatically creates a version number for packages by looking at the repository update time and it also pulls from the description of the repository on GitHub or Bitbucket. Developers *can* create a custom package.json file with their own version number if they wish (or if their package does not work on all three platforms supported by Sublime).

Packages are automatically upgraded on startup.

## Eclipse ##

Eclipse is built on a foundation of plugins (packaged together as "solutions" that you can install), so extension management is a core feature. It has been a few years since the last time I used Eclipse regularly, so I decided to install the latest and see how it works today.

On the Mac, at least, plugin management seemed a bit more difficult than I expected for a system that is built around plugins. First of all, I looked all over for how to install a new plugin before discovering that it was in the Help Menu:

![Install New Software from Help menu?](https://www.evernote.com/shard/s24/sh/a2912957-6dfc-4e4a-a4c5-c2facc911ba0/757fa0ec5dec1371347d12e3c8e0e89a/res/ec3611e2-7dd1-451e-a7cb-bda902949b9b/skitch.png?resizeSmall&width=832)

Next, you have to select a software site and then you get a filterable, categorized list of available plugins:

![Eclipse Add New Software](https://www.evernote.com/shard/s24/sh/890f1012-c45d-41f4-94a4-bd23eab17ccc/d13a7390830a61d49a80d91bafeb5926/res/94d69521-e6ca-4f73-8da8-227db09aa896/skitch.png?resizeSmall&width=832)

Installing software pops up a modal installation dialog that can be changed to "run in the background". Installation requires a click through license step. I had to trust the 3rd party certificate for the plugin I was installing (PyDev from Aptana). It also required restarting Eclipse.

Once the plugin is installed, you can see it in the list of installed plugins (which it appears you can only access from the Add New Software dialog):

![Eclipse Plugin Manager](https://www.evernote.com/shard/s24/sh/e4cf32ed-ee3f-4fb6-8f3a-8886028d1614/f640f01d9ef0520c769dc8af36078b4f/res/65ed2b13-3a73-43eb-b750-5728b57a39c1/skitch.png?resizeSmall&width=832)

I could uninstall the PyDev plugin easily from there, requiring a restart again (not surprisingly).

You can get more Eclipse plugins from the [Marketplace site](http://marketplace.eclipse.org):

![Eclipse Marketplace site](https://www.evernote.com/shard/s24/sh/1d6f4442-7f58-455b-91c7-eb8d90fbfc3b/4d0685bc5ed843a749c0e15ca996ad38)

The Marketplace site does not enable one click install of plugins. Instead, it provides you with the "Update Site" URL which you can add to your Eclipse installation via the preferences dialog. The site provides "reviews", but it is actually a threaded comments system. Not surprisingly, the first one listed for the "solution" page I looked at (EGit) was not really a review at all but rather someone having trouble with the plugin. Eventually, that thread led to a call to post bug reports in the proper place.

Seeing this reinforces my belief that it needs to be *very* clear what users should do in the event of things just not working and that reviews should be guided to cover just the functionality that is provided by the extension.

## npm and its web interfaces ##

[npm](https://npmjs.org/) is the package manager for Node. It's written in JavaScript for Node, so it may be possible to reuse npm bits, especially after [node integration](https://trello.com/card/5-live-development-on-localhost/4f90a6d98f77505d7940ce88/684) lands in Brackets.

npm's web interface provides some basic statistics (downloads for different time periods, number of packages that depend on a given package). There is a "star" feature, which it appears you have to be logged in to use. Starring of packages does not seem very common as the most-starred package has only 98 stars. underscore is relied upon by more than 2,000 other packages, which shows the value of that measure for Node packages.

npm's site relies on Google to power its search.

The work on npm has definitely centered around its command line utility and its power in allowing Node users to factor out small bits of code into reliably reusable libraries. It has been very successful at this, which may have some applicability to Brackets extensions because the code there may be useful... but Brackets extensions are less like libraries and more like end-user features.

I believe (but have not verified) that npm packages are stored on their server and all published versions of a package are maintained (so that people can require specific versions in their dependencies).

Uploading a new package to npm is straightforward using the [npm publish](https://npmjs.org/doc/publish.html) command.

Note that [npm's server infrastructure](https://github.com/isaacs/npmjs.org) is open source and could be reused.

## component ##

A [package system for browser-based components](https://github.com/component/component). The command line tool for Component is written for Node, but Component does not appear to use npm machinery at all. The idea is that it packages up JS and CSS into a bundle that you can install and depend on.

It's conceivable that we could offer support (through a Node-based extension) for Component, but I don't see anything we need to consider from a package management perspective.

## bower ##

Another [package system for browser-based components](http://twitter.github.com/bower), created by Twitter. Somewhat similar to component, but seems simpler. Uses a `component.json` metadata file that's similar to npm's `package.json`.

There are [more than 1,000 components available for Bower](http://sindresorhus.com/bower-components/). Bower is also used by [Yeoman](http://yeoman.io/).

Seeing all of those packages makes me wonder if some of the extension management UI we create can be reused for managing installed packages for the user's projects.

Registering a new Bower package is done via the `bower register` command, which takes a git endpoint as an argument. There is very little ceremony around this... "Think of it like a URL shortener," according to the Bower documentation.

## Firefox Add-ons ##

Firefox's [add-on manager](https://support.mozilla.org/en-US/kb/find-and-install-add-ons-add-features-to-firefox) and [add-ons website](https://addons.mozilla.org/en-US/firefox/) have a long history and a lot of development behind them.

* "Featured" add-ons
* Categories/Tags
* "Report Abuse"
* "Often Used With"
* Star ratings
* Reviews
* Collections (groups of add-ons that can't be installed together, but can be "followed" and voted up/down. [The Web Developer's Toolbox](https://addons.mozilla.org/en-us/firefox/collections/mozilla/webdeveloper/) is an example). Each add-on's page also shows collections that it's a part of
* Usage statistics (there's a daily ping that checks for updates, so there are stats on how many "daily users" an add-on has)
* Share (via tweets, etc.)
* Screenshots, Icon
* Links for home page, support
* Version info/what's new

![addons.mozilla.org](https://www.evernote.com/shard/s24/sh/ec12aba0-1a86-403d-b2be-7bf9fa3a74ca/91b4cfc0605912def4c20178dd86a4b5/res/3757b3c4-63c6-4106-9671-60410cf988f9/skitch.png?resizeSmall&width=832)

Installing an add-on from addons.mozilla.org is as simple as clicking the "Add to Firefox" button. Some add-ons no longer require a restart of the browser and a drop-down message acts as your confirmation that the add-on has been installed. That same drop-down message is used to prompt the user to restart their browser to see the new functionality for add-ons that do require a restart.

There is also a pop-up dialog with a time delay that forces the user to consider the security implications of installing an add-on. Add-ons can be installed from anywhere, as long as they are served with the proper mime type, but there are different security prompts depending on where you install from.

Add-ons from addons.mozilla.org go through a review process (which is run through a combination of Mozilla employees and community members).

Different kinds of add-ons:

* Traditional add-ons: could access any public API in Firefox and use "XUL overlays" (see also "XBL") to augment the user interface in nearly unlimited ways
* Restartless add-ons: these can do much of what traditional add-ons did, and in the same style of code. However, there are limitations, as [noted by the creator of Adblock Plus](http://adblockplus.org/blog/why-you-should-make-your-next-add-on-restartless). Even so, in that same article, Wladimir says that it is worthwhile to go restartless
* Add-on SDK (Jetpack): Jetpacks are organized with an extensions-specific API to make extension writing easier. Jetpacks are built on CommonJS modules and are restartless. To date, they have been "statically linked": the SDK itself and any libraries your add-on depends on, are bundled in your package. Having the SDK statically linked has proven troublesome, so they are migrating the SDK into Firefox.

All of these add-ons are basically zipped directories. To register a new add-on, developers use the web interface and upload the zip file (which is hosted at addons.mozilla.org).

The Firefox Add-on Manager makes it easy to disable and remove add-ons and also provides access to an add-ons preferences (which can be displayed in any form the add-on wishes... some use XUL windows, some use HTML windows).

![Firefox Add-on Manager](https://www.evernote.com/shard/s24/sh/d9bd61c9-af3a-4122-991e-2708293abfc1/ab3886f131c29bb0f0dbd8fa76fcf754/res/b03e1159-fc7b-41ca-917f-225896b73618/skitch.png?resizeSmall&width=832)

When an add-on is updated, if a restart is required that is noted in the Add-on Manager (with a link that allows for restarting with a single click). Add-ons can display a web page on restart or update.

The Firefox add-on manager has a "Get Add-ons" tab which helps to introduce new add-on users to add-ons, but is also confusing because it shows a few add-ons but does not link in a straightforward way to the addons.mozilla.org site which offers a nicer view of addons. Instead, it offers a direct, in-product stripped down view of add-on information.

## Ubuntu ##

The [Ubuntu Software Centre](http://www.ubuntu.com/ubuntu/features/find-more-apps) is a desktop "app store". It is notable as a graphical front end to [apt](http://en.wikipedia.org/wiki/Advanced_Packaging_Tool) which is a widely used Linux package manager that needs to be able to wrangle dependencies between lots of packages. The package management needed by Linux applications is likely an extreme case of what we would see in a Brackets extension ecosystem (though the Node community has proven with npm that a good package manager can result in the creation of many inter-related packages).

A cursory look leads me to believe that there's nothing especially notable here, relative to the things we can learn from others above.

# Conclusions #

The package management systems outlined above have a variety of features covering:

* discovery
* installation
* updates
* removal
* end user support
* developer management of packages
* package review
* statistics

## Desirable Features ##

Top features:

* Ability to install an extension from anywhere, not just our repository
  * this is good for extensions that are not quite "ready"
  * or, more usefully, for extensions that support proprietary workflows
* One, easy-to-get-at UI for installing extensions from our repository and managing them after installation
* a stable repository so that users don't lose extensions that they've come to rely on
* Users can find extensions via a quick search (minimally a QuickOpen-style search, if not something more advanced) and one click install
* Straightforward workflow for developers to publish and update their extensions
  * One idea that njx and dangoor discussed is the idea that the extension manager could list extensions in the "dev" extensions directory
  * publishing those extensions could be a one-click operation (a publish button rather than a remove button)
* Extensions can provide
  * description (can be Markdown)
  * home page and support links
  * screenshots (optional)
  * version
  * update info (changelog)
  * additional information to display after installation (either a text file or web page) or update
* Extensions needing update can display all (or, ideally part) of the changelog so that users know what they're getting
* Restartless extensions provide the best workflow for installation, update and removal. Updates are especially smooth when they can be done without interrupting the user.
* The extension manager itself should provide a link to get help for an extension that the user has installed and is having trouble with.
* Update checks should be automatic
* Crowdsourced control for extensions that are malicious (flag them) or no longer compatible (probably want some kind of authentication for this to avoid abuse)

Next features:

* Search, ratings, reviews, categories, tags and popularity/download count all help users find new and useful extensions
* Ratings and reviews should require the user to authenticate, which reduces gaming of the system
* Link from a detail preview to other extensions by the same author.
* Users will be more likely to rate and review extensions if they can do so from the extension manager.
* Sort/filter extensions by number of downloads, recently uploaded/updated, or Author.
* When Brackets has a command line, a Sublime Package Control-like interface for managing extensions would be a great *addition* to a Firefox Add-on Manager-like UI, but does not seem like a good substitute.
* Collections of extensions can be handy, especially if we're shooting for an extension ecosystem that provides lots of little features that people can pick and choose from. If one click can install a dozen useful features that work well together, that's a win. [The Janus Vim Distribution](https://github.com/carlhuda/janus) is an example of a combination plugins/configuration that helps people get going with a more productive vim setup.
* A nice-to-have feature would be for an authenticated user's preferences and installed extensions to sync between Brackets installations (possibly even on the web)
* Share extension on Twitter, FB, G+, etc.
* Website (outside of Brackets) for discovering and installing extensions.

Also interesting:

* Bower helps webapp developers reuse browser-based components. Though not directly related to Brackets extension management, it would be neat to provide similar UI for managing components used in the end-user projects.

[nj] From a UX perspective, I think it would be good to aim for something between the bare-bones feel of Sublime (or npm) and the slick, app-store-like experience of Firefox or Ubuntu. We should expose more functionality that aids discovery than Sublime/npm, but avoid feeling too "marketingy". For example, perhaps we shouldn't have icons for extensions; we wouldn't want developers who create useful extensions to feel penalized in their appearance in the extension manager because they don't happen to have access to someone who can create nice artwork for them. On the other hand, we should have a good search mechanism, perhaps a categorization or keyword tagging scheme, and ultimately ratings and reviews, because those all aid discovery in practical ways.

## Traits to Avoid ##

* Package Control is too terse, does not offer ratings/reviews
* Eclipse's solutions manager requires configuration from the preferences dialog and is hidden away in the Help menu. There's no real integration with the marketplace.
* Firefox Add-on Manager provides a less capable version of addons.mozilla.org within it

## Reusable Infrastructure? ##

* npm has powerful dependency management and a reusable server