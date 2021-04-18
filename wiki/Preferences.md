_**This page is no longer up to date.** The Preferences feature shipped in Brackets 0.36._

**See [Preferences System documentation](https://github.com/adobe/brackets/wiki/Preferences-System) for current information.**

<br><br>

----

## Preferences Model ##

Preferences are often considered as a failure to do good design and that making the right choices for users is a better way to go. There are circumstances in which prefs can [even be harmful](http://limi.net/checkboxes-that-kill/). For a programmer's editor, however, a good preferences system is important.

* Developers have different workflows
* Different projects have different coding standards
* Different languages have different requirements
* Certain features are more important for some languages than for others
* It's also possible that a single file may not conform to the usual rules (perhaps the filename is ".txt", but it should actually be treated as a Markdown file)

I think we need a preferences system that makes the common preferences easy and understandable to configure, but also makes less common configuration possible.

## An Example ##

I'll use an example that is simple but touches on the complexity of the problem: indentation style for new files. There are two related prefs here: tabs vs. spaces and the width of a single indentation stop. We'll look just at tabs/spaces.

* Brackets will have some default value (say "spaces")
* The user may want to set a global default to "tabs" for when they're starting something new
* A given project edited by that user may have standardized on "spaces" for JavaScript files

Indentation style is not the only pref that is like this. Most preferences won't flip back and forth like this from one context to another, but given the variability of developer workflows, there's no way to know in advance which preferences might vary from one to another.

Indentation style can be detected for an existing file. Other preferences, however, may even need to be overridden at the file level.

Ideally, a part of Brackets that needs to know the value of a pref should be able to just ask for it and get back the correct value in context.

## User Interface ##

A user interface to handle the kind of configuration in the Example could get complicated. Sublime Text punts on a UI altogether, instead just providing an empty editor window into which the user types JSON config information.

Firefox and Chrome provide a good model here, as they both have *many* options. They provide good, usable UI for the prefs that people will commonly want to tweak. They also provide a generic pref editing UI that can change *any* of the options available in the system.

In the case of indentation style, for example:

* A global preferences pane can ask the user for their default value
* A project preferences pane would default to the global pref, but allow the user to change the default for that project
* It could further allow customization for .js files in the project
* The status bar display allows the user to change the setting for the current file

Obviously, there is UX work required here, but there is a lot of prior art. We should have a generic editor that allows the changing of prefs at all levels.

## Extensions ##

Extensions should be able to make settings available and should also be able to provide UI for their important settings. We should encourage people to only make the most important settings visible in the UI, but to feel free to add other hidden settings.

## One More Use for Hidden Settings ##

If there's a feature that's in development and not quite ready to show to the world, it can be hidden behind a setting. Chrome and Firefox do this often.

## Data Model ##

A minor implementation note: JavaScript's object model [happens to be perfect for this kind of prefs system](http://steve-yegge.blogspot.com/2008/10/universal-design-pattern.html). Because JavaScript's object model is based on objects with prototypes, rather than classes, you can build up a prototype chain that is something like this (starting from the most specific with each successive line the prototype for the last):

* file
* project/file type
* global/file type
* global
* default

If you look at `prefs.indentationStyle` where "prefs" is the file object, each of objects in the chain will be traversed to find the closest one that has a specific value set.             
[pt] Would it also make sense to have customer specific preferences? (that is a collection of projects)

## Data Storage ##

Currently, we put preferences data into localStorage. This *may* be okay for global preferences, but it's not okay for project preferences. Project and file preferences should be put in a file with the project that can be checked into version control, allowing everyone on the team to share the settings associated with the project.

[dk] We might want to be able to do both: per-project settings shared with others via the repository and per-project settings just for me. For instance, a base URL is probably not universal enough to share with all developers as every developer my use a different test server setup.

## Preferences vs. "View State" ##

In Brackets today, much of what we save via our "preferences" system could actually be called something like "view state". Things like "sidebar width", "expanded folders in the project tree", "working set documents" would fall into this category.

An important difference between view state and preferences is that you wouldn't put view state into version control. Project-level view state would need to have a different storage backend from project-level preferences.

[nj] I wonder if caches are a third thing that could fit into this model. As an example, you could imagine that code hint info caching worked this way: JS builtins would go into the global cache, info for non-local libraries (e.g. jQuery from a CDN) would go into a per-project cache, and per-file info would go into a per-file cache. Like view state, you wouldn't want to check this into source control.

[nj] Also, while view state might never be checked into source control, you might still want it to be sharable in some easy fashion, so you can carry your settings with you from computer to computer.

## Conclusion ##

Not all prefs *need* the ability to be overridden at every level. Some, in fact, only make sense at the global level. But, we need to have a model in mind that can handle the cases that users do need to be able to customize and, in the case of some project preferences, share with others via version control.