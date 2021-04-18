# Development Process

This is a "developer's guide" for our current software development process. This applies to the Adobe Brackets team. 
References to columns in this document refer to our [Trello board](https://trello.com/b/LCDud1Nd/brackets).
### How do we plan our work? ("Backlog Grooming")

We have regular "backlog grooming" meetings to fill up our **Ready queue**. Generally speaking, if there's no space in our Ready state, we don't have a backlog grooming meeting.
##### Before backlog grooming meeting:
1. Take a look at the Tasking and Nomination columns
2. Add any bugs, pull requests and stories that you think are more important than what's already in those columns to the Nomination column.

<a name="backlog_grooming"/>
#####During the _backlog grooming_ meeting, we will:
1. Discuss the nominations, moving any accepted Small nominations directly to Ready. Discussion is guided by the product owner and we may not cover all nominations. Medium sized items which are accepted will move into tasking.
2. Discuss the items in Tasking to make sure we understand what is required and add some thoughts on how we might implement the story.
3. We make sure that the item being discussed is not too large (couldn't be done in roughly one release cycle)
4. For each item, we also assign one person to "task" the story (see below); once tasking is done a checkmark sticker indicates that it may move into Ready – a brief review in a daily standup usually guards that move.
5. Once the Ready column will be full (after stories are tasked), we're done
Note: there may be times when we discuss stories beyond what can fit in the Ready column because the product owner needs to plan for a specific set of stories targeting some time frame.

###Process Cheat Sheet
####Regular Kanban flow
1. Whenever you find yourself idle or blocked, work on one of the following (in priority order):
  a. A non-blocked task you are assigned to on a team card like Release or Post-Release.
  b. A Tasking card you are assigned to (see  Tasking  below).
  c. The rightmost card on the board with a checkmark sticker (see next steps).
       `Exception: If there is a checkmarked card in Testing and you're not the original developer, skip it.`
  d. The topmost item in Ready (see next steps).
2. If you are picking up a card, pull it into the next column to the right.
3. Remove any stickers and previous assignees, and assign yourself to the card.
4. Follow the steps in the table below for the column that you pulled the card into.
5. If at any point you become blocked on what you're working on, move it into Waiting/Dependent with a comment explaining the blockage, and go back to step 1.

| Development | Review | Testing | Bug Fixing/DoD |
|:----------------------|:---------------------|:-------------------|:--------------------|
|If this is a **PR card** skip this column and move this card into **Review** | **1. Assign yourself to the PR** on GitHub. |1. Run smoke tests and unit tests for related areas.|If there are open bugs, fix the bugs for this card that are assigned to you (or, if this is a community PR, work with the submitter to make sure they get fixed).| 
|**Otherwise:** <ol>1. If this is a bug, **assign yourself to the issue** on GitHub.</ol>| 2. Work on the review with the submitter. |2. Do any directed testing mentioned in the card.|After bugs are fixed, if you feel that the story needs more testing, move it back into **Testing**.|
|<ol>2. Work on the story or bug.</ol>|3. If this is a **bug or small PR card**, try to test it thoroughly and have the submitter fix any issues before merging.|3. Do any other focused testing you feel is appropriate.|3. If this is a story, **DoD the story**. If any issues are found during DoD, fix them before continuing.|
|<ol>3. When you have a PR up that's ready for review, add a **checkmark sticker** and leave the card in the Development column.<ol>|***When ready, merge it.*** **After merging:** <ol>1. If this is a **bug or small PR card** and you've done adequate testing during the review, move the card to **Ship It!** </ol><ol>2. Otherwise, add a **checkmark sticker** and leave the card in the Review column.</ol>|**When you're done testing:** <ol>1. If this is a **bug or PR card** and there are no bugs, move it directly into **Ship It!**</ol><ol>2. Otherwise, add a **checkmark sticker** and leave the card in the Testing column.|Move the card into **Ship It!**|

<a name="tasking"/>
####Tasking
"Tasking" a story means coming up with a set of development tasks that need to be done in order to complete a story. An important aspect of this step is to think about whether there are ways to "parallelize" the work. Is it possible for more than one person to work on the story? If it is, that's one way to get the story into Ship It more quickly.

######Let's start with the simpler case where a story doesn't really parallelize well.
1. Add a checklist to the card
2. Add each development task as a step on the checklist
3. If there are special review or testing requirements, add those in the description of the card
4. Once you feel that the implementation details are described well enough, move the card to the bottom of the Ready column

######If the story can be broken up into tasks that can be done in parallel:
1. Move the story card into the **Waiting/Dependent** column.
2. Create new cards for each parallel part.
3. In the description of the card, state that it's a part of the "X" story and the focus of this part of the implementation of the story
4. For each separate card, follow the steps above for adding a checklist and tasks.

####Nominations
Bugs, pull requests, and stories that need to get prioritized start in the Nominations column. Nominations are considered during the weekly backlog grooming for moving into Tasking or Ready.
#####Bugs and non-story Pull Requests
If you find a bug or PR (of any size) that you think is urgent enough to work on in the next week, or a large bug or PR that you'd like to see some movement on soon:

1. Add a card for the bug or PR to **Nominations**.    
  a. If you have an opinion about its priority relative to other items in the list, put it in the appropriate location.
2. If this is a stop-ship bug or PR:
  a. Bring it up in the next standup (or send email if it's really urgent).
  b. If the team agrees it's stop ship, set the milestone in GitHub to the next release number.
  
If you think a bug might be important but aren't sure about nominating it yet, add the Needs Review label, and we'll look at it in the next bug review.
If a bug isn't important enough to work on soon, just set a priority and assign it in GitHub.
#####Stories
If there is a story you think should be worked on within the next two weeks:
1. If there is no card for the story in the backlog, create one in the **Nominations** column.
  a. If you have an opinion about its priority relative to other items in the list, put it in the appropriate location.
####Releases
We want the board to visualize all work in progress, and releases are a bit special. They have a number of small, largely independent tasks. Because the tasks are small and mostly parallel, we track all of the tasks on one card and assign them to various people on the team. Most tasks do not take full-time work to accomplish, so release cards do not count against WIP limits.

There are two release cards:
1. Release – all of the steps required to get a build out the door
2. Post-branch – regular tasks that we do after we've branched for a release
