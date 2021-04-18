The new file system module assumes that its underlying file system implementations provide file and directory watching capabilities. That is, when the file system module asks its underlying implementation to watch a given file or directory the implementation should begin to send notifications back about changes to that file or directory in a timely manner. This is a partial list of functionality in Brackets that relies on these change events.

1. Once read, a file's contents are cached in memory until the watcher notifies the file system that the file has changed, at which point the cached contents are evicted. If the watcher fails to send a change notification, Brackets will not recognize the file's contents have changed on disk. This can lead to data loss if Brackets writes to a file that has changed on disk without warning the user.

2. Once read, a directory's contents are similarly cached in memory. Newly created, removed or renamed files in the watched directory won't be available if the watcher does not send change events for the directory. Consequently, the project tree will display the wrong list of file, find in files/project can omit or give inaccurate results, QuickEdit and Jump to Definition may fail to find their dependent resources, etc. This could also lead to data loss if a path is written that is not believed to contain a file, but which actually does.

3. The file system uses an index to attempt to maintain a single object in memory for each watched file. Without change or rename events, it is impossible for the file system to garbage collect or update stale objects after file deletions and renames. At the very least, this can cause memory leaks.


