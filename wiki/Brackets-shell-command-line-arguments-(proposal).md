# Handling Command Line Arguments

----
<font color="red">**Note:**</font> This is a _proposal_ for future functionality. The command-line arguments described here are _**not** currently implemented in Brackets_.

----

<br>_**Question**: Should this be merged with [[Command Line Arguments]]?_

## Overview
Generally, applications are able to process arguments specified by the user upon launch.  These arguments often can be used to configure the application, retrieve information from the application or instruct it to perform various operations upon initialization.

This spec describes an initial set of arguments that Brackets supports, with a general focus on exposing existing features.  It also describes the general framework in which the command line arguments are handled.  As additional arguments are needed in the future, they should be added to this document.

###  General Description

**Specifying a File Path**

   A single file path can be included as a Brackets command line argument.  If the path is to a File that is supported by the application then it will be opened for editing, in the same way it would have been had the user selected it in the dialog for the File->Open command.  If the path is to a directory on the filesystem, the folder will be displayed in the side bar.  This is analogous to having selected the folder interactively via the File->Open Folder command.

   Note that the file path specified can be relative to the current working directory.  Finally, any paths that are specified which do not exist on the filesystem will be ignored.


**Examples**

`Brackets.exe c:\MySite\index.html`

`./mac/Brackets.app/Contents/MacOS/Brackets ~/website/styles.css` 

`open Brackets.app ./website/scripts` 

### Scope

   Note that File extension/shell integration is enabled by this feature.  However, this entails work that is above and beyond the initial command line handling.  For example, once the file argument was implemented it would be possible to register with the Win32 registry keys that would allow Brackets to open double clicked HTML/JS/CSS files.
 

## Implementation Notes

CEF has a utility class named [CefCommandLine](http://magpcss.org/ceforum/apidocs3/projects/%28default%29/CefCommandLine.html) that we believe  will be sufficient for our purposes.  If it isn't, there are open source alternatives available.  One of these is the **Boost** [program.options](http://www.boost.org/libs/program_options/) library.

## Questions

### Extension Concerns

   It would be beneficial for Extensions to receive notification of the command line options, or perhaps have a mechanism where the arguments could be retrieved.  

   Do we want to have a scheme where extension specific arguments could be specified?  For example, given the HelloWorld extension:

`Brackets.exe -ext:HelloWorld="Some Data for the extension only" c:\temp\foo.html`

### Communication with existing Brackets Process

Note that on Windows, it is possible to launch multiple instances of an EXE.  Logic can be added to the Win32 startup sequence that would allow the application to use inter-process communication.  Do we want Brackets to follow the model where there can be multiple concurrent processes running.  Or do we want to add logic that would use the newly launched Brackets process to pass data to the running instance.  

For example, if Brackets is already running and the user attempted to start another one and passed a file to open, we could make it so that the file information is passed to the other brackets process and the newly launched one would terminate once the message has been handled.