This document proposes a set of user and extension developer workflow scenarios that we want 
the Brackets Extension Manager to support.

## Discovery

John, a Brackets user, is starting to experiment with CoffeeScript. He opens a CoffeeScript
file and sees that it's color-coded, but there doesn't seem to be any other built-in support
for it. He wonders if there are Brackets extensions available that add CoffeeScript-specific
features.

In the toolbar on the right-hand side of the Brackets window, the topmost icon is the
Extension Manager. He clicks on the icon, and an Extension Manager panel appears to
the right.

At the top of the Extension Manager are two tabs, "Find" and "Manage", with the
"Find" tab preselected, and buttons to the right labelled "Create" and "Upload". 
Within the Find tab is a search field, and below the field is a list of popular 
extensions, sorted by rating. He briefly browses this and notes that there's a 
cool-sounding extension for GitHub integration that's popular; he clicks the 
Install button on it, and it starts installing in the background. 

While it's downloading and installing, he continues his search for CoffeeScript 
extensions. He types "coffee" into the search field. As he types, the list of 
popular extensions in the main part of the window is filtered to show a list 
of all extensions that match his typed search, also sorted by rating.

He sees that there are three extensions that sound interesting at the top of the list: 
Auto-Compile CoffeeScript, CoffeeScript Quick Open/Quick Edit, and CoffeeScript
Formatting. Each extension has a screenshot thumbnail, a title, the first few lines 
of a description (with a "More..." link), and an "Install" button on the right.

He clicks on the "More..." link on the first extension to see more info. It shows
the full screenshot and more of the description text, which explains the basic usage 
of the extension, along with a link to the extension developer's website for more info.

There is a fourth extension listed, CoffeeScript Lint, but the "Install" button is disabled and there's a message stating that CoffeeScript Lint is only compatible with earlier versions of Brackets. John considers following the developer's website link to ask about a newer version, but decides he'll just try out the other three extensions to start with.

## Installation

He doesn't feel like he needs to read any more for the other extensions, so he clicks 
on each of the "Install" buttons on these three extensions. Each button changes to a 
grayed-out "Installing..." button.

A progress bar appears at the bottom of the window with the label "Installing
3 extensions". As they complete, a checkmark appears next to each extension indicating 
that it's been installed, and the "Install" button on each extension is replaced by 
two buttons, "Disable" and "Uninstall".

Also, next to the "CoffeeScript Formatting" extension, there is a "Settings" button.
He clicks on this button, and a dialog expands in-place with various formatting
settings for CoffeeScript code. He tweaks it to fit his style. Finally, he clicks
the Close button on the Extension Manager dialog to close it.

Now, back in his CoffeeScript file, he tries hitting Cmd-E on a function call, and
an inline editor opens on the function definition. Success! He also browses the
menus to see what other functionality has been added by his new extensions.

## Feedback

As he starts using the extensions, he notices that they mostly work well, but the
CoffeeScript Formatting extension doesn't seem to work properly&mdash;when he tries to
use it, his code gets mangled. He opens the Extension Manager and clicks on the
"Manage" tab, which shows all of his installed extensions. He clicks "Disable"
next to the CoffeeScript Formatting extension. He also sees that there's a "Report
a Problem" dropdown button, along with a rating widget. 

He wants to give the extension developer the benefit of the doubt, so rather than 
rating it, he clicks "Report a Problem", which takes him to the developer's GitHub 
issue tracker.

He's been really impressed with the Quick Open/Quick Edit extension, so he clicks
on the rating widget next to that one to give it a 5. Brackets prompts him to
login using OpenID in order to rate it. He does so, and chooses to let Brackets
remember his credentials.

## Update

The next day, John sits back down at his computer and brings Brackets to the
front. He sees that there's a little "2" badge on the Extension Manager icon.
He clicks on the Extension Manager, and a notification at the top informs him
that an update is available to the CoffeeScript Formatting extension and also to
the "Hover Preview" which he installed recently.

