This is an experimental implementation of Live Preview to replace the current ```LiveDevelopment``` architecture with something more flexible that isn't tied solely to Chrome Developer Tools. Since is still experimental, the basic functionality for CSS/HTML editing is working but there are could be some scenarios that might be partially or entirely not covered yet. There are also some known issues and key things to do (see the list below).

### How to make it work

It can be enabled by setting ```livedev.multibrowser``` preference to true. If enabled, it will launch the page in your default browser when clicking on the Live Preview icon as it works today. You should then also be able to copy and paste the URL from that browser into any other browser, live edits will then update all connected browsers at once.

### Landing in Brackets Core
Main assets are placed under ```LiveDevelopment/MultiBrowserImpl``` while the main module ```LiveDevMultiBrowser``` (equivalent to ```LiveDevelopment```) will land under ```LiveDevelopment``` folder. This is probably not the more elegant choice but the decision mainly relies on these two needs:

* Keep current modules from ```LiveDevelopment``` without moving them from the original place since many extensions require them as dependencies
* Keep multi-browser implementation under this folder since it is an (still experimental) implementation of ```LiveDevelopment```.

Changes included in ```LiveDevelopment/main.js``` allows managing both implementations. Both modules are loaded and initialized. A listener for preference changes is in charge of setting the active implementation based on pref value. Current implementation will be active and work as today by default.

### Basic architecture

The primary difference in this architecture is that communication with the browser is done via an injected script rather than CDT's native remote debugging interface, and the browser connects back to Brackets rather than Brackets connecting to the browser. This makes it so:

* launching a preview, injecting scripts into the HTML, and establishing the connection between the previewed page and Brackets are relatively simple and largely decoupled
* live preview can work in any browser, not just Chrome
* multiple browsers can connect to the same live preview session in Brackets
* browsers could theoretically connect from anywhere on the network that can see Brackets (though right now it's only implemented for localhost)
* opening dev tools in the browser doesn't break live preview

Communication between Brackets and the browser is factored into three layers:

1. a low-level "transport" layer, which is responsible for launching live preview in the browser and providing a simple textual message bus between the browser and Brackets.
2. the "protocol" layer, which sits on top of the transport layer and provides the actual semantic behavior (currently just "evaluate in browser")
3. the injected RemoteFunctions script, which is the same as in today's LiveDevelopment and provides Brackets-specific functionality (highlighting, DOM edit application) on top of the core protocol.

The reason for this factoring is so that the transport layer can be swapped out for different use cases, and so that anything higher-level we need that can be easily built in terms of eval doesn't have to be built into the protocol.

The transport layer currently implemented uses a WebSocket server in Node, coupled with an injected script in the browser that connects back to that server. However, this could easily be swapped out for a different transport layer that supports a preview iframe directly inside Brackets, where the communication is via `postMessage()`.

The protocol layer currently exposes a very simple API that currently just contains specific protocol functions: 
 * ```evaluate```  which evals in the browser
 * ```reload``` which reload the page in the browser
 * ```navigate``` which allow the browser navigate to a given URL
 
The over-the-wire protocol is a JSON message that more or less looks like the CDT wire protocol, although it's not an exact match right now.

#### Related Documents

Documents that are loaded by the current HTML live doc are being tracked by ```DocumentObserver``` at the browser side which relies on DOM MutationObserver for monitoring added/removed stylesheets and Javascript files. 

### Explanation of the flow
A block diagram of how the various bits talk to each other ca be found at https://raw.githubusercontent.com/wiki/njx/brackets-livedev2/livedev2-block-diagram.png

Here's a short summary of what happens when the user clicks on the Live Preview button on an HTML page.

1. ```LiveDevelopment``` creates a ```LiveHTMLDocument``` for the page, passing it the protocol handler (```LiveDevProtocol```). ```LiveHTMLDocument``` manages communication between the editor and the browser for HTML pages.
2. ```LiveDevelopment``` tells ```StaticServer``` that this path has a live document. ```StaticServer``` is in charge of actually serving the page and associated assets to the browser. (Note: eventually I think we should get rid of this step - ```StaticServer``` shouldn't know anything about live documents directly; it should just have a way of request instrumented text for HTML URLs.)
3. ```LiveDevelopment``` tells the launcher to open the page via the ```StaticServer``` URL, then it opens the page in the default browser.
4. The browser requests the page from ```StaticServer```. ```StaticServer``` notes that there is a live document for this page, and requests an instrumented version of the page from ```LiveHTMLDocument```. (The current "requestFilterPaths" mechanism for this could be simplified, I think.)
5. ```LiveHTMLDocument``` instruments the page for live editing using the existing ```HTMLInstrumentation``` mechanism, and additionally includes remote scripts provided by the protocol (```LiveDevProtocolRemote```), transport (```NodeSocketTransportRemote```), remote functions (```RemoteFunctions```) and document observation (```DocumentObserver```) which tracks related documents. The transport script includes the URL for the WebSocket server created when the transport is started after being set to the protocol layer.
6. The instrumented page is sent back to ```StaticServer```, which responds to the browser with the instrumented version. Other files requested by the browser are simply returned directly by ```StaticServer```.
7. As the browser loads the page, it encounters the injected scripts. The transport script connects back to the NodeSocketTransport's WebSocket server created in step 3 and sends it a "connect" message to tell it what URL has been loaded in the browser. The ```NodeSocketTransport``` assigns the socket a client id so it can keep track of which socket is associated with which page instance, then raises a ```connect``` event.
8. ```LivDevProtocol``` receives the ```connect``` event and makes a note of the associated client ID updating the current active connections. ```LiveDevelopment``` also receives the ```connect``` event, it checks that the URL that comes on the message matches the current live doc instance and updates session status to STATUS_ACTIVE in case this is the first client connected.
After the live document is loaded in the browser, ```DocumentObserver``` scan related stylesheets and Javascript files and send a message ```DocumentRelated``` which contains their URLs. ```LiveDevelopment``` receives the message, extract stylesshes and creates a LiveCSSDocument instance per each of them. ```LiveHTMLDocument``` listen to the same message and further notifications (added/removed) to keep info about its related docs which is needed by ```LiveDocument``` (eg. when saving changes of a JS file).
9. As the user makes live edits or changes selection, ```LiveHTMLDocument``` calls the protocol handler's "evaluate" function to call functions from the injected ```RemoteFunctions```.
10. The protocol's "evaluate" method packages up the request as a JSON message and sends it via the transport.
11. The remote transport handler unpacks the message and passes it to the remote protocol handler, which finally interprets it and evals its content.
12. If another browser loads the same page (from the ```StaticServer``` URL), steps 4-8 repeat, with ```LiveHTMLDocument``` just adding the new connection's client ID to its list. Future evals are then sent to all the associated client IDs for the page.
