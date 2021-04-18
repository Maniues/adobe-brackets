# Live Preview URL Mapping

## Introduction

Live Preview currently opens the local page directly in (Chrome) browser using the file protocol, as in file:///path/to/index.html. This limits Live Preview to only client-side HTML documents using document-relative (&lt;img src="img/photo.jpg" /&gt;) and absolute URLs (&lt;script src="http://ajax.googleapis.com/ajax/libs/jquery/1.8.1/jquery.min.js" &gt;&lt;/script&gt;).

This proposal will allow Live Preview to open pages on a server so paths that have to be interpreted by a server such as site-root-relative (&lt;img src="/img/photo.jpg" /&gt;) and protocol-relative URLs (&lt;script src="//ajax.googleapis.com/ajax/libs/jquery/1.8.1/jquery.min.js" &gt;&lt;/script&gt;) can be used. Also, server-side file extensions such as .php and .shtml pages with include files can also be used in Live Preview.

The backlog entry is [in Trello](https://trello.com/card/3-url-mapping-for-live-development/4f90a6d98f77505d7940ce88/664) and also [brackets/issue #1920](https://github.com/adobe/brackets/issues/1920) (where I borrowed file path examples from).

## Base URL

Base URL maps to the root folder of the project on the server. It is assumed that server path structure mirrors folder structure on disk.

## Preferences

Preferences are project-specific.

A gear icon will be placed next to Project name (and drop down menu) above the Project file tree. Clicking this icon opens a preference dialog which will have the following field:

* A text field to specify Base URL. This is the base URL of the web site on server (e.g. http://localhost/path/to/site/root/).

By default, the field is blank, which indicates that the local path to the file is used in Live Preview.

If URL is specified, then it is used plus project-relative path to file to generate the Live Preview URL.

## Live Preview

If you click Live Development button with file open that has a server-side file extension and a Base URL has not yet been specified, then preferences dialog is auto-opened.

Brackets has a list of server-side file extensions that are recognized in addition to .htm/.html. The default set of server-side file extensions is: .sthm,.shtml,.php,.php3,.php4,.cfm,.cfml. These will be defined globally in package.json, but will not be editable via UI until Brackets gets a Preferences Dialog.

Once base URL specified, Live Development button is clickable while any file server-side file extension, in addition to .htm/.html.

When a Base URL is specified and Live Development button is clicked, the path generated is Base URL + project-specific path to file.
