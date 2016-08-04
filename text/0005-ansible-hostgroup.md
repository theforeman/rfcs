- Feature Name: Allow roles assignment to hostgroup
- Start Date: 2016-08-04
- RFC PR: https://github.com/theforeman/foreman_ansible/pull/30
- Issue: http://projects.theforeman.org/issues/15837

# Summary
[summary]: #summary

In a similar way foreman_ansible allows assigning roles to hosts we should let users assign roles to host groups. Functionally we want to provide user experience similar to puppet classes in host and hostgroups. One of decisions to be made in this RFC is whether the Foreman should allow ordering of Ansible roles for hosts and hostgroups.

# Alternatives
[alternatives]: #alternatives

## Host groups

Not much alternatives here. We need to implement M:N association between hostgroups and roles and allow for standard hostgroup inheritance.

## Config groups

The Foreman uses config groups for bundling multiple puppet classes together. We could use them for Ansible roles too.

Alternatives of the mapping:

  1. **1:N** - The same approach as with puppet classes. One config group can have multiple roles.
  1. **1:1** - A role itself is a bundle of Ansible modules which makes it very similar to config group. Therefore it might make more sense to model it as one config group. One of implementations could be using config groups for modeling roles directly (probably via STI).
  1. **no config groups** - Don't use config groups for roles at all. Roles have similar purpose as config groups, but there's nothing to edit on them in Foreman (we only import them). Therefore we can keep the both models to live side by side.

## Role ordering

Alternatives for ordering the roles:
  1. Provide just some default predictable order without allowing users to change it.
  1. Allow ordering in both hostgroups and hosts, order from hostgroup serves as default order in the host form, but it's still editable there.
  1. No-order in hostgroups, allow final re-ordering only in host edit form.


# Proposed design
[design]: #proposed-design

Since the usecases aren't 100% clear at the moment, we should stick with the simplest design and let evolution to do it's job. None of the simplest alternatives puts obstacles into future migration to a more complicated model.

**New models**

Existing `AnsibleRole` model will be connected to `Hostgroup` via simple M:N associating model `HostgroupAnsibleRole`.

**Config groups**

We decided not to use config groups at all. The concept of config groups from the Foreman seem very similar to roles in Ansible world. Therefore we took only alternatives 2 and 3 into account. The difference between those two is rather on the implementation level and users shouldn't recognize it. From that perspective it's simpler to use existing `AnsibleRole` and don't add complexity when it's currently not needed.

**Ordering**

Foreman will do no ordering of Ansible roles. For the means of UI roles in a hostgroup will be ordered by their IDs (~ order of import). Sub-hostgroups and hosts will have inherited roles first and then alphabetically ordered additional roles. This ordering will have no effect on the final execution of Ansible roles.

This decision has been made based on the fact that roles can be dependent on other roles and Foreman can't easily reflect this relationship in the ordering. The problem of resolving the dependencies is solved in Ansible and Foreman shouldn't create additional layer. If users need explicit ordering, it's recommended Ansible practice to create an encapsulating role that specifies the order via dependencies.
