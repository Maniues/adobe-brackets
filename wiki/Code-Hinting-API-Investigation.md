<br>
### This page has been replaced by _[Code Hinting Functional Specifications](Code Hinting Functional Specifications)_
<br>

In Sprint 18, we are planning to improve our existing code hint API. The goal of this document is to summarize the requirements this API would need to meet so that we could provide "first class" code hinting providers for various languages.

Our API for code hints can be separated into two parts:

1. The API that interacts with providers and the editor in order to decide when to show and hide code hints, and to mediate which provider offers hints for a given hinting session.
2. The API that lets providers interact with the actual code hint UI (e.g. allowing a provider to put images in the code hint list, or otherwise augment the list UI).

We (currently) believe these issues are largely separate. This document focuses primarily on the first issue.

#General Requirements
* **Efficient** - When hints are _not_ being displayed and a character is typed, it must be fast to determine if hints should be displayed. When hints _are_ being displayed and a character is typed, it must be fast to determine if the list should be updated or closed.
* **Pluggable** - Extensions should be able to register code hint providers.
* **Multiple providers per language** - A given code hint provider may only offer suggestions for a portion of a language (e.g. provide font name hints in CSS for a particular font service). We should support having multiple concurrent providers for a given language, with a reasonable mechanism for resolving precedence.
 * **Open question:** Should multiple providers be able to offer hints _at the same exact time_? In other words, do we want to try to merge result lists? Joel votes "no".
* **Works in HTML Mixed Mode** - Should be able to use multiple providers from multiple languages in html, exactly as one would expect.
* **Synchronous** - Things will probably get messy if we allow providers to return results asynchronously.
 * **Open Question** - Is this actually true? It has the implication that it would be tough for a provider to use a Web worker to keep a code model up to date. (This is because the Web worker would have to keep the code model in its memory space, which would mean that querying the code model would have to happen asynchronously.) 
* **Works in inline editors** - We don't currently support showing the code hint *UI* in inline editors, but we would like to in the future. We should try to make sure we don't design this API in a way that precludes it from being used in inline editors.

#Specific Language Requirements

##HTML

* Multiple providers for different "parts" of HTML (e.g. tag names, attribute names, attribute values)
 * Implication: A single keypress could end the use of one provider and start the use of another provider. For example, if the user has typed "<a", they will have "a" and "address" in their list as tag names. When they press "space", the tag name will be decided, they will leave tag hinting, and start attribute hinting.
 * Implication: When in a "not showing hints" state, providers need to be given sufficient context to determine if they can provide suggestions.

##CSS

* Values can contain "space" characters
 * Implication: We can't use "space" to indicate that completion is finished. More generally, once a provider is selected to provide hints, the provider should be the one to specify when it is done providing hints.
* Specific values may need to be wrapped in quotes. For example, the user may currently have the rule ```font-family: Helvetica``` with the IP at the end of "Helvetica". If they bring up hints at this time, they may get the suggestion "Helvetica Neue". If the user chooses this, the rule would change to ```font-family: "Helvetica Neue"``` (with quotes).
 * Implication: Code hint providers may need to modify the contents of the editor before the initial insertion point. In general, code hint providers should be able to modify the editor contents arbitrarily when an insertion is chosen. This means that, after an insertion, we must wait for the provider to finish updating the buffer before we re-query all providers to determine if we should start a new hinting session. If we don't wait, providers could see the wrong context.

##JavaScript

* It is _possible_ to come up with suggestions for JavaScript basically any time the insertion pointer is surrounded by whitespace. It should **not** be the case that if it is _possible_ to provide hints then hints will _certainly_ be shown. As one example, if the user's IP is surrounded by whitespace and the user types a "space", hints should **not** appear.
 * Implication: For any keypress, it should be up to the provider to determine whether it is appropriate (from a UI perspective) for the editor to show hints. The provider needs information on what key was pressed, and with what modifiers.
 * Open Question: Should we have a global key command that **always** displays hints (if any are possible)? Joel votes "yes". This means that we'll need to have a parameter to the "do you want to provide hints?" API call that indicates whether the user has explicitly requested hints. The alternative would be to just let individual providers handle this by looking at key event modifiers.
* Hinting for identifiers should start immediately after the user starts typing the identifier, but should probably not immediately start if they start typing a literal.
 * Implication: The characters which start completion can change depending on the context of the cursor. Our API cannot _require_ providers to specify a static list of characters on which hinting should/could be initiated.

##Other thoughts
* Some autocompletion modes in other editors do a crazy thing: if the user types enough of a token for there to only be one suggestion, then that suggestion is automatically completed (without the user pushing 'tab' or 'enter'). This is only useful in languages that have a particularly rigid syntax (i.e. where the completion engine can be certain that the user wouldn't type anything else). One example of this is text macro expanders for prose. Do we want to support the creation of such providers?
 * Implication: If we want to support this, then we need the provider to be able to be able to cause a completion to occur on any keypress. If we choose not to support it, the provider could probably do this in a hacky way by saying "I'm done offering completions" on a keypress, and then changing the editor. But that may run afoul with the implication in CSS above that the provider needs to have an explicit chance to modify the buffer before the code hint manager re-queries providers for a new round of completion. 
 * [nj] I mentioned this in the other doc, but I think we should consider this kind of in-place autocompletion to be completely separate from code hinting.

#Proposed Architecture

TBD

#