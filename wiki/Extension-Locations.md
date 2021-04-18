## Introduction

Brackets looks in several different locations to find extensions to load:

### User Extensions Folder

You can view this location using _Help > Show Extensions Folder_. Normally, this folder's contents are managed by Extension Manger.

* Mac: `~/Library/Application Support/Brackets/extensions/user`
* Win: `C:\Users\<user>\AppData\Roaming\Brackets\extensions\user`
* Linux: `~/.config/brackets/extensions/user`

Note: the `disabled` folder that sits next to `user` has no special meaning for the moment. It's just a convenient location to manually move extensions you wish to disable.

### Extension Dev Folder

* `<brackets>/src/extensions/dev`

Normally only used when [pulling source from Git](https://github.com/adobe/brackets/wiki/How-to-Hack-on-Brackets). This location has advantages over the user folder, for extension developers:

* Extension Manager will not let you delete any of the extensions in this location (though they still show up in the Installed list)
* You can conveniently open the entire Brackets source tree along with your extensions, since this folder lies within the source tree. This makes it easier to refer to core Brackets APIs while developing, and gives you improved code hints etc.

More on [[How to Write Extensions]].

### Core Extensions

* `<brackets src>/extensions/default`

Some parts of Brackets core are implemented as extensions internally, and they are located here. Treat this as you could any other part of the Brackets core code: Brackets may not work properly if you remove or change any of these extensions.

> Note: Extensions in this location are not listed in Extension Manager, and Debug > Reload Without Extensions will still load these extensions.