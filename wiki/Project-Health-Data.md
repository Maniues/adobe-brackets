**Applies to:** Brackets 1.3 and newer

### Introduction

[Read the full blog post here](http://blog.brackets.io/2015/03/27/introducing-brackets-health-report/).

Understanding how people use an app helps decide what to build, prioritize when to build it, spot usability problems, and find lurking bugs. As the Brackets project has grown, we need real-world data to get a better picture of how people are using Brackets so we can continue making it even more awesome.

But Brackets isn't just any app – it's open-source, and for a developer tool privacy must be the absolute #1 priority. We only want to gather information in a way that is _transparent and respectful_ to our users. We've looked to other open source projects (particularly the [Firefox Health Report](https://blog.mozilla.org/metrics/2012/09/21/firefox-health-report/)) as models. The Data Adobe is collecting can be seen via _Help > Health Report_.

Brackets Health Report is:

* **pseudonymous** – the data sent will never include your identity or private information like filenames.
* **Aggregated** – Brackets does not send individual events – only averages and totals.
* **Transparent** – [the code is open-source](https://github.com/adobe/brackets/tree/master/src/extensions/default/HealthData), and you can view the data Brackets is sending via _Help > Health Report_. We'll openly share what we learn from the data so the whole community can benefit.
* **Minimal** – every piece of information has a purpose directly tied to making Brackets better.
* **Optional** – you can always opt out of the Brackets Health Report. But for all the reasons above, we urge you not to!

We're always open to feedback on this feature and how we can make it both more useful to the community, and more comfortable for everyone to use. Please join the community discussion on the [brackets-dev forum](https://groups.google.com/forum/#!forum/brackets-dev) to share your thoughts!


## Health Report Preferences

**To opt out**, choose _Help > Health Report_ and uncheck the checkbox.

Brackets sends one Health Report update every 24 hours, only while Brackets is running. To view your latest Health Report snapshot, choose _Help > Health Report_ and check out the JSON data shown.


## What Data is Collected?

* `uuid` - A randomly-generated, pseudonymous id
* `snapshotTime` - Time the Health Report is sent
* `os` - OS
* `bracketsVersion` - Brackets version
* `userAgent` - OS version (embedded in user-agent string)
* `osLanguage`, `bracketsLanguage` - Brackets locale and OS locale
* `installedExtensions` - List of installed extensions and their versions - _only_ those that are already published in the [extension registry](https://brackets-registry.aboutweb.com), so non-public or not-yet-released extensions are kept private
* `bracketsTheme` - Current Brackets Theme
* `fileStats` - Counts the number of times a file extension known to brackets or any of its extensions is opened. Custom file extensions are not registered.
* `encoding` - Counts the number of files opened using different encodings.
* `ProjectDetails` - Details of the number of files in a project and the size in bytes.
* `searchDetails` - Statistics related to find/replace in files and instant search.
* `prefNodeSearchDisabled` - Check if user has enabled/disabled Node search.
* `prefInstantSearchDisabled` - Check if user has enabled/disabled instant search.
* `AppStartupTime` - Time taken to start Application.
* `projectLoadTimes` - Time taken to load the project.
* `fileOpenTimes`
* `environment` - Type of environment (Default is Production).
* `time` - Time at which data was collected.
* `event.guid` - A randomly-generated, pseudonymous id.
* `event.user_guid`- A randomly-generated, pseudonymous id.
* `event.category` - Category of event.
* `event.subcategory` - Subcategory of event - (JS Refactor, Quick Edit, Quick Docs, Auto Update, Live Preview, Project Settings, externalEditors).
* `event.type` - Type of event.
* `event.subtype` - Type of Entry point for a particular event. 
* `event.user_agent` - User Agent.
* `event.language` - Brackets locale and OS locale, 5 character code.
* `source.name` - Brackets version.
* `source.platform` - OS from which data has been collected.
* `source.version` - Brackets version.


**Benefits** - with this data, we can answer questions like:

* How many people are using Brackets each month?
* Among people who don't stick with Brackets, how long do they try it out first?
* Which extensions are popular with regular Brackets users? This is can be helpful for both extension authors and for an idea of where we should expand the Brackets core.
* Are we paying the right amount of attention to each of the platforms we support?
* Which languages should we pay the most attention to for translations?

## Data Learnings

Because Brackets Health report is just rolling out, we don't have any real data to talk about yet.

But [this spreadsheet](https://docs.google.com/spreadsheets/d/1xCQh0Ii-MaVcqn-MdF-zDFy3quDBhEHZPeiOEmmITgM/edit?usp=sharing) shows some interesting, more limited data about Brackets and how the project has grown over time. See [this blog post](http://blog.brackets.io/2015/03/27/introducing-brackets-health-report/) for more info.


# Future Plans

Brackets Health Report will evolve to collect other useful statistics in these categories:

* Configuration (e.g. Brackets version) - helps put the other data in context
* Extensions & themes - lets extension authors understand how widely their projects are used, and helps correlate Brackets reliability problems that may be related to specific extensions
* Performance (e.g. startup time, average file switching time, average project size) - helps decide where to focus optimization efforts
* "Wear and Tear" (e.g. number of crashes, average session length) - helps spot reliability problems
* "Meeting user needs" (e.g. how often are features like Live Preview and Inline Editors used?) - tells us if important features are too hard to find, and helps prioritize which features to improve soonest

We'll be sure to announce any changes either on the [blog](http://blog.brackets.io/tag/health-data/) or the [brackets-dev forum](https://groups.google.com/forum/#!forum/brackets-dev).

### Next Up Proposal

* Error & warning counts
    * Total number of uncaught exceptions (exception message / stack trace _not_ captured, to preserve anonymity)
    * Total number of specific error cases, e.g. unable to connect to the built-in Node process or unable to save/load preferences state
    * For each deprecated API, total number of times it's accessed
* "Active time" using Brackets - rough number of minutes Brackets is open and not idle
* "Active time" for each of these file types: HTML, CSS, JS, SASS, SCSS, LESS, PHP, CoffeeScript, Python, Ruby
* Number of times certain specific commands/features have been used:
    * Quick Edit (`navigate.toggleQuickEdit` command)
    * Quick Docs (`navigate.toggleQuickDocs` command)
    * Live Preview (`file.liveFilePreview` command) - separate counts for custom server URL vs. the Brackets built-in server

**Benefits** - With the data above, we can answer further questions like:

* Is it safe to remove some of our deprecated APIs?
* Do we need to raise the priority of certain problems?
* Does the presence of certain extensions correlate with more errors?
* Are people spending a lot of time working with certain file types? Should we add more features specific to those languages?
* How much are some of the more unique, flagship features in Brackets being used? Should we improve their usability, discoverability, or usefulness?

### Further Ahead

* Average/max startup time
* Average/max time opening single files
* Average/max number of files per project
* Average/max time to open Quick Open inline editor
* Average/max time to complete a Find in Files search

**Benefits** - We want to make Brackets perform better.

* How quickly does Brackets perform common operations? Where do we most need to put in effort improving performance?
* Brackets handles projects up to ~10,000 files just fine. How many people are working with projects that Brackets doesn't handle as well today?


# Implementation Notes

[View the source code here](https://github.com/adobe/brackets/tree/master/src/extensions/default/HealthData).

No data is sent for at least 30 minutes the first time Brackets is launched. This ensures that users who want to opt out have time to do so. After that, data is sent every 24 hours when Brackets is running. If Brackets is not running when 24 has elapsed since the last Health Report snapshot, Brackets will send the report the next time it is launched.

Currently, Brackets does not store any data other than the time the next report is due. The data is just a "snapshot" generated at the moment it is sent. In the future, this will change to the Health Report can include totals and averages (see above).