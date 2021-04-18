For the current state of language support, see page [Language Support](Language Support). This page documents what additional capabilities extensions would need in order to add language support on par with the current level of support for HTML, JavaScript and CSS. 

Our goal is to base all language-related features like code hinting, text manipulation commands, quick edit, live preview, etc. on these capabilities. Support for non-web programming languages will be moved to dedicated extensions (see [issue #2969](https://github.com/adobe/brackets/issues/2969)).


## Providing code semantics

We need to allow adding electric __strings__ to language definitions*. Strings instead of chars because some languages have block boundaries that are not just one char long, for instance Ruby's `begin ... end`.

*From [CodeMirror's manual](http://codemirror.net/doc/manual.html): "electricChars: boolean - Configures whether the editor should re-indent the current line when a character is typed that might change its proper indentation (only works if the mode supports indentation). Default is true."

## Starting Live Preview

In general the possibility of live development for a given file depends on its format, the formats we can turn it into and the formats a client supports.

### New concept: clients

Consequently we need ways to define clients and the formats they support. Right now the only supported client is Chrome. We want to extend this to other browsers, and may eventually want to extend this to different types of clients, like Node.js, or a PDF viewer to preview LaTeX files.

Also, we need ways to define what formats the project's files are available in. In principle, we could use the language concept for this, but since browsers usually declare support for formats via a list of MIME types (e.g. "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8") and since web servers use MIME types in the response's header ("Content-Type: text/html; charset=utf-8"), we should allow adding MIME types to languages and use those instead.

__MIME type for HTML in languages.json:__

    {
        "html": {
            "name": "HTML",
            "mimeTypes": ["text/html"],
            "fileExtensions": ["html"]
        }
    }

__MIME types supported by Chrome in clients.json:__

    {
        "chrome": {
            "mimeTypes": ["text/html", "application/xhtml+xml", "application/xml"]
        }
    }

If the user opens "index.html" and clicks the live preview button, Brackets would know that Chrome supports this file and allow the live preview.

### Auto-detecting the format of a server-generated file

If the user provides a base URL, opens "index.php" and clicks the live preview button, Brackets could send a HEAD request for index.php to the server. If the returned MIME type is "text/html", Brackets would know that Chrome supports this PHP file's output and could allow the live preview. Otherwise, the live preview would not be enabled. The same is true for other server-side file extensions.

This way, we would not need to maintain a list of file extensions that may or may not generate content in a format supported by Chrome.

### Extending clients

Extensions should be able to add new MIME types to languages and clients:

    var ClientManager = brackets.getModule("LiveDevelopment/ClientManager"),
        chrome = ClientManager.getClient("chrome");
    chrome.addMimeType("image/svg+xml");

    var LanguageManager = brackets.getModule("language/LanguageManager"),
        svg = LanguageManager.getLanguage("svg");
    svg.addMimeType("image/svg+xml");

If the user opens "index.svg" and clicks the live preview button, Brackets would know that Chrome supports this file and allow the live preview. This would also work if index.php generated SVG code, assuming it provides the proper MIME type in the Content-Type header.

SVG is an interesting use case since Brackets would right now provide live development with SVG if only its extension were listed in the `_staticHtmlFileExts` array. Brackets would show XML code, the browser would show the rendered image, Brackets would reload the browser when saving the file, and even refresh styles as they are changed if the SVG file links to external stylesheets (`<?xml-stylesheet type="text/css" href="style.css" ?>` before the opening `<svg>` tag).

### Supporting derivatives (compilers)

Currently we only support the format a file comes in, but we could extend live development to files that could be converted to a format supported by the client.

__Registering an HTML compiler for Markdown:__

    var LanguageManager = brackets.getModule("language/LanguageManager"),
        md = LanguageManager.getLanguage("markdown");
    md.addCompiler("html", function (url) {
        var deferred = new $.Deferred();
        ...
        var mainFile = {
            url:          url.replace(/[^\.]*$/, "html"),
            lastModified: lastModified,
            content:      htmlCode,
        };
        deferred.resolve({
            files: [mainFile],
            mainFile: mainFile
        });
        ...
        return deferred.promise();
    });

If the user opens "index.md" and clicks the live preview button, Brackets would know that it could compile this code to HTML. It would look up HTML's MIME type and see that Chrome could open the compiled file. It would determine a URL to the Markdown file as seen from Chrome by calling `client.urlForDocument(doc)` and  pass it to the compiler. The compiler would somehow access the file using the URL, possibly via an XMLHttpRequest. It would resolve its promise with a list of generated files. Brackets would make these  files (so far, in memory only) available - either by writing them to the hard disk as a temporary file or by registering their URLs with the live preview server. Finally, Brackets would start a live preview using the URL to the compiled file. On a related note, [card 565](https://trello.com/c/4BHJSfzo) introduces the idea of HTML rewriting.

__Question:__ What should we pass to a compiler - a Document instance, a URL or a string with code?

If a LESS file imports another LESS file, the included LESS file either needs to be referenced with an absolute URL or LESS needs to be provided with a list of search paths. If the only search path is relative, LESS will use window.location.href as the base URL to figure out an absolute path. LESS uses this path to load the imported file with an XMLHttpRequest. Consequently, providing "./" as the search path would only work if the main file were located in Brackets' src directory.

Therefore, providing just a string with the content of the main LESS file is not enough. LESS also needs an absolute path to the directory the file is in. Then it can access all imported files. For live development on save, this is good enough, as long as the included file does not contain relative references to background images, fonts, etc. LESS would prefix the URLs to these with the path to the main file's directory. Since Brackets is loaded from the local file system, this would be a local file URL. The background images in the browser would then be loaded from the local hard drive. If we ever want to support live development on smartphones or other remote devices, this would no longer work. Instead, the URL as seen from the client should be used.

When LiveDevelopment is active, this URL is available via doc.url. If we want the compiler architecture to be more universal, say to compile LESS files provided by extensions, then doc.url wouldn't be set for such a file. Basically, a URL always relates to a client, so we would need something like `doc.urlForClient(client)` or `client.urlForDocument(doc)` (the latter seems preferable, but then we need a client object representing Brackets).

There's another problem: if a Document instance is passed, `doc.getText()` would be used to get the file's content. If the user has changed the document without saving, the unsaved version would be retrieved. This is what should happen for live development on change, but may not be wanted in other circumstances. In addition, LESS would still try to load referenced files via an XMLHttpRequest, normally not getting the unsaved content of the file. In some cases, we may be able to configure the compiler to use a source of our own choosing. LESS is hard to customize in this respect - while it is possible to overwrite less.Parser.importer, this function is also reponsible for parsing the resulting file. Our own implementation of the importer would therefore need to do more than just retrieve files and would be harder to keep in sync with updates to LESS. Effectively, we would provide our own version of the LESS compiler, which may be necessary for other languages as well.

An alternative would be to provide a URL to an HTTP server that delivers files straight from the Document objects in Brackets' memory. Compilers could then continue to use XMLHttpRequest. Unless the live development server uses Document instances, the resulting URLs in the CSS file (for background images, etc.) would need to be adjusted to access the actual live development server. If the user provides a base URL, we do not start our own server and may need to do this mapping anyway. One alternative to this would be to always launch our own live development server and turn it into a proxy if a base URL is provided. This would also allow us to continue rewriting HTML documents even if we do not read them from the hard disk directly, but from the server the user provided.

__Question:__ How should a compiler deliver its output?

In many cases, the compiler will only generate one file, even if it uses many input files (LESS with included files, a minifier). However, in many of these cases it also makes sense to generate a SourceMap alongside the core output file. The compiler could write these files to the hard disk directly, but this would prevent us from generating and serving files in memory (say, a Markdown preview in HTML format). Instead, the compiler should provide its outputs as an array of objects. The objects would contain the generated file's content, a URL to store them at, and possibly semantic information like `"SourceMap"` or "Main" to ease programmatic use of the output.

__Summary:__

Compilation isn't a simple 1:1 mapping of input to output. Any number of inputs can be mapped to any number of outputs. In addition, the output may contain references to files. These references can depend on the locations of the inputs. Therefore, a compiler needs a lot of information to produce the right output depending on the context. Variables are whether to use the saved or unsaved version of files, whether the output is saved to disk or provided from memory, and the base URL of the client that will consume the outputs.


### Adding new clients

Clients do not necessarily have to be external applications, a Brackets extension could just add a DIV to Brackets UI and render a file into it. Through the ClientManager, it could define a client in a structured manner.

__Embedded HTML Live Development client:__

    var ClientManager   = brackets.getModule("LiveDevelopment/ClientManager"),
        LanguageManager = brackets.getModule("language/LanguageManager"),
        $preview        = ...;

    var client = ClientManager.addClient("preview", {
        mimeTypes: ["text/html"],
        urlForDocument: function (doc) {
            // This method doesn't exist yet, but turning a path into a URL involves more
            // than adding a prefix, so there should be a utility function somewhere.
            return doc.file.getUrlForFullPath();
        },
        open: function (url) {
            // url would be something like "http://localhost:12345/markdown.html"
            $.get(url, function (content) {
                // ... extract the <body> ...

                // Fill and show the preview frame in Brackets
                $preview.html(content).appendTo(...);
            });
        },
        close: function () {
            // Remove the preview frame
            $preview.remove();
        }
    });

This client would only support "reloading" by calling client.open again. For a more elaborate updating behavior, clients would need to declare support for such operations in a structured manner.

## Updating Live Preview

For live development on _save_, the client needs to allow reloading the document. It also needs to reveal which files were requested by the document. This allows Brackets to reload the document if an included file is changed. For Chrome, the network agent handles this task by listening to the "Network.requestWillBeSent" event. We may however want to augment this with offline detection routines to detect server-side file inclusions (to reload index.php if shared.php is modified). Since those mechanisms can use arbitrary amounts of magic (i.e. [autoloading](http://php.net/manual/de/language.oop5.autoload.php) or [auto_prepend_file](http://www.php.net/manual/en/ini.core.php#ini.auto-prepend-file)), supporting all or even most scenarios is unrealistic. Instead, we could introduce a standard for server-side scripts to announce a list of included files in the response header. The community could provide the necessary plugins for server-side languages. For PHP, the function [get_included_files](http://php.net/manual/de/function.get-included-files.php) would provide an easy starting point.

For live development on _change_, the client needs to provide ways to directly inject the new content into the document. In Brackets, we need to be aware of such methods and use them instead of reloading the whole document. Extensions therefore need a way to register an updater. We currently provide this in a hard-coded way for CSS, and for HTML and JavaScript files if `LiveDevelopment.config.experimental` is true.

The following scenarios showcase what needs to change, using LESS as an example.

### External Precompilation

In this scenario, the LESS file is converted to a static CSS file by an external tool. This tool might monitor the LESS file for changes, or the user could run it manually. The resulting static CSS file is delivered to the browser in the usual way. Therefore, it can also be updated like normal CSS files.

To support this, we would need to detect external changes to included CSS files even if the corresponding CSS file is not open in Brackets. Such an external change would then need to be treated like the user saving the file.

Updating on change is not possible in this scenario since the external tool only has access to the content of the file through the file system.

### Internal Precompilation

In this scenario, the LESS file is converted to a static CSS file by Brackets. The resulting static CSS file is delivered to the browser in the usual way. Therefore, it can also be updated like normal CSS files.

To support this, the user would need to be able to turn this behavior on, possibly for individual files only since LESS files can include other LESS files, and typically only the master file should be converted.

However, the [BracketsLESS extension](https://github.com/olsgreen/BracketLESS) adds a menu entry to turn compilation on for all LESS files. Compilation occurs when the file is saved. Sadly, Brackets currently doesn't detect that the CSS file was changed, not even when switching to it from within Brackets after saving the LESS file. Switching to another application and back to Brackets will update the CSS file in the browser if at one point a Document object has been created for the CSS file (for instance by opening the CSS file within Brackets). There also is no straightforward way to create the file via the Document API in the first place. Even if the file already existed (and the Document object can therefore be created), there is no `doc.save()` method to call since all writing is apparently done via Editor instances. There is also no public API to access a document's editor. Even if there were, there is no `editor.save()` either - the code to save a file is only available through the File > Save command and therefore only available for the currently opened document. Clearly, programmatic changes to documents need to be made easier. For now, saving the file manually, then getting a Document for it and calling `$(window).triggerHandler("focus");` is the most convenient way to make Brackets aware of the changed file so the preview is updated accordingly.

Ideally, this extension could simply open a Document object with the target path and replicate all changes of the LESS file to the CSS file. If the LESS file is changed, `cssDoc.setText(cssCode)` is called, and the live preview is updated as if the user had typed these changes into the CSS file manually. If the LESS file is saved, so is the CSS file, similarly with deleting the LESS file.

### Client-side compilation

In this scenario, the browser includes less.js and references the LESS stylesheets in `<link>` tags with the MIME type "stylesheet/less". LESS finds these references and downloads the files with `XMLHttpRequest`s. Consequently, the network agent regards the LESS files as requested, causing the HTML document to reload when the LESS document is saved.

For better live development support, Brackets would need to compile the LESS code to CSS and update the corresponding CSS code in the browser.

There are three critical components to this:
* The compiler to convert LESS code into CSS code (could be requested via `doc.getLanguage().getDefaultCompilerToLanguage("css")`)
* The updater to find the `<link>` tag that referenced the LESS file, determine the ID of the corresponding `<style>` tag, generate the CSS code (using the compiler), and change the content of the `<style>` tag
* The live development module to allow registering the updater, trigger it at the right time and avoid reloading the page if the updater was successful

### Server-side compilation

In this scenario, the server compiles LESS code to CSS on the fly. There are various conceivable ways to implement this. The URLs for the file might look like this:

* `style.less` - same file name, but transparently returns the CSS version
* `style.less?compile=true` returns the CSS version while `style.less` will return the file as-is
* `style.less.css` returns the CSS version
* `style.css` returns the CSS version
* `compiled/style.less` returns the CSS version
* `compiled/style.css` returns the CSS version
* `compile.php?file=style.css` returns the CSS version
* `all_styles.css` - returns various styles combined into one file

If the file name is not modified (and even if a query string is appended to its URL), the network agent will already declare the file as requested. The network agent could detect usage of the two conventions that only modify the file extension by checking whether the file exists on the server. If not, it could conclude that the server generates derivatives of the current file for these URLs.

If the network agent also listens to the "responseReceived" event, it will be able to report the MIME type as well. We can then check whether there is a compiler for the current document's language (LESS) to a language that has the same MIME type as returned by the server. If so, we can update the file ourselves whenever it is changed in the same way normal CSS files are updated.

However, this might fail if the server compiler uses special settings, like a search path to use when including other LESS files in a LESS file. A safe compromise would be to only update the stylesheet after it has been saved: in this case, the compiled document can simply be requested by the server and we would only inject the result into the running page.

An alternative with other risks would be to transparently move the saved version to a backup location, silently save the changed version in the editor, request the compiled version from the server and restore the saved version from the backup. If either Brackets or the whole computer crash in the middle of that process, or a cloud storage service watches the directory, this approach could result in data loss.


## Showing the context in Live Preview

Showing the context consists of five phases: **injecting**, **identifying**, **locating**, **styling** and **highlighting**.

**Injecting** means enhancing the data delivered to the client by injecting information that makes elements easier to identify and locate. For instance, we rewrite HTML pages to insert explicit IDs for every tag to be independent of later modifications to the DOM tree. In addition, JavaScript code is injected to provide an API for drawing the highlights based on CSS selectors. To locate context elements in JavaScript files, wrapping API calls can be necessary to trace the effects of a function.

**Identifying** is about describing elements of the document that are related to the current cursor position. In an HTML file, this could be an XPath or a CSS selector describing the DOM node that got created from the closest tag around the cursor. In a CSS file, it could simply be the CSS rule around the cursor. In a JavaScript file, the context contains all the elements on the page that got affected by the closest function around the tag. It could be described by CSS selectors for the nodes that got created, modified, moved or styled by the function or that use the function as an event handler. It could also be the shape and position of areas in a `<canvas>` element that got drawn by the function. Similarly, in a LaTeX file, the context could be a shape with a page number and a position that happens to outline the area of the PDF file that got created from the code at the current cursor position.

**Locating** is about using the description to get a handle on the context elements in the rendered document. Browsers usually provide convenient lookup methods like `document.querySelectorAll()`, though extra work may be necessary (see Injecting). By determining which part of the document the client is showing and where the context elements are positioned, the locator can make sure that the client shows at least one such element (if there is one).

**Styling** is about the visual aspects of highlighting, i.e. what shapes and colors to use, whether to use animations, etc. The user may want to be able to configure this, or install an extension for a different highlighting style. Currently we highlight the context of a CSS rule by shading the background of affected nodes with a transparent blue background and a blue border around the element. If the user adds a one pixel wide red border around the element, they won't see the result until changing the context to a different element because the blue border of the highlight covers the red border created by the user. Similarly, every background color set by the user is affected by the highlight. The user may want to install a styler that instead dims everything around the targeted elements.

**Highlighting** is about enhancing the visual representation of located elements by applying the styling rules. In the most basic case, the locator provides just the position and size of a rectangle within the content area of the client, and the highlighter draws a rectangle at these coordinates. The idea behind separating styling and highlighting is that different clients may require different techniques for highlighting, yet we want the highlight to look roughly the same on all clients.

Injecting and highlighting are very client specific. Identifying is mostly language specific, but the resulting descriptions need to be in a format that can be used by the locator (and possibly by the highlighter).

In order to make highlighting extensible, extensions need to be able to add context identifiers to languages. There can be multiple identifiers per language, possibly creating descriptions of the same elements in different formats or providing additional descriptions. For instance, one extension may concentrate solely on identifying the nodes selected by a certain jQuery call, another may concentrate on allowing `<canvas>` elements to be highlighted.

Todo: flesh out API proposals


## General purpose hooks

Our live development support does not solely rely on the Chrome remote debugger. Via the RemoteAgent we also add capabilities like CSS rule highlighting by injecting code into the running page whenever it reloads. Similarly, extensions should be able to register such services. While it is tempting to merely allow adding more agents to the current LiveDevelopment module, this is not generic enough. Rather, the connection process should be split into phases.

1. Establishing a connection to the client
2. Installing services (i.e. loading the agents)
3. Extending services (i.e. running client plugins)
4. Initializing services (this would allow extensions to configure the agents)

Such plugins would be specific to a client and can therefore use client-specific capabilities like the agents we define for Chrome.





## Notes

### Features we think should be able to be built as plug-ins
_From the [Extensibility Proposal](https://zerowing.corp.adobe.com/display/brackets/Extensibility+Proposal)_ (internal)
- Access to code model
- Quick open / go to symbol
- Index files/code (background process)
- Better code hinting for "supported" languages (e.g. for a particular framework)
- JSLint tools
- Inline editor providers (e.g. CSS gradient editor)
- Language-specific search and replace (e.g. HTML tag search and replace)
- Code cleanup / refactoring tools

### LiveDevelopment considerations
* What hooks would need to be added where to allow extension-based language support?  
  _LiveDevelopment.registerDocumentClass(...)_
* What behavior would need to be configurable by extensions?  
  _turning off auto-reload for LESS files, configuring CodeMirror_
* What deployment scenarios do we want to support?  
  _Dynamic client-side, dynamic server-side, static server-side?_
* Do we need to integrate with 3rd-party build tools?  
  _I.e. run ant/cake/... when changing a LESS file to initiate compliation to CSS_
* Do we need a specification to delegate compilation to an existing server?  
  _The server might use a compiler written in a language other than JS, or might use a JS compiler with specific options that can't be easily extracted_
* What can we support without impacting performance too much?  
  _Running a build tool for every character entered will not work_
* How should the user configure Brackets to support server-side compilation?  
  _What file needs to be compiled how when changed - setting for the entire project or a subdirectory? Exceptions for single files?_
* How do we deal with distributed files?  
  _LESS supports including other LESS files, so changes to these should trigger compilation of the master file_