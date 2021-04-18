By default, **Find in Files** searches all files in your project - the whole tree beneath the root folder that's open in Brackets. You can exclude files, file types, whole folders, or other patterns:

1. Click the "No Files Excluded" dropdown
3. Choose "New Exclusion Set"
2. Enter one or more patterns to exclude

The more files you exclude from your search, the faster Find in Files will run.

To edit an exclusion set later, open the same dropdown and hover over the exclusion set in the list to reveal a pencil icon - click this to edit it.

## About filter patterns

A simple string matches any item whose full path includes that substring:

* `README` matches `/code/README.md`, `/code/modules/README/main.js`, `/code/myREADME.txt`, etc.
* `/node_modules/` matches `/code/node_modules/foo.js`, `/code/node_modules/sub/bar.js`, etc.<br>but does _not_ match `/code/node_modules_two/foo.js`, etc.

> Note: Always use **forward slashes** in filter paths, _even on Windows_. Brackets standardizes paths on all platforms to use "/" separators.

You can also use wildcards:

* `node_*` matches `/code/node_modules/foo.js`, `/code/node_core/foo.js`, `/code/node_foo.txt`, etc.
* `*.txt` matches `/code/readme.txt`, `/code/readme.js.txt`, etc.<br>but does _not_ match `/code/module.txt/foo.js` or `/code/notes.txt2` (filter strings containing "." are treated as filenames, so they won't match folder names or anything else with chars trailing the string you've entered - see below)
* `/jquery*.js` matches `/code/jquery-2.1.0.js`, `/code/foo/jquery-1.7.min.js`, etc.
* `jquery-1.?.js` matches `/code/jquery-1.6.js` but _not_ `/code/jquery-1.6.1.js`

A `*` matches _within_ a path segment (that is, it doesn't match "/" characters). To match "/" as well, use `**`:

* `thirdparty/**/jquery*.js` matches `/code/thirdparty/jquery-1.7.js`, `/code/thirdparty/foo/jquery-2.1.0.min.js`, etc.<br>but does _not_ match `/code/jquery-1.7.js`

(note that the "/"s surrounding a "**" can collapse, as in the first example above)

## Binary Files

Brackets automatically ignores all binary files.

If you have huge numbers of binary files in a project, ensure those file extensions are registered with LanguageManager using `isBinary: true` ([see languages.json](https://github.com/adobe/brackets/blob/master/src/language/languages.json#L236,L260) for defaults) or excluded by your filter - this will be slightly faster than the auto-detection.

## Details

To enable you to write simple filters that don't always include `**`s, Brackets automatically adds `**` using the following rules:

* A `**` prefix is always added - unless the filter string already starts with `**`
* A `**` suffix is added unless the filter string's last path segment contains a "." (suggesting it's a filename) - or unless the filter string already ends with `**`

After this conversion, the filter string must match the entire absolute path of a file for it to be filtered out.