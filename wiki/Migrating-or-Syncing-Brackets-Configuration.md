Brackets does not yet have built-in support for copying or syncing preferences, installed extensions, etc. between two different copies of Brackets.

Please upvote these items on our feature backlog: [sync preferences](https://trello.com/c/RYbXZXn1/368-settings-preferences-synchronization-across-brackets-copies), [sync installed extensions](https://trello.com/c/Y29lYIEL/889-sync-extensions-from-one-computer-to-another).

In the meantime, here's how to manually copy or sync your configuration:

## One-Time Migration

* Select _Help > Show Extensions Folder_
* **Extensions:** Copy the `user` folder to the equivalent location on the new computer
* Go up one level to the parent folder
* **Preferences:** Copy the `brackets.json` file to the equivalent location on the new computer

## Continuous Sync

Replace the folder above with a symlink to a network-shared folder -- for example, a folder on a network drive, or a folder that uses a cloud sync service like Dropbox.  **_However_**, unexpected behavior could occur if Brackets is running while the folder is updated from the network.