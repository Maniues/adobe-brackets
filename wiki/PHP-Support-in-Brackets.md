### We've introduced the PHP support in Brackets 1.14, powered by the [PHP language server](https://github.com/felixfbecker/php-language-server).

**Note:** The PHP features will only work if Brackets is able to find a valid PHP7 runtime in the system path, or if a valid PHP7 executable path configuration is set in `brackets.json`. Click [here](https://github.com/adobe/brackets/wiki/PHP-Support-in-Brackets#php-in-brackets-can-also-be-configured-using-custom-settings-in-bracketsjson) to understand about other PHP settings.


### PHP in Brackets can be configured using custom settings in `brackets.json`:
- Go to `Debug > Open Preferences File`

```javascript
// PHP Tooling default configuration settings
"php": {
        "enablePhpTooling": true, //false to disable PHP features
	"executablePath": "php"//Path format: "C:\\path\\to\\php.exe" for WIN  or "/Users/someuser/bin/php" for MAC & Linux
	"memoryLimit": "4095M", //Specify a memory limit for the PHP language server process
	"validateOnType": "false" //Configuration to have diagnostics "on type" or "on save"
}
```
**Note: Don't forget to remove comments while using as valid JSONs can't have comments.**

### Brackets supports the following features for PHP:

- **Code Hinting**  - Open a PHP file and just get going...
  <img src="https://github.com/shubhsnov/brackets/raw/LSP-Images/codehints.gif" alt="Code Hinting" width="750px">
- **Parameter Hinting** - Contextual parameter hints tell the user where they are in the call context
  <img src="https://github.com/shubhsnov/brackets/raw/LSP-Images/parameterhints.gif" alt="Parameter Hinting" width="750px">
- **Jump to Definition** - `Ctrl-J` and you're all set...
  <img src="https://github.com/shubhsnov/brackets/raw/LSP-Images/jumptodef.gif" alt="Jump to Definition" width="750px">
- **Linting** - Diagnostics 'on type' or 'on save'...
  <img src="https://github.com/shubhsnov/brackets/raw/LSP-Images/linting.gif" alt="Linting" width="750px">
- **Find References** - Place the cursor and just right click or `Shift-F12`...
  <img src="https://github.com/shubhsnov/brackets/raw/LSP-Images/findreferences.gif" alt="Linting" width="750px">
- **Find Document Symbols** - `Ctrl-T` to list all the symbols in the current document...
  <img src="https://github.com/shubhsnov/brackets/raw/LSP-Images/DocSym.gif" alt="Linting" width="750px">
- **Find Project Symbols** - `Ctrl-Shift-T` to list all the project wide symbols...
  <img src="https://github.com/shubhsnov/brackets/raw/LSP-Images/ProSym.gif" alt="Linting" width="750px">

