- Feature Name: Katello as a Plugin
- Start Date: 2016-10-11
- RFC PR: (leave this empty)
- Issue: (leave this empty)

# Summary
[summary]: #summary

The goal of this effort is to allow Katello to be treated from a community and technical perspective as any other plugin.

# Motivation
[motivation]: #motivation

Katello was originally a stand alone Rails application that attempted to integrate with Foreman through a single-sign-on process, Rails engines that knew how to communicate with one another and switching menu structures. After a desire to centralize on a single community and bring integration tighter, Katello was moved to be a plugin but given the constraints on Katello for depoyment along with timeline the deployment of Foreman and Katello was made unique when compared to any other plugin.

# Detailed design
[design]: #detailed-design

The design is being broken out into mini-RFCs due to the breadth of each one and this is being used as a tracker to link to them since there is the small efforts and designs along with the broader goal. This section should be updated with links to the individual mini-RFCs as they are opened.

# Drawbacks
[drawbacks]: #drawbacks

The grand drawback with this work would be that it would interrupt business as usual and no changes or work is needed that would disrupt the release flow of Katello.

# Alternatives
[alternatives]: #alternatives

# Unresolved questions
[unresolved]: #unresolved-questions
