<br>
_These APIs were implemented in Sprint 18._ For background on the direction our code hinting APIs are going, see [Code Hinting Functional Specifications](https://github.com/adobe/brackets/wiki/Code-Hinting-Functional-Specifications). 

## `CodeHintManager.registerHintProvider(provider, modes, priority)`

The method by which a `CodeHintProvider` registers its willingness to providing hints for editors in a given mode.

### `@param {CodeHintProvider} provider`
The hint provider to be registered. 

### `@param {Array[string]} modes`
The set of mode names for which the provider is capable of providing hints. If the special mode name "all" is included then the provider may be called upon to provide hints for any mode.

### `@param {Integer} priority`
A non-negative number used to break ties among hint providers for a particular mode. Providers that register with a higher priority will have the opportunity to provide hints at a given mode before those with a lower priority. Brackets default providers have priority zero.

## `CodeHintProvider.hasHints(editor, implicitChar)`

The method by which a provider indicates intent to provide hints for a given editor. The manager calls this method both when hints are explicitly requested (via, e.g., Ctrl-Space) and when they may be implicitly requested as a result of character insertion in the editor. If the provider responds negatively then the manager may query other providers for hints. Otherwise, a new hinting session begins with this provider, during which the manager may repeatedly query the provider for hints via the getHints method. Note that no other providers will be queried until the hinting session ends. 

The `implicitChar` parameter is used to determine whether the hinting request is explicit or implicit. If the string is null then hints were explicitly requested and the provider should reply based on whether it is possible to return hints for the given editor context. Otherwise, the string contains just the last character inserted into the editor's document and the request for hints is implicit. In this case, the provider should determine whether it is both possible and appropriate to show hints. Because implicit hints can be triggered by every character insertion, hasHints may be called frequently; consequently, the provider should endeavor to return a value as quickly as possible. 

[Glenn] I assume this means that IME compositions will *never* open code hints. Is that right? If so, it should be called out explicitly here.

[Ian] Assuming by "IME compositions" you mean character insertions effected by an input-method editor in the sense of http://en.wikipedia.org/wiki/Input_method, then I'm not sure why this would be any different from characters inserted by a more traditional method. The events that result from the insertion may be different, but I would think the manager could detect those events and pass the associated character to hasHints to determine if implicit hints are appropriate. 

[Glenn] IME insertion usually involves multiple characters, which is why I brought it up. In that case there wouldn't be a single character to pass to the hint provider (there is a single `change` event, but it typically has more than 1 character). I chatted with Raymond about this, and he agreed that code hints do not need to show up for IME insertions.

Because calls to `hasHints` imply that a hinting session is about to begin, a provider may wish to clean up cached data from previous sessions in this method. Similarly, if the provider returns true, it may wish to prepare to cache data suitable for the current session. In particular, it should keep a reference to the editor object so that it can access the editor in future calls to `getHints` and `insertHints`.

### `@param {Editor} editor`
A non-null editor object for the active window. 

### `@param {String} implicitChar`
If the hinting request was explicit then null. Otherwise, if the hinting request was implicit, the character code of the last keypress event, or the empty string if there was no keypress event (e.g., enter, tab, backspace, etc.)

### `@return {Boolean}`
Determines whether the current provider is able to provide hints for the given editor context and, in case `implicitChar` is non-null, whether it is appropriate to do so. 

## `CodeHintProvider.getHints(implicitChar)`

The method by which a provider provides hints for the editor context associated with the current session. The getHints method is called only if the provider asserted its willingness to provide hints in an earlier call to hasHints. The provider may return null, which indicates that the manager should end the current hinting session and close the hint list window. Otherwise, the provider should return a response object that contains three properties: 
 1. `hints`, a sorted array hints that the provider could later insert into the editor; 
 2. `match`, a string that the manager may use to emphasize substrings of hints in the hint list; and 
 3. `selectInitial`, a boolean that indicates whether or not the the first hint in the list should be selected by default. 
If the array of hints is empty, then the manager will render an empty list, but the hinting session will remain open and the value of the selectInitial property is irrelevant. 

Alternatively, the provider may return a `jQuery.Deferred` object that resolves with an object with the structure described above. In this case, the manager will initially render the hint list window with a throbber and will render the actual list once the deferred object resolves to a response object. If a hint list has already been rendered (from an earlier call to `getHints`), then the old list will continue to be displayed until the new deferred has resolved. 

Both the manager and the provider can reject the deferred object. The manager will reject the deferred if the editor changes state (e.g., the user types a character) or if the hinting session ends (e.g., the user explicitly closes the hints by pressing escape). The provider can use this event to, e.g., abort an expensive computation. Consequently, the provider may assume that `getHints` will not be called again until the deferred object from the current call has resolved or been rejected. If the provider rejects the deferred, the manager will end the hinting session.

The `getHints` method may be called by the manager repeatedly during a hinting session. Providers may wish to cache information for efficiency that may be useful throughout these sessions. The same editor context will be used throughout a session, and will only change during the session as a result of single-character insertions, deletions and cursor navigations. The provider may assume that, throughout the lifetime of the session, the `getHints` method will be called exactly once for each such editor change. Consequently, the provider may also assume that the document will not be changed outside of the editor during a session. 

### `@param {String} implicitChar`

Either null, if the request to update the hint list was a result of navigation, or a single character that represents the last insertion.

### `@return {(Object + jQuery.Deferred)<hints: Array<(String + jQuery.Obj)>, match: String, selectInitial: Boolean>}`

Null if the provider wishes to end the hinting session. Otherwise, a response object, possibly deferred, that provides 
 1. a sorted array `hints` that consists either of strings or jQuery objects; 
 2. a string `match`, possibly null, that is used by the manager to emphasize matching substrings when rendering the hint list; and 
 3. a boolean that indicates whether the first result, if one exists, should be selected by default in the hint list window. 
If `match` is non-null, then the hints should be strings. If the `match` is null, the manager will not attempt to emphasize any parts of the hints when rendering the hint list; instead the provider may return strings or jQuery objects for which emphasis is self-contained. For example, the strings may contain substrings that wrapped in bold tags. In this way, the provider can choose to let the manager handle emphasis for the simple and common case of prefix matching, or can provide its own emphasis if it wishes to use a more sophisticated matching algorithm. **Note:** the CodeHintManager currently supports only string hints.

[Glenn] Is a single prefix string sufficient here? Many (most?) code hints will do partial hinting, so for example, I can type "chm" and have CodeHintManager hinted (with **C**, **H**, and **M** bolded). Will that be supported? 

[Ian] The API description above has been elaborated to cover arbitrary matching and highlighting algorithms. 

[Jason] I feel like I'm always going back to Eclipse APIs. It might be worth looking at ICompletionProposal here for inspiration. http://help.eclipse.org/juno/index.jsp?topic=%2Forg.eclipse.platform.doc.isv%2Fguide%2Feditors_contentassist.htm. Instead of hint being a string, the proposal can provide it's own styled string, additional tooltip info (say JSDoc, MDN, caniuse.com, etc.), trigger characters (e.g. using an open paren to close method call proposals). I found all of these helpful when building code hints in Flash Builder.

## `CodeHintProvider.insertHint(hint)`

The method by which a provider inserts a hint into the editor context associated with the current session. The provider may assume that the given hint was returned by the provider in some previous call in the current session to `getHints`, but not necessarily the most recent call. After the insertion has been performed, the current hinting session is closed. The provider should return a boolean value to indicate whether or not the end of the session should be immediately followed by a new explicit hinting request, which may result in a new hinting session being opened with some provider, but not necessarily the current one. 

### `@param {String + jQuery.Object} hint`
The hint to be inserted into the editor context for the current session. **Note:** the CodeHintManager currently supports only string hints.

### `@return {Boolean}`
Indicates whether the manager should follow hint insertion with an explicit hint request. 

[Glenn] Is this return value needed? It seems like the code hint manager should *always* call `hasHints()` if there are no hints showing, which should allow a single key event to close the existing hints and open new hints.

[Ian] An additional explicit hint request is only desirable in some cases. For example, in HTML it is desirable after attributes with values are auto-completed, but not for attributes without values. It is also not generally desirable, as far as I can tell, after auto-completing most JavaScript identifiers or properties. (In *some* cases it may be useful, like function or method calls for which actual parameters desired.) 

## `CodeHintProvider.onHighlight(hint)`

This is an optional notification hook, if implemented, `CodeHintManager` uses this hook to notify the hint provider about highlight change in the `CodeHintList`. This is useful when a hint provider wants to implement some additional interactive workflows when a user crawls over hint items in hint popup like showing some tips/description or highlighting something in another context (design view / Live preview).  

### `@param {String + jQuery.Object} hint`
The hint highlighted in the `CodeHintList` for the current session. **Note:** It can be a string or a jQuery object based on the list that was provided on `getHints` call.

## `CodeHintManager` Overview: 

The `CodeHintManager` mediates the interaction between the editor and a collection of hint providers. If hints are requested explicitly by the user, then the providers registered for the current mode are queried for their ability to provide hints in order of descending priority by way their `hasHints` methods. Character insertions may also constitute an implicit request for hints; consequently, providers for the current mode are also queried on character insertion for both their ability to provide hints and also for the suitability of providing implicit hints in the given editor context. 

Once a provider responds affirmatively to a request for hints, the manager begins a hinting session with that provider, begins to query that provider for hints by way of its getHints method, and opens the hint list window. The hint list is kept open for the duration of the current session. The manager maintains the session until either:
 1. the provider gives a null response to a request for hints; 
 2. a deferred response to getHints fails to resolve; 
 3. the user explicitly dismisses the hint list window; 
 4. the editor is closed or becomes inactive; or 
 5. the editor undergoes a "complex" change, e.g., a multi-character insertion, deletion or navigation. 
Single-character insertions, deletions or navigations may not invalidate the current session; in which case, each such change precipitates a successive call to getHints. 

If the user selects a hint from the rendered hint list then the provider is responsible for inserting the hint into the editor context for the current session by way of its `insertHint` method. The provider may use the return value of `insertHint` to request that an additional explicit hint request be triggered, potentially beginning a new session. 