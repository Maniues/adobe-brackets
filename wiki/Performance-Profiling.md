TODO: add more info on using Timeline, repaint/compositing decorations, Profiles, etc. to track performance & memory problems

## Capturing a Timeline Log

1. Open a new tab in Chrome and enter http://localhost:9234 as the URL
2. You should see a link whose text matches the Brackets window titlebar -- click it
3. Select the 'Timeline' tab
    * Note: the next 3 steps should be done without any delay in between!
4. Click the record button in the upper left (a gray dot - turns red once clicked)
5. Go back to Brackets and perform the action that is slow
6. Go back to developer tools and click the record button again (turns back to gray)
7. Right-click anywhere in the area below and select "Save Timeline Data..."
8. Post the JSON file in a [Gist](https://gist.github.com) or upload it somewhere else where it can be shared. Then send us the link.