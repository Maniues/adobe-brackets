The front-end client-facing `FileSystem` API is decoupled from the back-end filesystem _implementation_ in order to accommodate different kinds of filesystems without changing Brackets core code. For example, accessing remote files stored on Dropbox or SkyDrive. When [running Brackets in-browser](https://github.com/adobe/brackets/wiki/Brackets-in-Browser), the implementation acts as the bridge between Brackets and your server backend -- Brackets calls the impl (running client-side in the user's browser), and in turn the impl talks to the server on Bracket's behalf.

This page describes the requirements for implementing a filesystem back-end implementation.

## `FileSystemImpl`

The client-facing filesystem API is provided by a singleton `FileSystem` object. This object has a single method, `FileSystem.init`, to initialize the filesystem with a particular filesystem implementation, which is responsible for core functionality like reading and writing files, and providing file and directory change notifications. The implementation object should satisfy a `FileSystemImpl` interface, as described below.

### `showOpenDialog(allowMultipleSelection, chooseDirectories, title, initialPath, fileTypes, callback)`
* `@param {boolean} allowMultipleSelection`
* `@param {boolean} chooseDirectories`
* `@param {string} title`
* `@param {string} initialPath`
* `@param {Array.<string>=} fileTypes`
* `@param {function(?string, Array.<string>=)} callback`
* Display an open-files dialog to the user and call back asynchronously with either an error string or an array of path strings, which indicate the file or files chosen by the user.

### `showSaveDialog(title, initialPath, proposedNewFilename, callback)`
* `@param {string} title`
* `@param {string} initialPath`
* `@param {string} proposedNewFilename`
* `@param {function(?string, string=)} callback`
* Display a save-file dialog to the user and call back asynchronously with either an error or the path to which the user has chosen to save the file.

### `exists(path, callback)`
* `@param {string} path`
* `@param {function(?string, boolean)} callback`
* Determine whether a file or directory exists at the given path by calling back asynchronously with either an error or a boolean, which is true if the file exists and false otherwise. The error will never be `FileSystemError.NOT_FOUND`; in that case, there will be no error and the boolean parameter will be false.

### `readdir(path, callback)`
* `@param {string} path`
* `@param {function(?string, Array.<FileSystemEntry>=, Array.<?string|FileSystemStats>=)} callback`
* Read the contents of the directory at the given path, calling back asynchronously either with an error or an array of FileSystemEntry objects along with another consistent array, each index of which either contains a FileSystemStats object for the corresponding FileSystemEntry object in the second parameter or a FileSystemErrors string describing a stat error. 

### `mkdir(path, mode, callback)`
* `@param {string} path`
* `@param {number=} mode`
* `@param {function(?string, FileSystemStats=)=} callback`
* Create a directory at the given path, and optionally call back asynchronously with either an error or a stats object for the newly created directory. The octal mode parameter is optional; if unspecified, the mode of the created directory is implementation dependent.

### `rename(oldPath, newPath, callback)`
* `@param {string} oldPath`
* `@param {string} newPath`
* `@param {function(?string)=} callback`
* Rename the file or directory at `oldPath` to `newPath`, and optionally call back asynchronously with a possibly null error.

### `stat(path, callback)`
* `@param {string} path`
* `@param {function(?string, FileSystemStats=)} callback` 
* Stat the file or directory at the given path, calling back asynchronously with either an error or the entry's associated FileSystemStats object.

### `readFile(path, options, callback)`
* `@param {string} path`
* `@param {{encoding : string=}=} options`
* `@param {function(?string, string=, FileSystemStats=)} callback`
* Read the contents of the file at the given path, calling back asynchronously with either an error or the data and, optionally, the FileSystemStats object associated with the read file. The optional `options` parameter can be used to specify an encoding (default `"utf8"`).

### `writeFile(path, data, [options], callback)`
* `@param {string} path`
* `@param {string} data`
* `@param {{encoding : string=, mode : number=}=} options`
* `@param {function(?string, FileSystemStats=)} callback`
* Write the given data to the file at the given path, calling back asynchronously with either an error or, optionally, the FileSystemStats object associated with the written file. The optional `options` parameter can be used to specify an encoding (default `"utf8"`) and an octal mode (default unspecified and implementation dependent). If no file exists at the given path, a new file will be created.

### `unlink(path, callback)`
* `@param {string} path`
* `@param {function(string)=} callback`
* Unlink the file or directory at the given path, optionally calling back asynchronously with a possibly null error.

### *optional* `moveToTrash(path, callback)`
* `@param {string} path`
* `@param {number} mode`
* `@param {function(?string)=} callback`
* Move the file or directory at the given path to the "trash" directory, optionally calling back asynchronously with a possibly null error.

### `initWatchers(changeCallback, offlineCallback)`
* `@param {function(?string, FileSystemStats=)} changeCallback`
* `@param {function(?string)=} offlineCallback`
* Initialize file watching for this filesystem. The implementation must use the supplied `changeCallback` to provide change notifications. The first parameter of `changeCallback` specifies the changed path (either a file or a directory); if this parameter is null, it indicates that the implementation cannot specify a particular changed path, and so the callers should consider all paths to have changed and to update their state accordingly. The second parameter to `changeCallback` is an optional `FileSystemStats` object that may be provided in case the changed path already exists and stats are readily available.
* If file watching becomes unavailable or is unsupported, the implementation must call `offlineCallback` if it was provided, optionally passing an error code. In addition, the implementation _must_ ensure that all future calls to `watchPath()` fail with an error (until such time as file watching becomes available again).

### `watchPath(path, callback)`
* `@param {string} path`
* `@param {function(?string)=} callback`
* Start providing change notifications for the file or directory at the given path, optionally calling back asynchronously with a possibly null error when the operation is complete. Notifications are provided using the `changeCallback` function provided by the `initWatchers` method. If the path is a directory, the expected behavior depends on the implementation's `recursiveWatch` flag: if true, notifications are expected for the entire subtree rooted at this directory; if false, notifications are expected only for the directory's immediate children.

### `unwatchPath(path, callback)`
* `@param {string} path`
* `@param {function(?string)=} callback`
* Stop providing change notifications for the file or directory at the given path _and all subfolders_, optionally calling back asynchronously with a possibly null error when the operation is complete. Unlike `watchPath()`, this is _always_ expected to behave recursively.

### `unwatchAll(callback)`
* `@param {function(?string)=} callback`
* Stop providing change notifications for all previously watched files and directories, optionally calling back asynchronously with a possibly null error when the operation is complete. 

### `recursiveWatch`
* `@type {boolean}`
* Indicates whether calls to `watchPath()` begin watching the entire subtree, or just the immediate children of the given path. If false (`watchPath()` does not watch recursively), the FileSystem will automatically call `watchPath()` on each subfolder in turn, as needed. This flag must not change value at runtime.

### `normalizeUNCPaths`
* `@type {boolean}`
* Indicates whether or not the FileSystem should expect [UNC paths](http://www.uwplatt.edu/oit/terms/uncpath.html), like `//myserver/drive/folder`. If set, contiguous blocks of leading slashes as in the previous example are normalized to a pair of leading slashes instead of a single leading slash. This flag must not change value at runtime.


## `FileSystemStats` and `FileSystemError`
The stats objects passed to callbacks above are instances of the [`FileSystemStats` class](https://github.com/adobe/brackets/blob/glenn/file-system/src/filesystem/FileSystemStats.js), and the possibly null error parameters are constants defined in the [`FileSystemError` class](https://github.com/adobe/brackets/blob/glenn/file-system/src/filesystem/FileSystemError.js).


## Designating the Implementation to Use

FileSystem expects to be able to load its impl via `require("fileSystemImpl")`, so the RequireJS config that loads FileSystem must define a mapping from this identifier to the impl's full module path. See the [root main.js in Brackets](https://github.com/adobe/brackets/blob/master/src/main.js) for an example.