The update descriptions describes the CoffeeScript Formatting bugs that were fixed in the 
update--one of which was his bug! He clicks on the "Update" link in the notification, and the update is downloaded and installed. Since John had previously disabled this extension, Brackets prompts him 
if he wants to re-enable the extension now that it's been updated, and he clicks Yes.

When updating Hover Preview, Brackets warns him that the HTML Image Preview extension
will be disabled because it is not compatible with the new version of Hover Preview.
John decides to go ahead and update Hover Preview anyhow.

Closing the Extension Manager, he tries the formatting command again, and
lo and behold, it works! He goes back to the extension developer's bug
tracker and lets him know the bug was fixed, and gives the command a rating
for good measure.

## Developing a new extension

While he's working on his CoffeeScript file, John is curious about the JS
that's being generated, and realizes that it would be cool to have a Brackets
extension that shows the generated JS for the current function as you type.
He figures he can write it himself.

He remembers that there was a "Create" button in the Extension Manager, so
he opens it up and hits "Create". A dialog appears prompting him for the
name of the extension, and showing that it will be created in his "dev"
folder. He enters "CoffeeScript Preview" as the name and clicks OK. Brackets
creates the extension project in his "dev" folder and switches to it, and
he sees in the file tree that a main.js, a package.json, a README, and other 
boilerplate files have already been created for his extension. 

He starts writing and debugging the extension, and loves how easy it is to 
get it working. As he's working on the extension, he checks it directly into 
GitHub from Brackets using the extension he had installed earlier.

## Uploading

When he's happy with the extension, he wants to share it with others. He goes
back into the Extension Manager and clicks on the "Manage" tab. Inside that
tab, along with the other extensions he'd installed, he sees the extensions in
his "dev" folder with a special "unpackaged" icon, and an "Upload" button next 
to it. (He could also have just clicked the main Upload button and chosen the
folder his extension is in, but since it's in his dev folder, Brackets sees it
automatically anyway.)

He clicks on the "Upload" button next to his extension, and the extension is 
packaged and uploaded to the Brackets registry, using the metadata from his 
package.json file, and registering it using the OpenID he had logged in with 
earlier. The icon changes to indicate that he is still using his own development 
version of a registered extension.

> **UNRESOLVED**:
> (pf) Can an author manage existing uploaded extensions? For example, pull an extension (or version thereof) from the registry if they realize something is badly broken or mis-licenced?

> (nj) Great point. I think we need to brainstorm around this a bit.

> (kd) perhaps we just provide contact information for the Brackets team somewhere in the publishing UI? I'm guessing the number of these requests will be pretty small.

## Brackets Update

A new Brackets version came out and John upgraded his Brackets. The CoffeeScript Formatting extension was not compatible with the new version of Brackets. As a result, the following happened:

1. The extension was disabled
2. The toolbar has an indicator on the extension manager icon reflecting that an extension has a problem
3. In the extension manager, there is a message for the CoffeeScript Formatting extension saying that the installed version is not compatible with the version of Brackets and that it has been disabled
4. As soon as an update to the CoffeeScript Formatting extension becomes available, John is given the option of updating and re-enabling the extension

# Historical content #

What follows is discussion that was contained within the original document. This section will be removed once the dust has settled a bit.

> **RESOLVED**:
> (kd) General question: do we want to add social sharing (so that users can share an interesting extension on Twitter, Facebook, G+)?

