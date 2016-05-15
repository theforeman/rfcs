# Foreman RFCs
[Foreman RFCs]: #foreman-rfcs

Many changes, including bug fixes and documentation improvements can be
implemented and reviewed via the normal GitHub pull request workflow.

Some changes though are "substantial", and we ask that these be put through a
bit of a design process and produce a consensus among the long-standing Foreman
developers.

The "RFC" (request for comments) process is intended to provide a consistent
and controlled path for new features to enter the project.

##Active RFC List
[Active RFC List]: #active-rfc-list
* [UEFI support for PXE booting](https://github.com/theforeman/rfcs/pull/2)
* [Update asset management](https://github.com/theforeman/rfcs/pull/3)

## Table of Contents
[Table of Contents]: #table-of-contents
* [Opening](#foreman-rfcs)
* [Active RFC List](#active-rfc-list)
* [Table of Contents](#table-of-contents)
* [When you need to follow this process](#when-you-need-to-follow-this-process)
* [Gathering feedback before submitting](#gathering-feedback-before-submitting)
* [What the process is](#what-the-process-is)
* [The role of the mentor](#the-role-of-the-mentor)
* [The RFC life-cycle](#the-rfc-life-cycle)
* [Reviewing RFCs](#reviewing-rfcs)
* [Implementing an RFC](#implementing-an-rfc)
* [Isn't this a bit vague?](#isnt-this-a-bit-vague)

## When you need to follow this process
[When you need to follow this process]: #when-you-need-to-follow-this-process

You need to follow this process if you intend to make "substantial" changes to
Foreman, the Smart Proxy, the Foreman Installer, its documentation, or any
[plugins][plugins] which have chosen to use this process. What constitutes a
"substantial" change is evolving based on community norms, but may include the
following:

* A new feature that creates new UI pages or substantially alters the existing UI
* A new feature that creates new or backward-incompatible API endpoints
* The removal of features that already shipped as part of a previous release

Some changes do not require an RFC:

* Rephrasing, reorganizing or refactoring
* Addition or removal of warnings
* Simple bugfixes
* Additions only likely to be _noticed by_ other developers-of-foreman, not
  users-of-foreman

If you submit a pull request to implement a new feature without going through
the RFC process, it may be closed with a polite request to submit an RFC first.

For plugins, consult the revelant plugin repo to learn about their processes.

## Gathering feedback before submitting
[Gathering feedback before submitting]: #gathering-feedback-before-submitting

A hastily-proposed RFC can hurt its chances of acceptance. Low quality
proposals, proposals for previously-rejected features, or those that don't fit
into the near-term roadmap, may be quickly rejected, which can be demotivating
for the unprepared contributor. Laying some groundwork ahead of the RFC can
make the process smoother.

Although there is no single way to prepare for submitting an RFC, it is
generally a good idea to pursue feedback from other project developers
beforehand, to ascertain that the RFC may be desirable - having a consistent
impact on the project requires concerted effort toward consensus-building.

The most common preparations for writing and submitting an RFC include talking
the idea over on #theforeman-dev, filing and discusssing ideas on the [RFC
issue tracker][issues], and occasionally posting 'pre-RFCs' on [the dev mailing
list][discuss] for early review. Gaining a mentor for your RFC from within
long-standing developers is invaluable to this process.

As a rule of thumb, receiving encouraging feedback from long-standing project
developers, and particularly ones who know that area of the project well, is a
good indication that the RFC is worth pursuing.

## What the process is
[What the process is]: #what-the-process-is

In short, to get a major feature added to Foreman, one must first get the RFC
merged into the RFC repo as a markdown file. At that point the RFC is 'active'
and may be implemented with the goal of eventual inclusion into Foreman.

* Fork the RFC repo http://github.com/theforeman/rfcs.
* Copy `0000-template.md` to `text/0000-my-feature.md` (where 'my-feature' is
  descriptive. don't assign an RFC number yet).
* Fill in the RFC. Put care into the details: **RFCs that do not present
  convincing motivation, demonstrate understanding of the impact of the design,
  or are disingenuous about the drawbacks or alternatives tend to be poorly
  received**.
* Submit a pull request. As a pull request the RFC will receive design feedback
  from the larger community, and the author should be prepared to revise it in
  response.
* Build consensus and integrate feedback. RFCs that have broad support are much
  more likely to make progress than those that don't receive any comments. In
  particular, if an RFC has a mentor, they should help to promote discussion
  and consensus-building.
  * The mentor may schedule meetings with the author/other stakeholders to
    discuss in greater detail.
  * Discussion should happen in the PR comments as much as possible. Meetings &
    offline discussion should be summarised in the PR comments.
  * RFCs rarely go through this process unchanged, especially as alternatives
    and drawbacks are shown. You can make edits, big and small, to the RFC to
    clarify or change the design, but make changes as new commits to the PR and
    leave a comment detailing the changes. Specifically, **do not squash or
    rebase commits after they are visible on the PR**.
* Once the RFC has been clarified and defended and the conversation is settled,
  the RFC will be merged by one of the long-standing developers, or rejected by
  closing the pull request. Optionally, a one-week "final comments" period may
  be used for late contributions to the discussion.

## The role of the mentor
[The role of the mentor]: #the-role-of-the-mentor

It is recommended (but not essential) to find a mentor from amongst the
long-standing developers (RFC authors who are already long-standing developers
may, of course, be their own mentor). The role of the mentor is to help the RFC
move along through the process. This would roughly entail:

* Reading the RFC in detail and providing the initial feedback
* Soliciting feedback from those who are likely to have strong opinions
* If required, scheduling meetings (IRC or Hangouts) to resolve complex issues
* If requried, announcing the "final comments" period

The aim is for the mentor to help gather as much feedback as possible before a
decision is reached - by the end of the process, the decision would hopefully
be obvious and not require a final comments period.

## The RFC life-cycle
[The RFC life-cycle]: #the-rfc-life-cycle

Once an RFC becomes active then authors may implement it and submit the feature
as a pull request to the appropriate Foreman repo(s). Becoming 'active' is not
a free pass, and in particular still does not mean the feature will ultimately
be merged; it does mean that the project has agreed to it in principle and is
open to merging it.

Furthermore, the fact that a given RFC has been accepted and is 'active'
implies nothing about what priority is assigned to its implementation, nor
whether anybody is currently working on it. While it is not *necessary* that
the author of the RFC also write the implementation, it is by far the most
effective way to see an RFC through to completion: authors should not expect
that other project developers will take on responsibility for implementing
their accepted RFC.

Modifications to active RFC's can be done in followup PR's. We strive to write
each RFC in a manner that it will reflect the final design of the feature; but
the nature of the process means that we cannot expect every merged RFC to
actually reflect what the end result will be at the time of the next major
release.

## Reviewing RFCs
[Reviewing RFCs]: #reviewing-rfcs

While the RFC PR is up, the mentor may schedule meetings with the author and/or
interested parties to discuss the issues in greater detail, and in some cases
the topic may be discussed at other meetings. In any such case, a summary from
the meeting will be posted back to the RFC pull request.

A decision is made by the long-standing developers on an RFC once the benefits
and drawbacks are well understood. These decisions can be made at any time,
there is no set period for an RFCs PR to be open. When a decision is made, the
RFC PR will either be merged (and assigned a number), or closed. In either
case, if the reasoning is not clear from the discussion in the comment thread,
a comment describing the rationale for the decision will be added.

If accepted, an issue will be created in the appropriate issue tracker, with a
link to the RFC document. The RFC should also be updated with a link to the
issue. This allows any developer to see if there is any activity on the RFC.

## Implementing an RFC
[Implementing an RFC]: #implementing-an-rfc

The author of an RFC is not obligated to implement it. Of course, the RFC
author (like any other developer) is welcome to post an implementation for
review after the RFC has been accepted.

If you are interested in working on the implementation for an 'active' RFC, but
cannot determine if someone else is already working on it, feel free to ask
(e.g. by leaving a comment on the associated issue).

### Isn't this a bit vague?
[Isn't this a bit vague?]: #isnt-this-a-bit-vague

The process is intended to be as lightweight as reasonable for the present
circumstances. As usual, we are trying to let the process be driven by
consensus and community norms, not impose more structure than necessary.

**Foreman's RFC process is inspired by the [Ember RFC] and [Rust RFC] processes**

<!-- link ids -->
[plugins]: http://projects.theforeman.org/projects/foreman/wiki/List_of_Plugins
[issues]: http://projects.theforeman.org/projects/rfcs/issues
[discuss]: https://groups.google.com/forum/#!forum/foreman-dev
[Ember RFC]: https://github.com/emberjs/rfcs/
[Rust RFC]: https://github.com/rust-lang/rfcs
