### What is LSP?
The Language Server protocol is used between a tool (the client) and a language smartness provider (the server) to integrate features like autocomplete, go to definition, find all references and alike into the tool.

### Why LSP?
For implementing tooling support for any language in Brackets, we need developers who are both

* Brackets experts 
* Language experts

**LSP decouples these two:**

* Language Server can be developed independently (by the language experts)
* Can be plugged into any editor having LSP framework (developed by Brackets experts) as an extension.

### A closer look at the communication between an LSP client and a Language Server

![Communication Sequence](https://microsoft.github.io/language-server-protocol/img/language-server-sequence.png)

Source: https://microsoft.github.io/language-server-protocol/overview

### LSP framework implementation in Brackets

**Current LSP infrastructure supports**
* Code completion
* Jump to definition
* Function signature
* Diagnostics/Linting
* Find Symbols
* Find References

**Future Work**
* Support for other capabilities like snippet, hover etc by LSP

See specification for reference: https://microsoft.github.io/language-server-protocol/specification

![LSP in Brackets - Overview](https://github.com/shubhsnov/brackets/blob/LSP-Images/overview.png)

**Brackets itself has two primary components**:
* Brackets-Shell (Customized CEF Client which allows browser capabilities)
* Node Server Process (Spawned by Brackets-Shell on startup which provides a separate node runtime)

The LSP framework in Brackets allows us to create a client for the language server in the node runtime and then 
seamlessly communicate with it using an interface instance from within Brackets runtime (CEF).

![LSP in Brackets - Overview](https://github.com/shubhsnov/brackets/blob/LSP-Images/node-server.png)

The actual LanguageClient is hosted within the node context. This client is responsible for spawning the language server, establishing communication with the server and then acting as a bridge between Brackets and the server for the two-way requests and notifications. 

![LSP in Brackets - Overview](https://github.com/shubhsnov/brackets/blob/LSP-Images/brackets-node.png)

Once the server is spawned and LanguageClient created, we get an interface object that allows us to send and receive requests and notifications to and from the server using the LanguageClient.

This interface is an instance of [LanguageClientWrapper](https://github.com/adobe/brackets/blob/master/src/languageTools/LanguageClientWrapper.js).

The entire communications logic is abstracted and is Promise based.

Refer to the control flow diagram below to understand how LSP works in Brackets.

![LSP in Brackets - Overview](https://github.com/shubhsnov/brackets/blob/LSP-Images/flow.png)

## Creating an LSP Extension

**Pre-requisites**: 
[How to write Brackets Extensions](https://github.com/adobe/brackets/wiki/How-to-Write-Extensions) 

The Language Client in Brackets abstracts the communication between the Extension context in Brackets and Node, streamlining the connection between the Extension and the Language Server.

The Extension usually would have three layers:

* Brackets Context (lies inside the CEF/Shell)
* Language Client Context (lies as a Node Domain)
* Language Server (Separate process)
The communication between Language Client and the Server happens through a [node module](https://www.npmjs.com/package/vscode-languageserver-protocol). 

The synchronization between the Language Client and Brackets is handled by an object which interfaces with the Language Client and acts as a wrapper called LanguageClientWrapper.

### Extension Structure:
Bare minimally the developer will need two files to create a LanguageClient extension:

  1. **_main.js_** (extension entry point)
      * Initiates client for a Language Server (provide path of client.js)
      * Manage the lifecycle of Language Server, using the client.
      * Interact with Brackets’ core and Language Server to provide tooling.

        See reference [here](https://github.com/adobe/brackets/blob/master/test/spec/LanguageTools-test-files/clients/FeatureClient/main.js)

  2. **client.js**
      * Contains Information specific to launching a Language Server
      * Instantiates a LanguageClient that interacts with a Language Server
      * Handles any other to-and-fro of information between Brackets and Node which might be required before spawning the server, like ‘runtime’ path, specific options etc.
      * Infrastructure provided as part of LSP.

        See reference [here](https://github.com/adobe/brackets/blob/master/test/spec/LanguageTools-test-files/clients/FeatureClient/client.js)

### Interfacing with Language Server

* **client.js**: This file describes how to start a language server and works as a bridge between the Language Server and Brackets Context. Any customizations that are required from the Brackets Context are handled here:
1. Load the LanguageClient Module.
```javascript 
var LanguageClient = require(global.LanguageClientInfo.languageClientPath).LanguageClient;
```
2. Define the **serverOptions**. serverOptions tells the client how to start the server. These options can be a JSON object or a function. Refer [here]() and [here]() to know various ways of defining the serverOptions.

    serverOptions can be defined in three ways:
      1) Runtime (if the server is not a node module) or Module (node runtime) JSON
      2) Function
      3) Command JSON
 
      Runtime, Module & Command are abstractions for node's spawn and fork functions.
      Refer [spawn](https://nodejs.org/api/child_process.html#child_process_child_process_fork_modulepath_args_options) & [fork](https://nodejs.org/api/child_process.html#child_process_child_process_spawn_command_args_options) to understand the option parameters.

```javascript 
//Sample Runtime based option (goes through spawn in case runtime is specified and fork otherwise)
//command line format: [runtime] [execArgs] [module] [args (with communication args)] (with options[env, cwd])
 serverOptions = {
     runtime: process.execPath, //Path to node but could be anything, like php or perl. No need to specify this if the module is a node module
     module: "main.js",
     args: [
                    "--server-args" //module args
                ], //Arguments to process
     options: {
         cwd: serverPath, //The current directory where main.js is located
         env: newEnv, //The process will be started CUSTOMENVVARIABLE in its environment
         execArgv: [
                        "--no-warnings",
                        "--no-deprecation" //runtime executable args
                    ]
     },
     communication: "ipc"
 };
 
 
//Sample function based options (executed directly)
serverOptions = function () {
    return new Promise(function (resolve, reject) {
        var serverProcess = cp.spawn(process.execPath, [
                            "main.js",
                            "--stdio" //Have to add communication args manually
                        ], {
            cwd: serverPath
        });
 
        if (serverProcess && serverProcess.pid) {
            resolve({
                process: serverProcess
            });
        } else {
            reject("Couldn't create server process");
        }
    });
};
 
//Sample command based options (goes through spawn)
//command line format: [command] [args] (with options[env, cwd])
serverOptions = {
    command: process.execPath, //Path to executable, mostly runtime
    args: [
                    "--no-warnings",
                    "--no-deprecation",
                    "main.js",
                    "--stdio", //Have to add communication args manually
                    "--server-args"
                ], //Arguments to process, ORDER WILL MATTER
    options: {
        cwd: serverPath,
        env: newEnv //The process will be started CUSTOMENVVARIABLE in its environment
    }
};

```
3. Set the LanguageClient options.
```javascript 
var options = {
    serverOptions : serverOptions
};
```
4. Instantiate the LanguageClient in the init function.
```javascript 
function init(domainManager) {
    client = new LanguageClient(clientName, domainManager, options); //Initiate a new LanguageClient with options
}
//or
function init(domainManager) {
    client = new LanguageClient(clientName, domainManager); //Initiate a new LanguageClient
}
client.setOptions(options); //We generally load the options later when the options
// require some information from the Brackets context that is not immediately available in init method.

//See https://github.com/adobe/brackets/blob/master/test/spec/LanguageTools-test-files/clients/CommunicationTestClient/client.js
```

### [Sample Language Client File](https://github.com/adobe/brackets/blob/master/test/spec/LanguageTools-test-files/clients/FeatureClient/client.js)

* **main.js**:
1. Load the LanguageTools module in main.js
```javascript 
var LanguageTools = brackets.getModule("languageTools/LanguageTools");
```
2. Load the LanguageClient described in client.js through the initiateToolingService API
     * _clientName_ (name of client)
     * _clientFilePath_ (absolute path of the LanguageClient)
     * _languageIdsArray_ (Array of languageIds describing the languages being supported by the LanguageClient)
```javascript 
var client = null;
AppInit.appReady(function () {
    LanguageTools.initiateToolingService(clientName, clientFilePath, languageIdsArray).done(function (_client) {
        client = _client; //LanguageClientWrapper object
    });
});
```

3. Now that we have the client, it can be used to interact with the server.
4. Server Lifecycle APIs
```javascript 
//Start with options
client.start({
    rootPath: projectPath,
    capabilities ? : capabilities //parameters marked with '?' are optional
});
 
 
//Stop a client
client.stop();
 
 
//Restart a client
client.restart({
    rootPath: projectPath,
    capabilities ? : capabilities
});
```
5. Message Format for Requests and Notifications: 
   You can communicate with the server by using the client API and send a proper JSON RPC message.
   This message can have two formats:
     * brackets (default)
     * lsp (as defined in the [specification](https://microsoft.github.io/language-server-protocol/specification))
```javascript 
//So you have two ways of sending the same message to the server:

//In Brackets Format
client.requestHints({ //Converted internally to the LSP format
    filePath: docPath,
    cursorPos: pos
});

//or 

//In LSP Format as defined here: https://microsoft.github.io/language-server-protocol/specification#textDocument_completion
//This is for advanced developers who understand the protocol and would like to customize Brackets tooling as per their need.
//Basically you can use the JSON as described by the Specification,
//with an additional key 'format' telling brackets not convert the message.
client.requestHints({
    format: 'lsp'
    textDocument: {
       uri: fileUri
    },
    position: position //LSP format
});
```
6.  Events APIs
```javascript 
//Notify server of file open event
client.notifyTextDocumentOpened({
    languageId: languageId,
    filePath: (doc.file._path || doc.file.fullPath),
    fileContent: doc.getText()
});
 
//Notify server of file closed event
client.notifyTextDocumentClosed({
    filePath: (previous.document.file._path || previous.document.file.fullPath)
});
 
//Notify server of project change
client.notifyProjectRootsChanged({
    foldersAdded: [this.currentProject],
    foldersRemoved: [this.previousProject]
});
 
 
//Notify server of file save
client.notifyTextDocumentSave({
    filePath: (doc.file._path || doc.file.fullPath)
});
 
 
//Notify server of file change
client.notifyTextDocumentChanged({
    filePath: (doc.file._path || doc.file.fullPath),
    fileContent: doc.getText()
});
```
7. Server request APIs
```javascript
//Client request for Language Server
//https://microsoft.github.io/language-server-protocol/specification
//All request return $.Deferred() promise.
 
 
//https://microsoft.github.io/language-server-protocol/specification#textDocument_completion
client.requestHints({
    filePath: docPath1,
    cursorPos: pos
});
 
//https://microsoft.github.io/language-server-protocol/specification#completionItem_resolve
client.getAdditionalInfoForHint({
    hintItem: hintItem
});
 
//https://microsoft.github.io/language-server-protocol/specification#textDocument_signatureHelp
client.requestParameterHints({
    filePath: docPath2,
    cursorPos: pos
});
 
//https://microsoft.github.io/language-server-protocol/specification#textDocument_definition
client.gotoDefinition({
    filePath: docPath2,
    cursorPos: pos
});
 
//https://microsoft.github.io/language-server-protocol/specification#textDocument_implementation
client.gotoImplementation({
    filePath: docPath2,
    cursorPos: pos
});
 
//https://microsoft.github.io/language-server-protocol/specification#textDocument_declaration
client.gotoDeclaration({
    filePath: docPath2,
    cursorPos: pos
});
 
//https://microsoft.github.io/language-server-protocol/specification#textDocument_references
client.findReferences({
    filePath: docPath2,
    cursorPos: pos
});
 
//https://microsoft.github.io/language-server-protocol/specification#textDocument_documentSymbol
client.requestSymbolsForDocument({
    filePath: docPath2
});
 
//https://microsoft.github.io/language-server-protocol/specification#workspace_symbol
client.requestSymbolsForWorkspace({
    query: query
});
```
8. Server Event APIs
```javascript
//Server Events for Language Server
 
 
//https://microsoft.github.io/language-server-protocol/specification#window_logMessage
client.addOnLogMessage(function (params) {
    //do something
});
 
//https://microsoft.github.io/language-server-protocol/specification#window_showMessage
client.addOnShowMessage(function (params) {
    //do something
});
 
 
//https://microsoft.github.io/language-server-protocol/specification#telemetry_event
client.addOnTelemetryEvent(function (params) {
    //do something
});
 
//https://microsoft.github.io/language-server-protocol/specification#textDocument_publishDiagnostics
client.addOnCodeInspection(function (params) {
    //do something
});
 
//https://microsoft.github.io/language-server-protocol/specification#client_registerCapability
client.onDynamicCapabilityRegistration(function () {
    //do something
});
 
//https://microsoft.github.io/language-server-protocol/specification#client_unregisterCapability
client.onDynamicCapabilityUnregistration(function () {
    //do something
});
 
//https://microsoft.github.io/language-server-protocol/specification#window_showMessageRequest
client.onShowMessageWithRequest(function () {
    //return something, can be a promise.
});
 
//https://microsoft.github.io/language-server-protocol/specification#workspace_didChangeWorkspaceFolders
client.onProjectFoldersRequest(function () {
    //return something, can be a promise.
});
 
//Can be any notification mentioned above, or not yet implemented in Brackets Core
client.onCustomNotification(type, function (params) {
    //do something
});
 
//Can be any request mentioned above, or not yet implemented in Brackets Core
client.onCustomRequest("custom/serverRequest", function (params) {
    //return something, can be a promise.
});
 
//Can be used to extend the server and handle Brackets specific events
//https://github.com/adobe/brackets/blob/master/test/spec/LanguageTools-test.js#L1482
client.addOnCustomEventHandler("triggerDiagnostics", function () {
    //do something
});
```
Notes: Refer to Sample implementations for [Requests](https://github.com/adobe/brackets/blob/master/src/languageTools/DefaultProviders.js) and [Notifications](https://github.com/adobe/brackets/blob/master/src/languageTools/DefaultEventHandlers.js) for reference.

Additionally, we have created a [reference](https://github.com/adobe/brackets/tree/master/src/extensions/default/PhpTooling) adoption for the [PHP language server](https://github.com/felixfbecker/php-language-server) which can be studied to understand how these APIs work within Brackets.