> (nj) Seems like a second-tier feature. I'd like to keep this workflow to (mostly) first-tier
> features for now--I don't feel like adding sharing will significantly affect the overall UI design
> (it would probably just be another button associated with the extension, perhaps in the "More..."
> section). I've added this to the list of desirable features in 
> [Extension Package Manager Research](https://github.com/adobe/brackets/wiki/Extension-Package-Manager-Research).

> **DEFERRED**: (don't need to decide form factor now, doesn't significantly affect overall workflow)
>
> Should this be in a panel or a popover? Garth's wireframes suggest
> a panel. NJ thinks that makes it feel too permanent, and that a popover would be
> consistent with where we're likely to go with EWF (as in Reflow), which seems
> kind of similar (large browsing interface that only needs to appear temporarily).

> (re) The Extension Manager dialog doesn't need to be *modal*, correct? Is that just 
> for simplicity of initial implementation?

> (nj) No, I just wasn't thinking about it too hard :) Garth suggests using a side
> panel; I was thinking of a popover. Shouldn't affect the workflow all that
> much either way, but we should resolve it before moving forward.

> (pf) I don't see why modal is bad. You're not going to be bopping in & out of this thing constantly. It should probably be easier to stay in (harder to close accidentally) rather than optimized for getting out quickly...  I agree a panel feels wrong though.

> (nj) Another option is to have it open as a full view that takes over the editor
> area, as if it were another item in the working set. Overall it doesn't seem like
> we need to resolve this choice right now to evaluate the high-level workflow--we 
> can deal with it later when we get to more detailed design work.

> **RESOLVED**:
> Note that this implies that extension download and installation is asynchronous and
> non-modal as well as restartless. 

> **RESOLVED**:
> (re) It might also be helpful to be able to sort/filter extensions by number of 
> downloads, recently uploaded/updated, or Author

> (nj) Agreed, though I don't think those need to be called out in this workflow.
> I've added this to the list of desirable features in 
> [Extension Package Manager Research](https://github.com/adobe/brackets/wiki/Extension-Package-Manager-Research).

> **RESOLVED**:
> (kd) is it limited to just the popular extensions, or will it put the popular/highly rated on top
> and seek out all of the coffee extensions?

> (nj) It will show all matching extensions, but just sort them by popularity. We
> could add a way to change the sort order (e.g. by Quick Open match heuristic or
> alphabetically).

> (pt) Yeah, I agree it might be valuable to be able to sort by criteria like rating 
> or date published (newest)

> (nj) Yup, added that to the "desirable features" list as noted below Randy's comment above.

> **RESOLVED**:
> (kd) if there are screenshots, are any displayed in thumbnail form in this view?

> (nj) That makes sense, but I actually wonder how meaningful a small thumbnail of a
> screenshot would be--they might all kind of look the same. I've added it to the
> description above--we can see if it works out.

> (pf) We could encourage people to use a cropped & zoomed version instead of a literal downscaled thumbnail. I assume even after clicking "More" we don't show a full-resolution screenshot though right? What resolution are we expecting/enforcing? Can you click the image to see a 1:1 full-res copy?

> (nj) I think we'd leave that up to the developer--they could specify both a thumbnail 
> and a full version, and if a thumbnail isn't specified we could just downscale the full 
> version.

> **RESOLVED**:
> (pf) Other details we might want to show: download count, date last updated, author (GitHub username?). I assume the rating is shown without needing to click More? Perhaps some of those other blurbs should be foregrounded too.

> (pf) B feature: "More by this author" link -- that's very useful in mobile app stores.

> (nj) Added these to the general extension manager features doc. We can figure out which
> of these should appear initially vs. in the "more" screen when we get to detailed design.

> **RESOLVED**:
> (js) Have we considered out-of-app workflows for discovery? Tying into Kevin's thoughts
> on social sharing, it seems useful to have an in-browser discovery experience that
> might use a URL scheme to launch Brackets or even provide a manual download link.

> (nj) That seems useful but second-tier to me...although if we know we want to do it
> eventually, that might affect how we write the code for the in-Brackets version
> (i.e., make sure as much of it as possible is reusable for a browser version). I'll
> add it to the list of desirable features in
> [Extension Package Manager Research](https://github.com/adobe/brackets/wiki/Extension-Package-Manager-Research).

> **RESOLVED**:
> (kd) has there been a change in which tab he's on to see disable/uninstall?
> Those would be in the Manage tab, right?

> (nj) Good point. I guess I was assuming that the view of a given extension would be
> the same no matter which tab you're in (and I don't think it would make sense to
> *eliminate* installed extensions from the "Find" view). But then that suggests that the 
> "Manage" tab isn't really necessary, in a way&mdash;the list of installed extensions 
> could just be another filter on the list of all extensions. Would it make sense to just
> have a single view with the search box, and then an "All" vs. "Installed" radiobutton?
> Or is it clearer to have separate tabs?

> (nj) For now, let's keep the tabs--I think it makes things clearer even if you could
> theoretically treat the "installed" list as just another filter.

> **RESOLVED**:
> (pf) I think tabs labeled Find/Installed or All/Installed would make more sense than Find/Manage if the two views are so similar.

> (nj) I currently have "Find Extensions" and "Installed" in the wireframes, but we can
> tweak this in the final design.

> **RESOLVED**:
> (pf) Re '"Installing Auto-Compile CoffeeScript (1 of 3)..."': this implies download, unzip, and init are serial if you have clicked multiple extensions? Or just some of those steps? Can you cancel a download if it's too slow or stalled?  Seems especially important if it's serial -- otherwise you're hosed on installing other extensions until it times out.

> (nj) Good point. I was assuming we'd want to serialize these to avoid the various
> installations overlapping each other, but in retrospect there probably isn't any
> particular reason to do that. I'll change this to just say "Installing 3 extensions..."
> for now.

> **RESOLVED**:
> (pf) Do we envision having a Prefs dialog eventually? Would be expose each extension's settings UI there too, or would the user have to go to different places for core vs. extension settings? (Maybe eventually the Settings... button could just pop open the Prefs dialog with that panel pre-selected...)

> (nj) Yup, if we ended up with a single prefs dialog we'd link to it from the
> extensions anyway.

> **RESOLVED**:
> (pt) Not sure if we need to display a license while the user is installing an 
> extension, we should check with legal on that.

> (nj) I was assuming we wouldn't need to since app stores, Firefox's add-on
> manager, etc. don't seem to do that. The developer can add license info to
> the description.

> **RESOLVED**:
> (pf) "Report Version Incompatibility" sounds cool but maybe B-feature level. I also wonder whether users would understand the distinction between that and "Report a Bug" (incompatibility is a bug too...).

> (kd) I share the concern that it may be hard to understand when to click on which button. I was thinking of something along the lines of "This extension used to work, but doesn't any more". This is only a B-level feature depending on the extensions dependency work.
> We should probably define the "Report Abuse" flow somewhere. I'm thinking we should prompt the user for more information. Additionally, what happens on the other side? Definitely an email notification. Do we disable the extension right away? (I'm thinking yes. I'd rather be conservative to start with and then if "Report Abuse" proves to be problematic, we fix it.)

> (nj) For now, I'll just change this to a simple "Report a Problem" button. We
> can figure out the details later.

> **RESOLVED** (in that we probably need login for the workflow, but the actual
> mechanics need to be figured out):
> 
> We probably need some sort of login for ratings to avoid abuse. I'm assuming
> OpenID makes the most sense, but Facebook, Twitter, G+, etc. logins might be
> more likely to be convenient.

> (kd) This is going to require a little thought about the authentication flow,
> since we're not in the browser. If we open up an iframe to Twitter within Brackets,
> for example, the user would have to actually authenticate. I wonder if it's possible
> to open the user's browser, have them approve the oauth request there, and then
> return them to Brackets where they're logged in.
> After writing that, it seems a bit worse to do it that way than to have them type
> in their Twitter username/password in a Brackets-controlled window.

> (nj) Yeah, I feel like I've seen apps that pop up their own Facebook, Twitter, etc.
> windows, authenticating within that app window, and then caching the credentials
> in that app (though I don't even exactly know how the credential stuff works--I'm
> assuming there's a token that the app hangs onto that expires after some time).
> Since we *are* a browser (but not in the user's browser), I'm assuming that we
> could just do this the same way a normal web app does.

> Also, I'm not sure how credential remembering would work&mdash;would we need to
> have it time out after awhile?

> (kd) seems like a good question for Andrew. For the purposes of rating/reviewing things,
> I'm not so worried. For publishing extensions, it's more of a concern.

> (pt) I share Kevin's concerns around the publish workflow. Beside that, as a user 
> I would find reviews or a kind of a user rating (wrote x extensions with an average 
> rating of z) useful too.

> (nj) If things work the way I mentioned above, then presumably we could just hold
> onto the auth token until it expires (which is presumably controlled by whichever
> sign-in service provided the token).

> **DEFERRED**: (we do need to figure this out but it doesn't affect the user workflow
> significantly--either way they will need to authenticate)

> (pf) Opening an iframe onto the web within our app shell feels a little scary. What if the login UI has links to other pages (e.g. forgot password, etc.)? Will that all work properly? Is there any risk of landing on an untrustworthy page that might abuse the app shell APIs? Seems hard to get right. What do 'normal' native apps do? I'm guessing many show their own native UI to enter credentials and then the app hands them to the server itself; you have to trust the app a bit more since it then knows your cleartext password, but the app stays walled off from the web in the bargain...

> (kd) on iOS, the typical app seems to open a UIWebView, which wouldn't have any privileged access. OAuth is definitely nicer (and likely better supported) than taking the user's username and password for a third party service. I'll write more about authentication, but the more I've thought about it, the more I think we should treat oauth support as a separate story.

> (nj) I feel like I've seen that in some desktop apps as well now--where the app just
> opens a browser window hosted in the app. However, in our case we would need to make
> sure that any such window doesn't have access to Brackets APIs--i.e., we can't just
> do `window.open()`. We'd need to add something to the native shell that creates a
> window that doesn't have access to our v8 extensions or node server. All that said,
> I don't think we need to figure this out for the workflow--the main thing is that
> the user will need to authenticate somehow.

> **RESOLVED**:
> (pf) Is the icon size in Topcoat big enough to accommodate a legible number badge?

> (nj) Need to figure that out as part of the detailed design.

> **RESOLVED**:
> (pf) Instead of a message at the top of the dialog, what about an "Updates" tab? Seems like that'd work better when multiple updates are available. (And clicking the tab could trigger us to ping for updates if we haven't already done so recently).

> (nj) That's a good idea--it doesn't seem like it would fit in the panel, but it
> could fit in the full-screen example. I'll add an "Updates" tab to that version.

> **RESOLVED**:
> (pf) The workflow here implies restartless extension removal -- which I still think is _much_ harder to do well than restartless addition (installation). Are we totally sure we want to bite this much off right away? (Is there an expectation that everything in this doc will be implemented by May?). How many other editors/tools/browsers do restartless extensions widely? It seems like a _subset_ of FF extensions support it; but FF leaves most of the tricky work up to devs, and devs there have much stronger incentives to get it right (the extensions code-debug cycle is _way_ more painful otherwise -- not true in Brackets).

> (nj) Regarding whether all this will be done by May, I think the answer
> is clearly "no" :) We still need to slice this into a set of smaller stories,
> but the intent is for this to cover most of the things we think are high-
> priority features (even if they don't get done by May).

> (kd) I'm not convinced that "good enough" restartless extensions are going to be *that* difficult, and we'll find out soon in the extension API research. That said, one thing that is not called out in this document is current extensions which decidedly *do* require a restart. I think we should be explicit about whether current extensions are managed at all via this UI.

> (kd) If we do decide to support both restartless and restart-requiring extensions, the UI need only change a little bit (as Firefox's does). When installing/updating a restart-requiring extension, the user is prompted after the installation is complete to restart Brackets. Since the actual installation happens in the background, this would probably be in a popover.

> (nj) Let's work out whether current or non-restartless extensions need to be supported
> as we come up with the criteria for the user stories. In the UI, I think we would
> want to do two things: (1) an indication before the user installs a restart-requiring
> extension--e.g. a small popover that appears as soon as you mouse over the Install button
> saying "Restart Required"; (2) after such an extension is installed, a warning icon and
> notice in the status bar (where the installation progress bar was) saying something like
> "One or more installed extensions requires restarting Brackets."

> **RESOLVED**:
> (js) My interpretation here is that we are promoting the Create workflow pretty highly.
> I'll reserve judgement once we get to mockups, but I'm not sold on foregrounding this
> since most users won't be authoring their own extensions. Maybe we instead have a
> Brackets developers extension or some other mechanism to enable this workflow without
> adding noise for our typical user.

> (nj) That was actually deliberate in that I think we *want* to encourage users to
> think of themselves as potential extension developers (at least in Brackets).
> I don't think adding a single "Create" button to the UI adds that much noise, and
> the other bits of the workflow only kick in once you've clicked that button.

> (pf) This feels a little orthogonal or B-featurey to me... While it'd be awesome to have a "new extension wizard" or whatnot, it doesn't seem key to the extension management workflow (while Upload clearly is critical). Also, is there a strong enough lead-in from Create to Upload? Upload seems sort of hidden.

> (nj) The idea is that if they were to create an extension in their dev folder,
> it would automatically show up here (regardless of whether they used the Create
> button or not). But we probably also need a button for just uploading an
> extension from any folder. I've added a note at the top that the Extension Manager
> has an "Upload" button next to the "Create" button, and the full-screen wireframes
> (there isn't room in the panel-based wireframes), and clarified below that he
> could have used this global button as well. We could make the Create/dev folder
> workflow a B feature.

> **RESOLVED**:
> (pf) I think we need a section here on security. Letting semi-anonymous third parties upload text that will be rendered in the HTML content of _every_ Brackets user has enormous risks if it's not all escaped religiously.

> (nj) I don't think that affects the workflow specifically--it's something we'll
> need to figure out for sure when we spec it out for real, but I think we need to
> assume that extension descriptions will show up in the Brackets UI, and plan to
> sanitize accordingly.

> **RESOLVED**:
> (pf) Where in the workflow does the user enter the description, big tracker URL, upload thumbnail/screenshot, etc? Or are we expecting they include it in the repo and link to the right files in package.json?

> (kd) These could be in the package metadata (even URLs for screenshots).

> **RESOLVED**:
> (pf) Should we describe the update workflow? Can the author enter 'update notification' text? Do we aggregate it across skipped versions like Brackets core does?

> (nj) That's in the Update section above already, where it mentions that the update
> lists the bugs that were fixed in the update. I don't think we would necessarily add
> any specific functionality to aggregate it, etc.--we would just show whatever the
> author included as update info, and the author could choose whether or not to keep
> older info in.

> **RESOLVED**:
> (pf) Have we considered whether this should be more tied into GitHub? It seems more natural to have the author's ID be a GitHub ID, for example. And that we ensure the source of every extension is easily accessible. The current workflow makes it easy to upload & share extensions that are sitting on your local disk, and makes it hard for other users to inspect the source before installing. (And if we used GitHub repos as the means of giving your source to us, it'd be easy for us to link to a specific SHA tree and feel confident that it's exactly the bits we're about to install -- rather than trusting the author that the ZIP file they just uploaded to us is identical (or even related) to the source available at the homepage link they provided in their metadata.

> (kd) That's a good question. I don't think we'd want to assume that *all* extensions are on GitHub. Consider extensions written to support an internal company workflow.

> (nj) I agree with Kevin--we shouldn't tie this fully to GitHub. However, perhaps
> there are fun things we could do if the extension were associated with a
> GitHub repo--like providing you with a button to fork the source into your
> dev folder so you can hack on it :)

He clicks "Disable" on his development version, and then chooses to install
the "official" version he just uploaded to test it out. It seems to be working
fine, so he disables the official version and re-enables his development version
so he can keep hacking on it.

> **RESOLVED**:
> This is a little funny&mdash;not sure if this is the right way to handle this
> scenario or if there's something smarter we want to do here.

> (kd) I guess we could handle this case specifically. In addition to the "Upload"
> button there could be a "Try Published Version" button that acts as a toggle
> between the released version and the in-development version.

> (re) If the version number was displayed then I think it would be easy to see. 
> For example, it would be be easy to see that v1.0.0 is the published version and 
> v1.0.1 is the in-development version.

> (nj) Makes sense. For now, let's not add anything special for this case.

> (pf) Seems like a fairly non-critical use case anyway: unless uploading munges the bits in some way, the published version should basically be guaranteed to behave identically to your local bits at time of upload... 

