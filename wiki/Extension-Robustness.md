Kevin Dangoor, dangoor@adobe.com

Brackets extensions have been a huge success. They're easy to write and have lots of power. Because extensions are so useful, which is exactly what we want, lots of people use them. Unfortunately, some extensions cause problems for Brackets users and these problems are indistinguishable from core Brackets problems. Problems include:

* Brackets failing to start
* Slow performance
* Features stop working (e.g. Live Preview)

Further compounding the issues is that Brackets as a project is not sitting still. We try to maintain compatibility as best we can, but our project is constantly moving forward.

There are *many* things we can potentially do to mitigate these problems. We need to strike a balance between:

* protecting Brackets core functionality
* extension power
* user convenience

For example, we can absolutely protect Brackets core functionality and even go a long way toward proecting performance by sandboxing extensions. However, this will limit what extensions can do, especially when it comes to interacting with the user interface.

Without sandboxing and without too much difficulty, we can ensure that an extension cannot cause Brackets to fail to start up. Once Brackets is started, however, an extension can easily cause the editor to not respond.

We knew that what we're seeing today would come and others have had to deal with it. Every extensible application has to decide which tradeoffs to make and we're at that point where we need to decide for ourselves.

## Brackets Core Solutions

There are steps we can take within the Brackets core to ensure that Brackets runs well.

* Replace our event system
* Sandboxing
* Try to log extension errors
* Harden public APIs

### Replace our event system

This is something we absolutely should do. The biggest problem with our current event system is that an event handler that fails will prevent all subsequent event handlers from running. That's probably okay when all of the code is part of a single codebase. But, when you have code that is maintained by others, it's important to minimize the ways in which one extension can unintentionally interfere with another.

I have seen this problem actually cause Brackets to not start up (because the AppInit event failed to fire).

It's also worth noting that Peter Flynn commented to me about the relatively slow performance of firing events using jQuery, not to mention that all of our application events have a useless first parameter which is a more DOM-oriented event object.

_Note: Ian pointed out that jQuery Promises have the same problem_ -- if one done()/fail() handler crashes, none of the subsequent ones attached to that same Promise get to run. Interestingly, Promises are also really inefficient in jQuery. Changing to a different library like Q would change the semantics quite a bit; if we didn't want to go that far, I wonder if we could just write out own simple jQuery-style Promise impl. - Peter

### Sandboxing

The idea behind sandboxing of extensions is that extensions run in some separate context from the Brackets core and only have access to the APIs that we explicitly provide. Because they run in a separate context, the amount of damage that an extension can cause is minimized. The biggest problem with sandboxing is that sandboxed extensions would not be able to directly manipulate the DOM, something that some extensions do to provide custom UI. Additionally, there is a performance cost to crossing the line between the sandbox and the main process.

As our thinking progressed on [extension APIs](https://github.com/adobe/brackets/wiki/Extension-API-Research), we realized that it would be possible to maintain quite a bit of backwards compatibility even when moving to a sandbox. It may also be possible to allow an extension to provide a bit of code that runs outside of the sandbox (though such extensions would not work if we ever have a context in which the sandbox is also used for security reasons).

Sandboxing may be worthwhile, but would require considerable effort and some further design and feasibility thinking would need to be done.

### Logging extension errors

If we can trace errors back to the extensions that caused them, this will help everybody to get the problems fixed quickly. For example, our new event system would trap exceptions in handlers. If we can log that the handler was in extension X, that would be useful.

### Harden public APIs

Most of our public APIs are built for specific features that we needed to implement. It's possible that they don't all validate their arguments fully. This is hypothetical, but if we find that there are extensions that are failing because of bad arguments to our APIs, we can make those APIs more resilient.

Additionally, we can try to ensure that our public APIs validate their arguments as we create those APIs.

## Extension Management Solutions

Through the Extension Manager and Extension Registry, we can make it easier for Brackets users to keep their Brackets running well, identify the extensions that are causing issues and report problems to the extension author and other users.

* [Reload without user extensions](https://github.com/adobe/brackets/pull/6334)
* "Safe Mode" (disable all extensions and restart)
* Selectively disable extensions (possibly a feature like "bisect" to identify the problem extension)
* Rating, flagging and commenting
* Link to extension issue tracker
* Flag extensions that use private APIs – these are more likely to break

## Testing

Finally, proactive testing of extensions, especially popular ones, will help ensure that functionality that Brackets users rely on will continue to work well.

* Review/test popular extensions
* Post scenario testing guide, provide other testing help

# Plan of Action

Once we've had some discussion, this section will have a plan of action to resolve the issues we're seeing.