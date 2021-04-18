_This API was implemented in Sprint 18._ For background on the direction our code hinting APIs are going, see [Code Hinting Functional Specifications](https://github.com/adobe/brackets/wiki/Code-Hinting-Functional-Specifications). **_TODO: Is this document fully up to date with what was implemented?_**

In Brackets we have an HTML utility API ``getTagInfo(editor, cursorPosition)`` to provide the context of an HTML tag and is already used by tag hinting and attribute hinting. However, we do not have a similar CSS utility API to provide CSS rule information. So this document specifies the new CSS utility API in order to achieve the following goals.
* To avoid duplicate effort in detecting CSS context by individual extension provider.
* To avoid inconsistent or incomplete implementation of CSS context detection.
* To provide a maintainable, scalable implementation of CSS context API that has a complete coverage of all possible CSS context.

## getInfoAtPos ##
The new API will be defined as <del>``getRuleInfo(editor, cursorPos)``</del> getInfoAtPos. It takes two arguments -- editor object and cursor position, and returns a rule information object.
<br />
 * **Open question** - should we call it *getCssInfo* since some of the context do not apply to a css rule (eg. @charset "u|tf-8" where `|` denotes the cursor location)? 
 * [Jason: What about ``getOffsetInfo()``? I'm reminded here of APIs we design in Flash Builder for querying our code model: http://livedocs.adobe.com/flex/3/extensibility/CodeModel/com/adobe/flexbuilder/codemodel/tree/ASOffsetInformation.html]
 * [Glenn]: I like ``getOffsetInfo()`` or even ``getInfoAtPos()``.
 * [raymond]: I like ``getInfoAtPos()`` since one of the info is the offset for the current token and ``getOffsetInfo()`` may cause confusion.

<br />

The rule information object is defined as follows...
<br />
**Old Version**
```
{ 
  selector:                              // Used when tokenType == SELECTOR
      { index: currentIndexInValues,
        values: [selector1, selector2, ...] },
  prop:                                  // Used when tokenType == PROP_NAME || tokenType == PROP_VALUE
      { name: propName,
        index: currentIndexInValues,
        values: [propValue1, propValue2, ...] },
  position:
      { tokenType: tokenType,
        offset: offsetInCurrentToken } 
}
```

**New Version**
```
{
    context: context,
    offset: offsetInCurrentToken,
    name: name,                    // only set if context is PROP_NAME or PROP_VALUE for now
    index: currentIndexInValues,
    values: [value1, value2, ...], // selectors or property values
    isNewItem: true|false         // true if the index refers to a new insertion, false for an existing one
}
```
[Randy] Each item in the array of selector.values (e.g. selector1, selector2, ...), is also a list of simple selectors (e.g. #nav div li > a:hover). Should these also be parsed into an array? Another idea would be to only initially parse into a string (not the array shown above), then have a separate getSelectorInfo() function to parse the selector list deeper only when needed.

[raymond] Yes, the idea here is to have them as individual selector stored in the array. The main reason to have them as separate pieces is due to the CodeMirror tokenizer. And another reason is to allow users to insert a new selector between two existing selectors. If we just return a combined string, it will make the caller harder to decide where to show selector hints and what type of selector hints to show. With the individual selector approach, the caller just have to check for the first character to figure out whether it is a class selector, id selector, attribute selector or pseudo selector.

<br />
[nj] The format of this object seems a little odd to me--it's not clear to me what the value of the extra levels of structure are, especially since the contents are similar in both cases. Could we just flatten it out like this:

```
{
    tokenType: tokenType,
    offset: offsetInCurrentToken,
    propName: propName, // only set if tokenType is PROP_NAME or PROP_VALUE
    index: currentIndexInValues,
    values: [value1, value2, ...] // selectors or property values
}
```
[raymond] I like your format. I was influenced by the tag info object used in getTagInfo API.

[nj] For naming, I'd suggest `context` instead of `tokenType`.
[raymond] I agree.


[nj] For PROP_VALUE, I'm assuming the value list is a list of comma-separated values (like font names). I know we're not dealing with shorthands yet, but we should think about what the API should look like when we do. Will all the parsing for shorthands be pushed down into the provider, or should the context API provide some information?
<br />
[raymond] CodeMirror breaks the property value list into separate tokens based on "," and white spaces. So my plan is to put them together by combining "," and trailing spaces into the prior non-space value token. So the plan works well in most cases except in the case of some functions like ``rgba(0, 0, 0, 0.4)``. Not quite sure that we can add enough logic to combine them. My current implementation is returning separate pieces as ["rgba(0, ", "0, ", "0, ", "0.4)"]. So the caller still needs to figure out the semantic token by inspecting the returned values array. 

<del>`tokenType`</del> `context` is either an empty string or one of the following values that represents the different context in a CSS document.
 * **PROP_NAME** 
   - will implement in sprint 18 for CSS hinting and font hinting
 * **PROP_VALUE** 
   - will implement in sprint 18 for CSS hinting and font hinting 
 * **SELECTOR** 
   - not plan to implement it in sprint 18, but possibly in sprint 19
 * <del>**MEDIA** </del>
   - <del>may support in the future with some modification to the rule info structure</del>
 * <del>**CHARSET** </del>
   - <del>may support in the future with some modification to the rule info structure</del>
 * **AT_RULE**
   - may support in the future for all at-rules. We may use "name" for the name of at-rule and "values" array for any strings after the at-rule string.

[Glenn] We may want to simplify this to just use AT_RULE instead of MEDIA and CHARSET. This way hints could be provided for *any* at-rule.
<br />
[raymond] I thought about it also, but besides the first token with ``@`` prefix the subsequent tokens have different formats and may require some distinguishable context. 

The value of ``context`` is an empty string for the following context.
 * Current cursor position is in a non-css/non-less document 
 * Current cursor position is within a not-yet-supported or unsupported context - ([examples](#notsupported))
 * Current cursor position is inside some of the invalid context - ([examples](#invalid))
   - We do not have the business logic to distinguish between comma separated values from space separated values. So in the case of a missing comma we may still return PROP_VALUE in "context" and the caller (who has the business logic) needs to check the preceding value to catch the missing comma. See [example 3](#invalid)

###Default values of rule information###
If the cursor is in a non-css /non-less document, or inside the unsupported context of a css/less document, a rule info with the following default values is returned.

[Jason: These empty selector and prop properties seem unnecessary to me. Earlier you mention that selector and prop will only be set if the matching tokenType is specified. Would it make more sense to return only a position property with tokenType=UNKNOWN]
<br />
[raymond] I agreed with you. nj's suggestion to flatten out the info structure will eliminate the redundant
ones.

```
{
    context: "",
    offset: 0,
    name: "",
    index: -1,
    values: [],
    isNewItem: false
}
```
<a name="notsupported"> </a>
###Examples of Not-Yet Supported CSS Context###
>@char|set "UTF-8";
>
>@import ur|l("booya.css") print,screen;
>
>@import "whatup.css" sc|reen;
>
>@import "|wicked.css";

>@namespace "|http://www.w3.org/1999/xhtml";
>
>@namespace s|vg "http://www.w3.org/2000/svg";
>
>@media sc|reen { ... }

<br />

<a name="invalid"> </a>
###Examples of Invalid CSS Context###
> 1. div { clear |: both; }   /* cursor before the colon and after a valid property name */
>
> 2. div { clear b|oth; }      /* missing colon after "clear" property name */
>
> 3. div { font-family: Arial |, Helvetica; }   /* cursor before the comma separator */

<br />

###Examples of Property Name context###
> 1. div {|
>
> 2. div {
><br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|
>
> 3. div {
><br />|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;clip:
>
> 4. div {
><br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|clip:
>
> 5. div {
><br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;clip|:
>
> 6. div {
><br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;c|lip:
>

<br />

All the above examples will return a rule info object with "context" assigned to PROP_NAME. "index" and "values" are not used for PROP_NAME token type so they will have default values.

For example 1, 2 and 3 "name" will be an empty string since the cursor is not in any property name. Example 4 to 6 will have "clip" assigned to "name". "offset" is the cursor offset in the current token "clip" and it is zero for example 4, 4 for example 5 and 1 for example 6.

<br />

###Examples of Property Value context###
> 1. div { font-family:| "Helvetica Neue", Helvetica, Arial, sans-serif; }
>
> 2. div { font-family: "Helvetica Neue"|, Helvetica, Arial, sans-serif; }
>
> 3. div { font-family: "Helvetica Neue", |Helvetica, Arial, sans-serif; }
>
> 4. div { font-family: "Helvetica Neue", Helvetica, Arial,| sans-serif; }
>
> 5. div { font-family: "Helvetica Neue", Helvetica, Arial, sans-serif |; }

<br />
All the examples above will have PROP_VALUE in "context" and "name" will be "font-family". Also, all of them will have the same array in "values" ``[""Helvetica Neue", ", "Helvetica, ", "Arial, ", "sans-serif"]`` regardless of the location of the cursor. Please note that each item in the values array except the last one has a comma and the trailing white spaces. We intentionally append the comma and the trailing spaces so that the caller can reconstruct the actual string or can calculate the start or end position of a specific value.

The values of "index" and "offset" depend on the cursor position in the existing property value. Example 2 will have 0 for "index" and 16 for "offset". Example 3 will have 0 for "offset" and 1 for "index".

When the cursor is in the property value context and there is a white space immediately after the cursor, then we are in a location where the user can add a new property value. This is the case for example 1, 4 and 5. "isNewItem" flag will be true and offset will be 0 for these cases. "index" will be 0 for example 1, 3 for example 4 and 4 for example 5.

<del>[nj] It would be good to explain that the empty string will be at the beginning in example 1 and at the end in example 5. Also, it looks like examples 3-5 have an extra "|" in them.</del>
<br />
<del>[raymond] Added explanation and removed extra "|" from examples 3-5.</del>

[raymond] Introduced "isNewItem" in the info object instead of adding an empty string at the location where a new property can be added. So striking out the above comments that no longer applied to the new version of info object.

[Glenn] How do you feel about *always* returning the selector info at the current pos? This would eliminate the need for the `findSelectorAtDocumentPos()` function, and it seems like it would be useful information for many types of code hints.
<br />
[raymond] I don't think we want to spend time collecting selector info for all possible cursor positions. And I just make changes to the info structure suggested by nj and we no longer have separate index or values array for selectors. So when we implement info for selector, we will return current selector index and selector array only when the cursor is in selector context.

### Sample code that uses getInfoAtPos API ###
```
    CssPropertyHints.prototype.getQueryInfo = function (editor, cursor) {
        var query       = {queryStr: null},
            ruleInfo    = CSSUtils.getInfoAtPos(editor, cursor);

        if (ruleInfo.context === CSSUtils.PROP_NAME) {
            query.queryStr = ruleInfo.name;
        } else if (ruleInfo.context === CSSUtils.PROP_VALUE) {
            if (ruleInfo.isNewItem) {
                query.queryStr = "";
            } else {
                query.queryStr = (ruleInfo.index !== -1) ? ruleInfo.values[ruleInfo.index] : "";
            }
            query.propName = ruleInfo.name;
        }

        if (query.queryStr && ruleInfo.offset < query.queryStr.length) {
            query.queryStr = query.queryStr.substring(0, ruleInfo.offset);
        }
    }
```

### Usage
See [CSSUtils, HTMLUtils](https://github.com/adobe/brackets/wiki/CSSUtils,-HTMLUtils) for general usage of this API.
