Native crash logs are useful for diagnosing cases where the Brackets window disappears or goes blank suddenly.
(If Brackets is misbehaving in other ways, please see [[Troubleshooting]] for other help).

## Gathering a Crash Log

### Mac

1. In Finder, choose _Go > Go to Folder..._ in the menu
2. Enter `~/Library/Logs/DiagnosticReports`
3. Look for a file that says "Brackets" in the name, with a timestamp matching the time of the crash
4. Post the file's contents in a [Gist](https://gist.github.com) and include the link in your bug report (or zip up the files and share them privately)

### Windows

1. On the Start menu, search for and run the "regedit" app
2. Navigate down the tree and select: "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\Windows Error Reporting"
3. If there is no "LocalDumps" folder, choose _Edit > New > Key_ to create it
4. Inside the "LocalDumps" folder, choose _Edit > New > Key_ and create a subfolder called `Brackets.exe`
5. Inside this "Brackets.exe" subfolder, choose _Edit > New > DWORD_.  Enter `DumpType` for the name, then double-click it and enter `1` for the value
6. Close regedit and restart your computer (or at least fully log off and log back on, which is almost as much work)
7. Run Brackets until you see the problem again
8. In a Windows folder view, enter `%LOCALAPPDATA%\CrashDumps` as the path
9. Look for a .dmp file whose timestamp matches when you saw the problem.
10. Upload this file somewhere and include the link in your bug report (or zip up the file and share it privately)


## Gathering a Freeze Log

If Brackets is still running, but has become unresponsive, follow these steps instead:

### Mac

TBD

### Windows

1. Download the [procdump utility](http://technet.microsoft.com/en-us/sysinternals/dd996900.aspx)
2. Open Windows Task Manager and switch to the Processes tab
3. Locate the _two_ Brackets.exe processes and write down the PID (process id) for each
    * You may have to use View > Select Columns to show the PID column
4. Run `procdump <PID>` once for each of the two PIDs
5. Locate the two .dmp files - in the same folder as procdump.exe
6. Upload these file somewhere and include the link in your bug report (or zip up the files and share the zip privately)


## Analyzing a Log

### Mac

TBD

### Windows

1. Get the .pdb file (symbol table) for the build of Brackets that the crash log came from
2. Launch windbg (x86 version)
3. File > Symbol File Path:
   ```
   SRV*<path to store cache in>*http://msdl.microsoft.com/download/symbols;<path to folder with .pdb file>
   ```
4. File > Open Crash Dump (usually found in `%LOCALAPPDATA%\CrashDumps`)
5. Run `!analyze -v` to get exception type, stack trace, etc.
6. If the stack ends in our code (not CEF or OS), run `dv` to get local variables' values