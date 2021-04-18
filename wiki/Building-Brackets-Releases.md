_(work in progress)_

# Introduction

This page documents build steps for Brackets that are typically _only run on the build servers when building finished installers_.  To modify Brackets locally and submit pull requests, simply follow the [How to Hack on Brackets](https://github.com/adobe/brackets/wiki/How-to-Hack-on-Brackets) instructions instead.

# Prerequisites

* Git command line tools - follow the setup instructions [on GitHub](https://help.github.com/articles/set-up-git) or download [here](http://git-scm.com/downloads)
* Local copy of Brackets repo - see [[How to Hack on Brackets]]
* [Build brackets-shell locally](https://github.com/adobe/brackets-shell/wiki/Building-Brackets-Shell) - the hacking instructions above outline how to point an existing shell, usually from the most recent stable release - but when building a new release, no such download exists yet.
* [Node.js](http://nodejs.org/) 0.8.x or newer
* [Grunt](http://gruntjs.com/getting-started/)
    * Run [Grunt set-up steps](https://github.com/adobe/brackets/wiki/Grunt-Setup)
* Run `npm install` at the root of the Brackets repository

# Build

Run `grunt build` to compile the Brackets `src` tree to the `dist` folder.

To test this build, just open a copy of brackets-shell that hasn't been "setup for hacking" or hold down the shift key as you open Brackets so that it prompts you for the location of the index.html file. Select dist/index.html and you will use the minified copy.

# Usage

After a successful build, ...