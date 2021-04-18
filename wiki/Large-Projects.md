To conserve memory, Brackets currently disables some functionality on projects that contain over 30,000 files. This includes most features that search across multiple files, such as **Find/Replace in Files**, **Quick Edit** (inline editors), and **Quick Open**.

You may see the following error message in that case:
> This project contains more than 30,000 files. Features that operate across multiple files may be disabled or behave as if the project is empty. <a href='https://github.com/adobe/brackets/wiki/Large-Projects'>Read more about working with large projects</a>.

## Workarounds

Tips for working around this limit & re-enabling those features:

#### Open a more specific subfolder

Open a more specific subfolder in Brackets to screen out parts of the subtree you don't need to edit. For example...

* If you've been opening the root folder containing _all_ your software projects, instead open the folder for _one specific_ project. You can use the dropdown in the sidebar (just above the folder tree) to rapidly switch between different projects - also accessible via Ctrl-Alt-R (Win) / Cmd-Alt-R (Mac).
* If you've been opening a folder which contains your front-end HTML/CSS/JS code _and_ your backend code in a server-side language, consider opening just the subfolder containing the front-end code. (As above, you can quickly switch folders when you need to work on the backend code).

#### Exclude folders like node_modules

Use the unofficial [Exclude Folders](https://github.com/gruehle/exclude-folders) extension, which removes the `node_modules` folder from Brackets's view of the filesystem. Because npm can easily place 10,000+ files in this folder, this can drastically reduce the number of files Brackets is looking at.

Exclude Folders automatically hides node_modules from Brackets, but you can hack it to exclude other folders as well. 

## Future Brackets improvements

In the near future we hope to provide a clean way to exclude folders directly in Brackets core - no extensions needed.

It's unlikely that the limit on project size will go away soon. Because Brackets is a lightweight code editor and not a large-scale IDE, it's not a high priority to support enormous projects with GBs of files. 