## Defining Preferences for Code Hints

All the preferences defined are available as code hints in `brackets.json` and `.brackets.json` unless they are excluded.

## Documentation for `definePreference`
`definePreference` was modified to support code hints.

`definePreference(id, type, initial, options)`

- `id` unprefixed identifier of the preference. Generally a dotted name.
- `type` Data type for the preference (generally, string, boolean, number)
- `initial` Default value for the preference
- `options` Additional options for a preference


| Name | Description |
| ---- | ----------- |
| `options.name` | Name of the preference that can be used in the UI.|
| `options.description` | A description of the preference.|
| `options.validator` | A function to validate the value of a preference.|
| `options.excludeFromHints` | `true` if you want to exclude a preference from code hints.|
| `options.keys` | An object that will hold the child preferences in case the preference type is `object`|
| `options.values` | An array of possible values of a preference. It will show up in code hints.|
| `options.valueType` | In case the preference type is `array`, `valueType` should hold data type of its elements.|

### Starting off...
```js
var PreferencesManager = brackets.getModule("preferences/PreferencesManager"),
    prefs              = PreferencesManager.getExtensionPrefs("ext");
```

### Defining a Boolean Preference

**Note:** Since Boolean can either be `true` or `false`, you don't need to pass a `values` array when defining preferences.

```js
prefs.definePreference("foo", "boolean", true, {
    description: "This is a boolean preference"
});
```

### Defining a Number Preference

```js
prefs.definePreference("foo", "number", 4, {
    description: "This is a number preference",
    values: [128, 256, 512, 1024]
});
```

### Defining a String Preference

```js
prefs.definePreference("foo", "string", "bar", {
    description: "This is a string preference",
    values: ["foo", "bar", "baz"]
});
```

### Defining an Array Preference

**Note:** When the preference type is `"array"`, you also need to pass the `valueType` to show the data type of values an array can hold.

```js
prefs.definePreference("foo", "array", ["foo"], {
    description: "This is an array of string",
    valueType: "string",
    values: ["foo", "bar", "baz"]
});
```

### Defining an Object Preference

**Note:** When preference type is `"object"`, you can pass its child keys as `keys` object.

```js

prefs.definePreference("foo", "object", {}, {
    description: "This is an object",
    keys: {
        foo: {
            type: "boolean",
            description: "This is a nested boolean preference"
        },
        bar: {
            type: "number",
            description: "This is a nested number preference",
            values: [128, 256, 512, 1024]
        },
        baz: {
            type: "string",
            description: "This is a nested string preference",
            values: ["foo", "bar", "baz"]
        },
        foobar: {
            type: "array",
            description: "This is a nested array preference",
            values: [128, 256, 512, 1024],
            valueType: "number"
        },
        foobaz: {
            type: "array",
            description: "This is also a nested array preference",
            values: ["foo", "bar", "baz"],
            valueType: "string"
        }
    }
});

```

## Exclusion of Preferences

In case you don't want to allow any preference to show up in code hints, you can exclude it as well. If an `object` preference is excluded then all of its child keys will also be excluded from code hints.

```js
prefs.definePreference("foo", "boolean", true, {
    excludeFromHints: true // This preference won't show up in code hints.
});
```
