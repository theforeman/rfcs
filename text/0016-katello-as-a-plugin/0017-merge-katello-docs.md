- Feature Name: Merge Katello documentation to theforeman.org
- Start Date: 2016-10-11
- RFC PR: (leave this empty)
- Issue: (leave this empty)

# Summary
[summary]: #summary

Today the Foreman website and plugins are kept in a different repository than Katello. Further, the websites and documentation are at different URLs http://www.theforeman.org and http://www.katello.org.

# Motivation
[motivation]: #motivation

The separation of documentation serves to push users into feeling there is a difference between Foreman and Katello. And while there is some technical differences currently, the goal is to bring everything in to alignment. One way to do this is by having a central place to go for documentation on all things Foreman and plugins. The expectation is that users will go to theforeman.org and be able to find Katello plugin documentation as you would any other plugins.

# Detailed design
[design]: #detailed-design

Here are the set of action items for achieving this:

 * Set a redirect from katello.org to theforeman.org Katello plugin page
 * Carry forward Katello 2.4+ documentation versions
 * Add a Troubleshooting guide to theforeman.org and include info from katello.org
 * Remove katello.org Media section
 * Add note to Foreman security to mention plugin in report
 * Drop katello.org community section
  - keep the scripts section or make a community tools repository
 * Drop Bastion section from katello.org
 * Merge style guides section with theforeman.org style guides
 * Move katello.org Developer guide to theforeman.org as a top level developer guide
 * Add a multi-issue example to commit message guide

# Drawbacks
[drawbacks]: #drawbacks

This will generate some churn for users and an overhaul of the documentation layouts potentially. There is a possibility with current technical requirements around installing Katello to leave users confused.

# Alternatives
[alternatives]: #alternatives

N/A

# Unresolved questions
[unresolved]: #unresolved-questions

N/A
