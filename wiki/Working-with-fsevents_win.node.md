To support recursive file watching on Windows, Brackets uses a native Node.js addon, called `fsevents_win`.  This page describes how to build and debug this node module.

Node.js addons are described in more detail here: http://nodejs.org/api/addons.html

#### Pre-requisites

1. Node.js
 * into the root of your c: drive, `git clone https://github.com/joyent/node.git`.  If you choose to clone node into a different folder, you'll need to change the Visual Studio project settings later to point to the correct location.
 * using a DOS command prompt, run `vcbuild release nosign` from the c:\node folder
 * `git checkout v0.10.24` (or whichever version of Node currently used by Brackets)
2. Node-inspector
 * `npm install -g node-inspector`
3. Microsoft Visual Studio 2010 or 2012 (Express)

#### Building `fsevents_win.node`

1. `cd brackets/src/filesystem/impls/appshell/node/node_modules/fsevents_win`
2. to create the VS project, run `node-gyp configure` or `node-gyp configure --debug` to build the Release or Debug builds of `fsevents_win.node`, respectively
3. `node-gyp build`

Note: to use the Debug build, you'll need to edit `fsevents_win.js` and change the line
`var binding = require('./build/Release/fswatch_win');`
to
`var binding = require('./build/Debug/fswatch_win');`

#### Debugging `fsevents_win.node`

1. from a git bash window, run Node Inspector using `node-inspector &`
2. launch Brackets
3. Debug / Enable Node Debugger
4. launch Chrome.  Browse to the URL displayed from Node Inspector after running step (1) above.  This will allow you to step thru `fsevents_win.js`.
5. launch VS 2010.  Open build\binding.sln.
6. Debug / Attach to process...  In the resulting dialog, find node.exe and click Attach.  This will allow you to step thru `fsevents_win.cpp`.
