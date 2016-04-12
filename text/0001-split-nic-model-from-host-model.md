- Feature Name: `split-nic-model-from-host-model`
- Start Date: 2014-12-01
- RFC PR: (N/A)
- Issue: [Foreman #2409](http://projects.theforeman.org/issues/2409)

# Summary
[summary]: #summary

The interface (`nic`) object should be made a first-class model and split away
from the `host` model.

# Motivation
[motivation]: #motivation

Currently we cannot properly handle hosts with more than one nic. Foreman
expects the first nic to have special status - all the names, certificates, PXE
files and so forth relate to this nic. Secondary nics can be added but they are
mostly for inventory purposes. In the case of a user with separate provisioning
and production networks, it is currently not possible to set the TFTP/PXE
config to a secondary interface. Likewise, virtual interfaces, bonds and tags
are hard to manage.

The expected outcome is for all interfaces to have equal status within a host.
Any interface should be usable for identifying the host, and for provisoning
the host.

# Detailed design
[design]: #detailed-design

The `nic` model already exists, but it's not used for the primary interface on
the host object. We'll need to start by extracting all the nic-related methods
on the `host` model (ip, domain, subnet, fqdn to name but a few) and delegate
them to the `nic` model.

That will also require databse migrations - `ip` has been a `host` column since
the beginning of the project, and much is built upon it.

Once the nics all have their own model, we can extend the model with new
attribute `primary_flag` and `provisioning_flag`. These can be used in
appropriate places where either the identity or the booting of a machine is
being queried (for example, `fqdn` should be using `host.name` and
`primary_interface.domain`)

The UI will need a significant rewrite as well. With all nics having equal
status, we can probably use a table on the Interfaces tab of the Host>Edit
page, and a modal window for all the required details. Doing it this way allows
us to ask for many more details from the user without cluttering the main edit
page. The table should summarize the current configuration of the nics - as a
minimum, the name, device, ip, and mac should be displayed, as well as icons
for the primary and provisioning interfaces.

There'll be corner-cases around provisioning, particularly in the
virtual-machine provisioning, as we have to know where to SSH to for
image-based deployments. It may also be possible to finally unify virtual
machine nics with Foreman nics on the new interface modal - a long-standing
niggle to many users.

# Drawbacks
[drawbacks]: #drawbacks

The main drawback is risk - this is huge change to the way we store and consume
hardware information in Foreman. Many plugins (e.g. Discovery) are built on the
`host` model, and will need advance warning of these changes. Despite all
tests, I expect this feature to contribute some instability to the project in
the short term, until we get wide enough feedback on it, and a few rounds of
bugfixes.

# Alternatives
[alternatives]: #alternatives

Not doing *something* like this is not really an option. We cannot today
support a fairly common request of correctly handling servers with more than
one NIC - since segmented networks for services like provisioning are common in
larger businesses, this is a blocker for adoption in many places.

No other designs have yet been deeply considered - despite the extensive
changes above, this seemed to be the simplest set of changes required to reach
the goal.

# Unresolved questions
[unresolved]: #unresolved-questions

Fact importers will need updating - we currently import the networking data but
this will need substantial reworking to match on device names and handle
detected changes in data. However, the level to which we should act on this
imported data is unclear - e.g. if we detect a change in IP, should we re-run
any orchestration for that nic?

More complex setups such as handling multiple provisoning flags on bonded nics
have not been detailed. It is likely this will be handled in a later patch once
the initial code is settled, but it could be added here if a suitable design
can be created.
