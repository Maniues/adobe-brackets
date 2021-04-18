## History of Brackets stats

[This public Google spreadsheet](https://docs.google.com/spreadsheets/d/1xCQh0Ii-MaVcqn-MdF-zDFy3quDBhEHZPeiOEmmITgM/edit?usp=sharing) includes stats starting from Brackets's very first public release all the way to the present. Be sure to check the tabs at bottom for graph views.

## Stats for the release about to ship

#### Commits & files

* **Number of commits** - use a link of the form https://github.com/adobe/brackets/compare/release-0.XX...release-0.YY (use just `release` as the endpoint if we haven't tagged the build yet). Then do the same for brackets-shell and sum together the number of commits.
* **Number of files touched** - same as above

#### Pull requests

1. Install Peter's "Brackets Reports" extension / make sure you have the latest version
2. Find the day the _previous_ version branched to `release` (usually just before the time the _current_ release's "set-release number" PR landed).
3. Run _View > Brackets Reports > Pull Requests in Release_
4. Enter the date from step 2 as "Earliest commit date" (following the date format shown). Leave "Latest commit date" blank (unless you're generating a report for an older release). Hit Ok.
    * It may take a few seconds before showing the results
5. Remove all "R" entries near the top of the list (PRs that actually went into the _previous_ release, merged into its release branch late in the game)
6. Remove all "M" entries near the bottom of the list (PRs that are actually in a _future_ release, merged into master after the current release branched)
7. Generally speaking, remove any "?" entries as well (branch-to-branch merges)
8. _Update the totals at the bottom_ to reflect these deletions

We usually look at the following stats:

* **Number of pull requests merged (total)** (_doesn't_ count PRs that were closed without being merged)
* **Number of external pull requests merged**
* **Number that came from external committers**
* **Total number of external contributors this release**

#### Locales

To see if there were any **new locales**, compare the set of folders under `src/nls` in this sprint vs. the previous sprint. You can also `git difftool release-0.XX...release-0.YY -- src/nls` to gather a list of **updated locales**. (Or just look at the "Community contributions to Brackets" list in the Release Notes).

#### Lines of JS Code

Use the [Lines of Code Counter for JavaScript](https://github.com/peterflynn/simple-sloc-counter) extension, with these exclusions:

```
/thirdparty/
/3rdparty/
/node_modules/
/bootstrap-
/test/smokes/
/test/lib/
-test-files/
-test-files-
-known-goods/
-perf-files/
/unittest-files/
/testfiles/
```

#### Bugs

We normally don't report the number of issues fixed, because (a) it's hard to distinguish legacy bugs vs. bugs that were both opened _and_ closed within the one sprint's lifecycle, and (b) we often forget to tag every bug with a sprint milestone, leading to varying degrees of undercount.

But you can get a _rough_ number via a "closed issues" milestone query on GitHub -- something of the form https://github.com/adobe/brackets/issues?q=is%3Aclosed+milestone%3A%22Release+0.44%22.


## Stats for the release about to be obsoleted

#### Downloads

1. Go to the brackets.io analytics page
2. Drill down to Behavior > Events > Overview > Event Action and select the desired release number
3. In the "Secondary dimension" dropdown, choose Users > Operating System

> Note: if you select Behavior > Events > Top Events instead, you must separately select each of the Downloads, E4BDownloads, and Other-Downloads items and sum their values to get the total number of downloads

* **Total downloads** - sum Mac + Linux + Windows (ignore the tiny number of downloads on misc other OSes)
* **Platform breakdown (%)** - divide each platform by the total
    * Note: this may understate Linux users, since we know there are some alternative distribution channels that don't hit our download site

> **Warning** - If a sprint lasts longer than 30 days, you'll need to change the date range filter in the upper right to ensure _all_ the download events are included in the total (GA by default excludes anything older than 30 days ago).

#### Extensions - current

1. _View > Brackets Reports > Save Extension Registry Snapshot_
2. Name the file something like "registry - end of Sprint XX lifespan.json" (where XX is the _old_ sprint that's about to be EOL'ed)
3. _View > Brackets Reports > Extension Registry Report_ and select the file you just saved

This gives you:

* **Total available** - listed at top
* **Total number of authors** - sum the internal & external totals
* **Number of external authors**

#### Extensions - changes over time

To compare with previous sprints, you'll need a similar JSON file saved from an earlier sprint _(Peter has an archive of these if needed)_.

1. _View > Brackets Reports > Extension Registry Diff_
2. Choose the older JSON file first (e.g. if we're about to ship Sprint 35, choose Sprint _33_'s file)
3. Choose the JSON file you've just generated (e.g. if we're about to ship Sprint 35, choose Sprint _34_'s file)

Scroll to the bottom of the report for these totals:

* **Number of new extensions added**
* **Number of existing extensions updated**
* **Number of existing extensions deleted** (this is normally zero since there's no easy way to delete deprecated extensions)


## Other Stats

* **[GitHub traffic report](https://github.com/adobe/brackets/graphs/traffic)**
* **[Top-starred repos](https://github.com/search?l=&q=stars%3A%3E10000&type=Repositories)** (Brackets is #13 as of November 2014)
* **[Top-starred JS repos](https://github.com/search?l=JavaScript&q=stars%3A%3E10000&type=Repositories)** (Brackets is #8 as of November 2014)