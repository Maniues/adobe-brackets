Significant API changes were made in Sprint 34 as part of the [File System Evolution](File System) plan:

**Note:** These quick reference tables don't cover arguments or return values in detail. Be sure to check the **detailed JSDoc comments** on the new APIs for full usage details -- see [FileSystem.js](https://github.com/adobe/brackets/blob/master/src/filesystem/FileSystem.js), [File.js](https://github.com/adobe/brackets/blob/master/src/filesystem/File.js), [Directory.js](https://github.com/adobe/brackets/blob/master/src/filesystem/Directory.js), & base class [FileSystemEntry.js](https://github.com/adobe/brackets/blob/master/src/filesystem/FileSystemEntry.js).

* For example, the old APIs used two separate success & error callbacks, while the new APIs use a _single callback_ with combined arguments.

### Backwards-compatible APIs

These APIs are either unchanged or "drop-in compatible" - code using them probably does not need to change at all, though in some unusual usage scenarios the APIs may behave differently from before.

<table>
<thead>
<tr><td><b>Old API</b></td><td><b>New API</b></td><td><b>Notes</b></td><td><b>Usage</b></td></tr>
</thead>
<tr><td>FileEntry<br>DirectoryEntry</td><td>File<br>Directory</td><td>Drop-in compatible (same fields/members)</td><td>(lots)</td></tr>
<tr><td>entry.fullPath, entry.isFile, entry.isDirectory</td><td>(unchanged)</td><td>Properties are now read-only</td><td>fullPath: many<br>isFile/Directory: 9</td></tr>
<tr><td>FileUtils.readAsText(fileEntry)</td><td>(same except takes new objects)</td><td>Drop-in compatible</td><td>13</td></tr>
<tr><td>FileUtils.writeText(fileEntry)</td><td>(same except takes new objects)</td><td>Drop-in compatible</td><td>5</td></tr>
<tr><td>Concatenation: folderPath + "/" + filename</td><td>(unchanged)</td><td>Doubled "/"es are now normalized out by fs APIs</td><td></td></tr>
<tr><td>Concatenation: folderPath + filename</td><td>(unchanged)</td><td>Directory.fullPath always ends in "/", just like DirectoryEntry.fullPath</td><td>Several</td></tr>
<tr><td>Substrings: relpath = fullPath.slice(dirFullPath.length)</td><td>(unchanged)</td><td>Directory.fullPath always ends in "/", just like DirectoryEntry.fullPath</td><td></td></tr>
</table>

### Removed APIs

These APIs are completely gone - code using them will throw exceptions. Extensions using these APIs will be broken immediately, and should be fixed ASAP.

<table>
<thead>
<tr><td><b>Old API</b></td><td><b>New API</b></td><td><b>Notes</b></td><td><b>Usage</b></td></tr>
</thead>
<tr><td>fileEntry.file()... new NativeFileSystem.FileReader().readAsText(fileObj)</td><td>file.read()</td><td></td><td>None</td></tr>
<tr><td>DirectoryEntry.getFile(relpath, {create:true})</td><td>filesystem.getFileForPath(fullPath).write("")</td><td></td><td>2</td></tr>
<tr><td>DirectoryEntry.getDirectory(relpath, {create:true})</td><td>filesystem.getDirectoryForPath(fullPath).create()</td><td></td><td>2</td></tr>
<tr><td>ProjectManager "projectFilesChange" event</td><td>FileSystem "change" event</td><td></td><td>2</td></tr>
<tr><td>fileEntry.getMetadata()</td><td>file.stat()<br>(note: fields of resulting object differ also)</td><td></td><td>1</td></tr>
<tr><td>NativeFileError.*</td><td>FileSystemError.*<br>(different constants)</td><td></td><td>1</td></tr>
<tr><td>error.name<br>e.g. when passing to FileUtils.showFileOpenError()</td><td>error<br>(errors are now string constants)</td><td></td><td>2</td></tr>
<tr><td>instanceof NativeFileSystem.InaccessibleFileEntry</td><td>instanceof InMemoryFile</td><td></td><td>1</td></tr>
<tr><td>entry.remove()</td><td>entry.moveToTrash()</td><td></td><td>None</td></tr>
<tr><td>entry.filesystem</td><td>n/a</td><td></td><td>None</td></tr>
<tr><td>NativeFileSystem.Encodings.*</td><td>(none)</td><td></td><td>None</td></tr>
<tr><td>NativeFileSystem.isRelativePath()</td><td>!FileSystem.isAbsolutePath()</td><td></td><td>None</td>
</tr>
<tr><td>FileUtils.isAffectedWhenRenaming()</td><td>n/a</td><td></td><td>None</td></tr>
<tr><td>FileUtils.updateFileEntryPath()</td><td>n/a</td><td></td><td>None</td></tr>
<tr><td>FileUtils.getFilenameExtension()<br>(this was already deprecated)</td><td>FileUtils.getFileExtension()<br>(note: different return value)</td><td></td><td>None</td></tr>
</table>

