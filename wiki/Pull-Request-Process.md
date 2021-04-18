If you're submitting a pull request or are reviewing one, hopefully this page will help set expectations for how we see pull requests moving along and landing.

A pull request can be reviewed by any Brackets committer. With this process, we seek to respond to pull requests quickly and to resolve them as quickly as is reasonable. There are two phases to the review: Triage and Review.

The basic process:

1. A contributor submits a new pull request using [Contributor Pull Request Checklist](https://github.com/adobe/brackets/wiki/Pull-Request-Checklist) as a guide.
2. A Brackets committer assigns it to themselves for a quick triage (see Triage below)
3. They do that triage immediately after assigning it to themselves.
4. The person doing triage continues with the back and forth until either the PR is closed or the PR has the right characteristics for a more detailed review.
5. Assuming the PR is continuing, the committer gives the PR a “Triage Complete” label and unassigns it.
6. The same or a different Brackets committer picks up the PR for review and assigns it to themselves.
7. The reviewer starts the review immediately after assigning it to themselves and uses the [Pull Request Review Checklist](https://github.com/adobe/brackets/wiki/Pull-Request-Review-Checklist) as a guide.
8. There’s an expectation from both ends that communication won’t stall for more than a few days (see the Daily Pull Request Report section below)

In general, our goal is that no one should have to wait more than a week or so (barring someone being unavailable for a time and stating so in the pull request comments.)

A pull request can have a "Discussion" label to designate that it is not really intended for landing and not subject to the normal timing constraints.

## Triage

The goal with triage is to quickly skim the code and make sure that it meets some basic requirements for landing before it proceeds.

* Is this something we want in Brackets? (If not, say so and close it or mark “needs review” with a comment)
* Does the code look like core code? (If not, say so and point to coding conventions or comment on the overall layout)
* Does the code have tests? (If not, comment on what you’d like to see in terms of tests)
* Has the contributor signed the CLA?

"Is this something we want in Brackets?" can be a tough question and very much depends on the specific pull request. We have generally preferred for new features to start life as extensions, but that's not always possible and some things definitely do belong in core. If we see patterns of things we don't want to see in Brackets core, we'll add them here.

## Translations

Generally speaking, translations should be quick to triage. Our review process for translations is generally simple: find another person in the Brackets community that speaks the language and can verify that the translation is a good one.

## Daily Pull Request Report

To glue this process together, we'll have a [daily report](http://prksingh.github.io/pr-tracking/) emailed to the Brackets committers that highlights overdue pull requests:

1. PRs that have not started triage in 8 days
2. PRs that have not started review within 8 days of Triage Complete
3. PRs that have been waiting for a response from the assignee for 8 days
4. PRs that have been waiting for a response from the contributor for 8 days

In addition to overdue PRs, the report will also list "available PRs" which are requests are not assigned.