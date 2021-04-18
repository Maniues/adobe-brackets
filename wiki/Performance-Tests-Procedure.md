Sprint DoD has a task for logging performance tests results. We have a small set of performance tests that run within the unit test window. Eventually these tests will be automated. For now, here are the instructions for logging the sprint performance results:

1. Confirm you have edit access to [Brackets Performance Tests (Google Spreadsheet)](https://docs.google.com/spreadsheet/ccc?key=0Aras0diokeHxdEc5RGtOeVI0V0xGU3FPUXBuX3ZYTlE#gid=0). If not, ask @jasonsanjose.
2. Get the performance laptop (get key from @pflynn's desk)
3. Uninstall any existing Brackets, and install the new Brackets build
4. Sync the brackets repo on the machine to match the SHA of the installed build (look for the version field in <path to install>\www\package.json)
    * Don't forget to run `git submodule update` too
    * Note: You may need to open the SSH tunnel to access GitHub
5. Run ``tools/setup_for_hacking.sh``, passing it the Brackets .app folder
6. Delete the [[Cache Folder]]: `rm -r ~/Library/Application\ Support/Brackets/cef_data/`
7. Reboot the computer (wait for other apps like the browser to finish auto-restoring)
8. Launch Brackets
9. Debug > Show Performance Data
10. Log "Application Startup" time under "Startup > _Cold_ Startup (first run, no prefs)"
11. Debug > Run Tests, Performance, All
12. Log "Performance Tests File open performance" under "<i>Cold</i> - Performance Tests File open performance"
13. Log the _first_ set of "JavaScript Inline Editor Creation" results under "<i>Cold</i> - JSQuickEdit Performance site should open inline editors."
14. Log the _last_ set of "JavaScript Inline Editor Creation" results under "<i>Warm</i> - JSQuickEdit Performance suite should open inline editors."
15. Exit and restart Brackets
16. Debug > Show Performance Data
17. Log "Application Startup" time under "Startup > _Warm_ Startup"
18. Debug > Run Tests, Performance, Performance Tests
19. Log "Performance Tests File open performance" under "<i>Warm</i> - Performance Tests File open performance"
20. Go through each chart tab in the spreadsheet, click Advanced Edit, click the Start tab, and adjust each data range to add your new column (i.e. increment the letter after each ":").