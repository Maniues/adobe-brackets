# Brackets Command Line Tools

**Applies to:** Brackets 1.3 and newer

Mac and Windows users can open files and folders in Brackets from the command line, if optional command-line shortcuts are set up:

* **Mac:** Select _File > Install Command Line Shortcut_ in Brackets
* **Windows:** When installing Brackets, ensure _Add "brackets" launcher to PATH for command line use_ is checked off

## Using Brackets from the Command Line

Type `brackets` at the Terminal (Mac) or Command Prompt (Windows) to launch Brackets from any location.

To open file(s), add the filenames as command-line arguments; paths can be relative to the command prompt's current directory. If Brackets is already running, the file(s) will open there instead of launching another copy of Brackets. Filenames that include spaces must be wrapped in quotes on Windows.

To open a project, add the folder name as a command-line argument. Currently, Brackets only opens one project at a time, so if Brackets is already running it will switch to this folder. (Any unsaved changes in the previous project are confirmed first).

#### Examples

Open a file in the current folder, without switching projects:<br>
`brackets index.html`

Open _two_ files from a specific path, without switching projects:<br>
`brackets ~/Sites/index.html ~/Sites/index.css`

Open the current folder as a project in Brackets:<br>
`brackets .`

Open a specific folder as a project in Brackets:<br>
`brackets ~/Sites/mysite`

## Removing Command Line Tools

**Windows** - Uninstalling Brackets reverts the PATH changes. You can also manually edit your path this way:
1. Right-click My Computer and choose _Properties_
2. Click _Advanced System Settings_
3. In the Advanced tab, click the _Environment Variables_ button
4. Select `PATH` from the first list you see it in, then click Edit
5. Remove the segment referencing Brackets, including

**Mac** - Delete the file `/usr/local/bin/brackets`

## Troubleshooting

If the automatic setup described at the top of this page doesn't work, please [file a bug](https://github.com/adobe/brackets/issues/new) with details so we can improve Brackets.

You can also manually create shortcuts to launch Brackets from the command line.

**Windows** - Follow the steps above to edit your `PATH` environment variable, and add `;%ProgramFiles(x86)%\Brackets\command` to the end. Close and reopen any Command Prompts you have open. You may also need to restart your computer for the change to take effect.

**Mac** - Manually run `sudo ln -s /Applications/Brackets.app/Contents/Resources/brackets.sh /usr/local/bin/Brackets` and type your admin password to confirm. (If Brackets is not in `/Applications`, substitute the correct path).

## Multiple Versions of Brackets

If you have multiple copies of Brackets on your computer, you can control which version will be launched:

**Windows** - The command line will launch Brackets from whatever folder it was installed to the last time the Brackets installer was run. To change this, manually edit your PATH following the steps above.

**Mac** - The command line will launch whatever copy of Brackets the _Install Command Line Shortcut_ command was run in. To change this, rerun the command from a different copy of Brackets.

## Linux

Linux support is still in progress - Brackets can be _launched_ from the command line, but it does not accept arguments.