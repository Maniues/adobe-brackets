Inline Color Editor is implemented as an inline widget that pops up after the invoked line in the main editor. Since it is an inline widget, the current implementation has the following two usability issues.

- It cannot be invoked from any inline text editor because Brackets does not support nested inline widgets yet. 
- Cannot have two inline color editors open from two separate colors on the same line. Invoking the second one will override the first one in the widget.

Current implementation supports all different color formats including named colors. And it can be invoked in any language mode only if the currrent context matches to a valid color. When invoked, the color editor grabs the current context and checks it against with a regular expression. The regular expression is defined in utils/ColorUtils.js with an array of named colors and all possible valid color formats. Once the context matches a valid color, then an instance of InlineColorEditor is created. Otherwise, the invocation is ignored. So current implementation does not allow users to create a new color or correct an existing invalid color in the inline color picker.

### InlineColorEditor.js
This is the file that implements the InlineColorEditor class which is derived from InlineWidget base class with the following methods.

- load(hostEditor) 
	- Creates a new instance of ColorEditor that loads the color picker UI from ColorEditorTemplate.html
	- Collates existing colors from host editor and sorts them based on usage frequency
    - And creates a new instance of ColorEditor

- onAdded 
	- sets up a handler to watch the manual updates of the selected color in host editor

- onClosed
	- cleans up text marker, “change” event handler and ColorEditor created from template

- getCurrentRange
    - Returns the current text range of the color we're attached to, or null
- _handleColorChange
	- Updates text in host editor when the color value or format is changed in the color picker.
- _handleHostDocumentChange
    - Updates the color value in the color picker when there is a manual change to the selected color in host editor


### ColorEditor.js
This file is the implemenation of `ColorEditor` class and all the necessary methods that handle user interactions through the color picker UI. This class uses the third party JavaScript library `tinycolor` to do the conversion between different color formats.