### Deprecated APIs

These APIs will _temporarily_ continue to behave as before, but will be removed in the near future. Extensions using these APIs should be fixed relatively soon, within the next sprint or two.

Under the hood, the old API has been reimplemented in terns of the new APIs, so there may be subtle differences noticeable in some edge use cases. (Also - the shim code may be a helpful reference when migrating over to the new APIs).

<table>
<thead>
<tr><td><b>Old API</b></td><td><b>New API</b></td><td><b>Notes</b></td><td><b>Usage</b></td></tr>
</thead>
<tr><td>new NativeFileSystem.FileEntry(fullPath)<br>new NativeFileSystem.DirectoryEntry(fullPath)</td><td>filesystem.getFileForPath(fullPath)<br>filesystem.getDirectoryForPath(fullPath)<br>(You never directly construct a File / Directory)</td><td>These now return a File / Directory object using the new APIs instead of constructing a FileEntry / DirectoryEntry</td><td>19</td></tr>
<tr><td>DirectoryEntry.getFile(relpath)<br>DirectoryEntry.getDirectory(relpath)<br>(*without* 'create' flag)</td><td>filesystem.resolve(dirFullPath + relpath)</td><td></td><td>4</td></tr>
<tr><td>NativeFileSystem.resolveNativeFileSystemPath(fullPath)</td><td>filesystem.resolve(path)</td><td></td><td>4</td></tr>
<tr><td>NativeFileSystem.requestNativeFileSystem(fullPath)... fs.root</td><td>filesystem.resolve(fullPath)</td><td></td><td>6</td></tr>
<tr><td>directoryEntry.createReader().readEntries()</td><td>directory.getContents()</td><td></td><td>5</td></tr>
<tr><td>fileEntry.createWriter()... writer.write(text)</td><td>file.write(text)</td><td></td><td>5 (unused in 3)</td></tr>
<tr><td>NativeFileSystem.showOpenDialog()<br>NativeFileSystem.showSaveDialog()</td><td>filesystem.showOpenDialog()<br>filesystem.showSaveDialog()</td><td></td><td>4</td></tr>
<tr><td>FileIndexManager.getFileInfoList("all")<br>FileIndexManager.getFileInfoList("css")</td><td>ProjectManager.getAllFiles()<br>ProjectManager.getAllFiles(<br>
&nbsp;&nbsp;&nbsp;ProjectManager.getLanguageFilter("css") )</td><td>Returns an array of Files, but they provide same properties as the old FileInfos</td><td>7</td></tr>
<tr><td>FileIndexManager.getFilenameMatches("...", filename)</td><td>ProjectManager.getAllFiles(filter)</td><td></td><td>None</td></tr>
<tr><td>brackets.fs.*()</td><td>(various)</td><td>These low-level APIs continue to exist, but have never been officially supported. They may break at any time.</td><td>Several</td></tr>
</table>

### Extensions that will break

The following 9 extensions use at least one API on the "Removed" list:

* angularui.angularjs -- (FileEntry.getMetadata())
* bsirlinger.github-access -- (DirectoryEntry.getFile({create}), DirectoryEntry.getDirectory({create}))
* xunit -- (DirectoryEntry.getFile({create}), DirectoryEntry.getDirectory({create})
* zaggino.brackets-git (won't crash, but notable bug) -- ("projectFilesChange" event)
* variousimprovements (won't crash, minor bug) -- ("projectFilesChange" event)
* camden.caniuse (won't crash, minor bug) -- (FileError.name)
* jrowny.brackets.snippets (won't crash, minor bug) -- (FileError.name, though was buggy already)
* pflynn.brackets.editor.nav (won't crash, very minor bug) -- (InaccessibleFileEntry)
* interactive-linter (no effect on behavior) -- (NativeFileError)

We will file bugs with all extensions that are using removed APIs to make sure authors are aware of the change.