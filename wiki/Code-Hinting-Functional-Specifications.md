_**This page describes features that are not yet implemented, or only partially implemented. Comments are welcome!**_

_Some functionality was [implemented in Sprint 18](New-Code-Hinting-API-Proposal)._

The purpose of this document is to describe the expected functionality of code hints for a number of important languages. We don't plan to implement all the functionality specified below at the same time (and indeed may never end up implementing some of it). Instead, the goal is to have a specification that can evolve over time.

At any point in time, this document should represent how we currently believe code hinting should work from a functional/user-facing perspective. We can then use this to drive the design of APIs to support code hinting, as well as guide the implementation of actual code hint providers.

#General functionality (all languages)

* The user should be able to enable/disable code hinting entirely on a language-by-language basis.
* The user should be able to _explicitly_ invoke the code hint dialog at any time using a universal (i.e. the same in every mode) key command. By default, that key command should be Ctrl-Space, but should be configurable.
* If the user tries to _explicitly_ invoke the dialog and no hints are available, the user should be notified in an unobtrusive way (e.g., a temporary message in the status bar).
 * [nj] Do other editors do this? I don't feel like I've seen this anywhere (not that it's a bad idea, just curious how others deal with it).
* There should be a universal (i.e. the same in every mode) key command to dismiss the dialog: Escape. This is the global key to dismiss all dialogs and is not configurable.
* Individual code hint providers (e.g. for a specific editing mode) may bring up the dialog _implicitly_ (i.e. without the user explicitly asking for it) when a particular condition is met. For example, in an HTML file, when the user types a "<" character in a location where an HTML tag is valid, code hints for tag names could automatically be displayed.
* Users should have a way to globally disable the _implicit_ display of code hints (i.e. turn off all automatic popups so that code hints only appear when the user presses ctrl-space).
 * **Open question:** Do we want this functionality? Joel votes yes! Raymond is less sure it's necessary.
 * [nj] Yes, we need to let the user have control over this. There are people who find autopopups really annoying, but would appreciate being able to bring them up manually.
* The dialog should appear below the user's cursor, and should move with the user's cursor as he types. The list should be filtered / sorted after each character is typed, and there should be an indication of how much the user's typing matches each result (e.g. the matching portion of the string could be made bold).
 * [Randy] _Preferred_ location is below user's cursor, but if there's not enough room then it should go above.
* There should be a universal (i.e. the same in every mode) key command to accept a code hint. By default, this should be both "tab" and "enter", but should be user configurable.
 * <del>[Randy] Tab key is a valid and _very_ common user input in CSS and JavaScript, so it cannot be used for universal list selection.</del>
 * [Raymond] The real issue is not on using Tab key to select hint. It is the initial selection in the hint list that prevent users from inserting a tab or a new line in the editor. See issue [#2286](https://github.com/adobe/brackets/issues/2286) and the solution in pull request [#2279](https://github.com/adobe/brackets/pull/2279).
 * **Open question:** should this be configurable? Raymond votes no. Joel doesn't see it as important. However, some providers might want to make completion happen automatically without the user needing to press tab or enter. For example, text expanders might automatically expand "ASAP" as soon as the user types enough letters to make it the unique completion.
 * [nj] That kind of auto-expansion seems to me like a different feature from code hints.
* [Raymond] There should be a way to indicate whether the hint dialog should have initial default selection or not. This is necessary to allow users to insert a tab or a new line while the hint dialog is displayed. If the hint dialog does not have the initial selection, Tab or Enter key won't insert any hint into editor. User can use Down arrow key to select the first hint in the hint list before using Tab or Enter key to accept the selected hint.
* [Randy] For values that need to be enclosed in delimiters (e.g. quotes) there needs to be functionality to select a value and move the IP outside of the delimiters (such as with specifying an HTML Attribute value) to facilitate natural typing. There is also a need to _not_ move the IP outside of the delimiters such as with the case with url hinting which displays both folders and files. When a folder is selected, it is desired to keep the IP inside the delimiters to then display the folders and files of the next selected subfolder.
* If the user types a search string while code hints are being displayed that causes no hints to be available, then the dialog should automatically hide.
 * **Open question:** An alternative would be to display "no hints available". This would make it more straightforward for the user to re-bring-up the list in the event of a typo: they would just hit backspace until they had a valid query. Individual providers could still explicitly close the dialog if they chose to (e.g. after the user successfully typed an entire identifier by hand). Raymond believes we don't need this. Joel offered it as an easy way to deal with the situation where a user makes a typo and then corrects it. The main question here is whether we leave it up to the _provider_ to close the hints (because then they could do either), or whether we close the hints in the CodeHintManager whenever the list is empty.
 * [nj] It does depend on what kind of hints you're providing. If you're hinting an enum, and you know for sure that there are no other valid values for the given context, then it might be okay to keep the hint menu open. But in other cases, the user might just be typing a value that you don't know about (e.g. a new class name that they haven't written the rule for yet), and it would be really annoying for the hint menu to stay open. My guess is that we should just follow the lead of other editors here, which I think means always dismiss the hint menu, unless we find some particular use case where we really need to give control to the provider.
* If the user navigates with his insertion pointer (e.g. presses left or right arrow keys, or clicks with mouse), the code hint should behave as if he typed or deleted those characters between the old and new IP location.
 * For efficiency, we could simply close hints if the IP changes lines, or if the IP moves to a point before where this round of code hinting began (e.g. the user hits ctrl-a on mac to go to the beginning of the line). 


#HTML

* Should support the completion of:
 * tag names
 * attribute names (only listing attributes valid for the current tag)
 * enumerated attribute values (only listing values valid for the current attribute)
 * attribute values that correspond to something externally defined (e.g. CSS class and ID names).
* The code hint dialog should be implicitly (automatically) displayed at the following times:
 * the user types a "<" character in a location where a new tag is valid (tag name hints should be displayed)
 * the user types a "space" character after a tag name, attribute value, or attribute name w/o a value (such as "checked" (attribute name hints should be displayed)
 * the user types a "=" character after an attribute name (attribute value hints should be displayed)
* If the user completes an attribute value that requires quotes, and the user has not explicitly typed quotes, the string should be surrounded by double quotes automatically. If the user has already typed a quote (either single or double), the quote type should be preserved.
* When completing strings that optionally have quotes, the query should be robust to the user adding a quote. For example, suppose a file contains ```<div id=``` and the user's IP is after the "=" character. If the user invokes the code hint dialog, it should stay visible if the user next types a quote character.
* Sub-mode autocomplete should work (e.g. CSS hinting should work in ```<style>``` tags)
* [nj] Might want to list the behavior where when you choose an attribute name using code hints, we automatically insert the ="", put the cursor in the quotes, and bring up the attribute value hints. This is a special case that I think had implications on the existing API, and we want to preserve that.

#CSS

* Should support the completion of:
 * enumerated selectors (e.g. html tag names)
 * class and IDs for selectors where the class or ID appears in an HTML file
 * <del>rule</del> property names
 * enumerated <del>rule</del> property values
 * font family names from a generally accepted list of safe fonts. Extensions may extend this list.
 * color values used elsewhere in the file
  * **Open Question:** Do we want to provide autocomplete support for pseudo-selectors/pseudo-classes. If so, which ones, and how would it work?
    * Raymond says: Yes, when user types ":", we can show a list of valid pseudo-selector if the cursor is in the right context.
* Code hints should be implicitly (automatically) displayed:
    * After the user types ": " immediately after a <del>rule</del>property name. (<del>rule</del>property values should be displayed)
        * [Randy] hints need to be displayed after ":". The space is optional whitespace. After user types whitespace, hints should continue to be displayed. Same comment applies for ";" and "," below.
        * [nj] This is an interesting question. If you usually type a space after the ":", you might be annoyed to see hints pop up right away, even though they could logically be applicable. We could finesse this by saying that we don't *auto* pop-up hints until after whitespace, but you could still hit Ctrl-space immediately after the ":" to bring up hints.
        * [nj] Note that Sublime doesn't seem to bring up hints immediately after whitespace; it waits until you type a non-whitespace character. Should we consider following this model in general?
        * [rcs] Coda pops up code hinting after the ":" and keeps it up when a space is typed.
        * [rcs] Espresso simply automatically adds a space after the user types ":" and shows no code hinting afterwards.
    * After the user autocompletes a <del>rule</del>property name (which should automatically insert the ":" as well) (<del>rule</del>property values should be displayed)
        * [nj] By default, I think it should insert ": " (i.e., with a space after the colon).
    * After the user types ";[return]" after completing a <del>rule</del>property value (<del>rule</del>property name completions should be displayed)
        * [nj] I definitely feel that in this case, we should wait until you hit return, and not pop up the completion list right after the semicolon.
    * After the user types ", " in a comma-separated list of values (like font families) (<del>rule</del>property values should be displayed
        * [nj] I'm on the fence about this one, but we should probably be consistent with what we do after ":".
    * After the user types a space in a space-separated list of values (e.g. border shorthand) (<del>rule</del>property values should be displayed)
    * After the user types a ", " in a comma-separated list of selectors (selectors should be displayed)
        * **Open Question:** Should we do this? If so, should we also implicitly display selector hints after the user types "}[return]"
        * Raymond says: Yes, if we provide selector hints, then this should be part of it.
* When the user completes strings that require quotes, they should automatically be added exactly as in HTML.
    * If the user's file already contains ```Helvetica``` (without quotes), and they bring up the completion on Helvetica and choose "Helvetica Neue", quotes should automatically be added.
* We should _not_ append a ";" after completing a rule value. Instead, if the user presses "space" we should bring up the next value completion dialog. 
* **Open Question:** Do we want to provide any support for media queries? If so, how would it work?
    * Raymond says: Yes, but low priority — after selector hinting. (e.g. "@media |" or "@media screen and p|")

#JavaScript

* Should support the completion of:
 * Identifiers in scope
 * Globally defined identifiers (when possible)
 * Property names (when possible) (Note: property names includes functions and instance methods, since all classes are objects)
 * String literals, when they have structure (such as selectors in a jQuery statement) (when possible)
  * Doing this well will be really really hard. :-)
 * **Open Question:** Do we want to support completion of keywords (e.g. "function")? If so, should we do this only _explicitly_ (e.g. when the user types "fun" and then presses ctrl-space)? Or should we also do it _implicitly_ (e.g. the user types "f" in a place where "function" is a valid keyword, so we automatically show  code hints)
 * [Ian] Sublime implicitly suggests keywords. This isn't important for program understanding, but it is for programmer efficiency: it's easier to type 'f[tab]' than 'function'.  
* Code hints should be implicitly (automatically) displayed when:
 * The user types any valid start-of-an-identifier character (e.g. [A-Za-z$_, etc.]) in a place where an identifier is valid. (identifiers should be displayed)
 * [Ian] The user types "(" in a place where an actual parameter to a function call is valid.  
 * The user types "." or "[" in a place where a property name is valid (property names should be displayed)
 * The user types special sequences of characters, such as ```$("``` that indicate string literals of a specific type will be entered (e.g. CSS selectors)
  * Doing this well will be really hard
* Code hints should be closed when the user types a character that is NOT a valid part of an identifier/property name. E.g., if ```foo``` is in scope, and the user types "f" (in a place where an identifier is valid), then hints should be automatically displayed and should contain "foo". But, if the user presses space (thus deciding on "f" as an identifier) then the code hint list should be hidden
 * **Open Question:** In the above situation, if the user then presses backspace, should the list reappear?
  * Raymond says" no. Joel says: most autocomplete systems do _not_ seem to do this. Implementing it well could be really hard.
* Search for identifiers/properties should be "fuzzy", much like quick open. For example, if there is an identifier named ```getValue``` and the user types ```val```, then "getValue" should appear in the list (possibly ranked lower than other literals that start with "val").
* **Open Question:** We should consider how it would work to provide hints for function call parameters for known library functions. For example, what could we offer if the user has ```$.ajax(``` before his IP?

# Feature-specific Code Hinting

In addition to providing the general language support above, extension authors may wish to provide more "targeted" code hinting for a specific part of a language. For example, the user might want to install a code hint plug in that provides color hints in CSS taken from their [Kuler](https://kuler.adobe.com/) profile.  So that we can better define our APIs to enable this, we detail a few functional specs here.

## [Edge Web Fonts](https://github.com/adobe/brackets-edge-web-fonts) Code Hints

* When the user is typing font family values, this provider should override the default CSS hinter to provide Edge-Web-Fonts-specific suggestions.
 * **Open Question:** Should we merge the results from the specific and general providers? Joel votes "no" -- the specific provider should always win if it offers any suggestions. Ultimately, we could imagine allowing users to specify precedence of providers, but we should probably wait until this becomes a problem to design a solution.
* At the bottom of the code hint list, there should be a link to browse for additional fonts. This should open up a dialog to select new fonts.

## Other possible needs

* Specific providers may want to put other "types" of items in the list. For example, they may want to put image previews that autocomplete to image paths.
* [nj] In some cases, we might have multiple providers that really feel somewhat disjoint (as opposed to contributing to the same list). It would be interesting to consider providing a multi-column code hint list in this case. I think this will be rare, though, so we should wait until we have a specific use case for it.