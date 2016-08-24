- Feature Name: UEFI support for PXE booting
- Start Date: 2016-05-04
- RFC PR: [#2](https://github.com/theforeman/rfcs/pull/2)
- Issue: [Redmine #431](http://projects.theforeman.org/issues/431)

# Summary
[summary]: #summary

The goal of this feature is to implement PXE booting of UEFI systems. In the default configuration on at least RHEL6, RHEL7 and Debian stable platforms, both BIOS and UEFI systems should be able to PXE boot including Discovery Image. Foreman Bootdisk UEFI support is out of scope of this task.

# Owners

Lukáš Zapletal (Foreman) - `lzap_at_redhat_dot_com`

# Motivation
[motivation]: #motivation

Currently, Foreman supports booting of x86 BIOS systems via PXELinux and several other platforms via Grub1, ZTP and POAP. These are represented as PXE "variants": syslinux, pxegrub, ztp and poap which are hardcoded in operating system model classes. However, UEFI is not currently supported, and should be.

# Detailed design
[design]: #detailed-design

Initial testing was done to get UEFI working on PXELinux and it looks like stable version of Syslinux (= PXELinux), which is currently 6.02, does not appear to support PXE booting correctly (at least not with TianoCore UEFI ROM in QEMU/KVM). The only version which was found working was 6.03-pre20, but there is no stable release yet with UEFI support.

Additionally, there is no PXELinux version with UEFI support in RHEL6 and RHEL7 and both systems ship with Grub (versions 1 and 2 respectively). Grub is more mature when it comes to UEFI support and since it can boot via PXE and Foreman already supports Grub1, it is more feasible to get UEFI working via officially supported Grub. To have complete support matrix, PXELinux will be part of this implementation, but hardware support (and thus testing) will be limited.

Both Grub 1 and 2 have MAC-based config file retrieval built-in and both have different configuration syntax, therefore new PXE kind (Grub2) must be added. Currently supported PXE combinations are:

Variant  | Template kind | Arch | DHCP filename | MAC config | Default config
--- | --- | --- | --- | --- | ---
SYSLINUX | PXELinux | BIOS | pxelinux.0 | pxelinux.cfg/01-aa-bb-cc-dd-ee-ff | pxelinux.cfg/default
PXEGRUB | PXEGrub | BIOS | Solaris-* | boot/grub/menu.lst.01aabbccddeeff | boot/grub/menu.lst
POAP | POAP | - | poap.cfg/aabbccddeeff | poap.cfg/AABBCCDDEEFF | -
ZTP | ZTP | - | poap.cfg/aabbccddeeff | poap.cfg/AABBCCDDEEFF | -

PXELinux covers all x86 systems, PXEGrub is used for Solaris (firmware filename is Solaris-*-pxegrub) and POAP/ZTP have built-in loaders and only downloads configuration files. This feature aims for this support matrix:

Variant | Template kind | Arch | DHCP filename(s) | MAC config | Default config | Comment
--- | --- | --- | --- | --- | --- | ---
SYSLINUX | PXELinux | BIOS | pxelinux.0 | pxelinux.cfg/01-aa-bb-cc-dd-ee-ff | pxelinux.cfg/default | No changes
SYSLINUX | PXELinux | UEFI | bootia32.efi, bootx64.efi | pxelinux.cfg/01-aa-bb-cc-dd-ee-ff | pxelinux.cfg/default | Configuration compatible
PXEGRUB | PXEGrub | BIOS | pxegrub | grub/menu.lst.01aabbccddeeff | grub/menu.lst | PXELinux will be used instead usually, but default filename will be set for generic OS.
PXEGRUB | PXEGrub | UEFI |  grub/bootia32.efi, grub/bootx64.efi | grub/01-AA-BB-CC-DD-EE-FF | grub/efidefault | New
PXEGRUB2 |  PXEGrub | BIOS | pxegrub2 | grub2/grub.cfg-01-aa-bb-cc-dd-ee-ff | grub2/grub.cfg | PXELinux will be used instead usually, but default filename will be set for generic OS.
PXEGRUB2 |  PXEGrub | UEFI |  grub2/grubx32.efi, grub2/grubx64.efi | grub2/grub.cfg-01-aa-bb-cc-dd-ee-ff |  grub2/grub.cfg |  New
POAP | POAP | - | poap.cfg/aabbccddeeff | poap.cfg/AABBCCDDEEFF | - | No changes
ZTP | ZTP | - | poap.cfg/aabbccddeeff | poap.cfg/AABBCCDDEEFF | - | No changes

For UEFI systems, 32bit or 64bit UEFI loader will be configured on DHCP according host's architecture association. Most Linux distributions have limited or zero support of 32bit UEFI (loaders are simply not shipped in their repositories, UEFI specifications also assume systems are 64bit), but to give the possibility of booting native 32bit UEFI loaders filenames "bootia32.efi" and "grubx32.efi" will be handed over by DHCP in order to support that path.

In secure boot environments, loaders must be either signed or an alternative shim loader can be used. Users will be able to select "Secure Boot UEFI" options which will cause setting DHCP filename to either "grub/shim.efi" or "grub2/shim.efi" depending on variant which will then pass the check and chain-load unsigned firmware named "bootx64.efi" or "grubx64.efi" from the same directory.

PXEGrub (version 1) configuration path is changing from boot/grub/ to grub/ for consistency, a change in Foreman core will be needed but no migration or manual change after upgrade is needed - existing hosts in boot mode will continue booting from the old dir, new ones will use the new path. TFTP file deletion does not trigger error, thus the only drawback is there might be configuration files left after some time. We will put this into upgrade instructions.

## Implementation

Both Foreman Core and Smart Proxy changes are required in order to support all of the above.

### [Foreman proxy #12633](http://projects.theforeman.org/issues/12633) - Add Pxegrub2 variant and allow multiple TFTP config files

Foreman Proxy must be enhanced with new PXE variant "Pxegrub2" to allow deploying configuration files in "grub2/" folder. In addition to that, Pxegrub (version 1) must deploy two configuration files instead of one now to support both BIOS and UEFI naming conventions (grub/menu.lst vs grub/efidefault and grub/menu.lst.01aabbccddeeff vs grub/01-AA-BB-CC-DD-EE-FF). Also, Pxegrub variant will use new base directory "grub/" instead of "boot/grub/" for consistency with "grub2" and "pxelinux.cfg/".

### [Foreman proxy #15864](http://projects.theforeman.org/issues/15864) - Pxelinux kind as an alias for Syslinux

We are removing template variant attribute from OS in Foreman Core and template kind will define what to pass to Proxy. Since Syslinux variant is implemented as PXELinux kind and PXELinux = SYSLinux (it's the same just different name), I will submit a patch to accept both.

### [Foreman core #12634](http://projects.theforeman.org/issues/12634) - New HW Model flags pxe_template and loader

Foreman will support both BIOS and UEFI systems and it is required to associate extra information for each indidivual host (or hostgroup) in order to provide TFTP "filename" option correctly. It can be set per instance via new Host/Hostgroup string flag (free-form dropdown field) called "PXE Loader" (`pxe_loader` model attribute) with possible values:

* "PXELinux BIOS" => pxelinux.0 (default value)
* "PXELinux UEFI" => pxelinux.efi
* "Grub UEFI 64bit" => grub/bootx64.efi
* "Grub UEFI 32bit" => grub/bootia32.efi
* "Grub SecureBoot" => grub/shim.efi
* "Grub2 UEFI 64bit" => grub2/bootx64.efi
* "Grub2 UEFI 32bit" => grub2/bootia32.efi
* "Grub2 SecureBoot" => grub2/shim.efi
* "None" => Skip TFTP orchestration (useful for example with iPXE provisioning with libvirt)

Model validation does not allow arbitrary strings for the field.

OS template_kind flag will be refactored to a list of possible compatible kinds and TFTP orchestration will always render all compatible kinds which were associated with the OS (or throw an error if there is no template associated for the selected PXE loader type).

Orchestration TFTP code must be modified to use multiple "KIND default local boot" and "KIND global default" templates. Local and global boot templates must be added for Grub2 new kind. Since the proxy has been already modified to deploy both UEFI and BIOS templates, the template is always sent once (and rendered twice for PXEGrub).

Template variant will be completely removed from the codebase because PXE template implies variant to use and there are no variants which share the same configuration syntax.

New templates will be created as part of the PR (and submitted into [community-templates repo](https://github.com/theforeman/community-templates/pull/291) as well)

* PXEGrub2 default local boot
* PXEGrub global default
* PXEGrub2 global default
* Kickstart default PXEGrub
* Kickstart default PXEGrub2
* pxegrub_discovery
* pxegrub2_discovery

### [Foreman installer #14920](http://projects.theforeman.org/issues/14920) - Change dhcpd.conf to support UEFI loaders

Default DHCPD configuration must be changed in order to allow booting unknown UEFI hosts as it was shown above (dhcpd.conf snippet). Where appropriate (RHEL) shim.efi should be used. Expected configuration:

```
option space PXE;
option arch code 93 = unsigned integer 16; # RFC4578

subnet 10.0.0.0 netmask 255.255.255.0 {
    option routers 10.0.0.254;
    range 10.0.0.2 10.0.0.253;

    class "pxeclients" {
        match if substring (option vendor-class-identifier, 0, 9) = "PXEClient";
        next-server 10.0.0.1;
        if option arch = 00:06 {
            filename "grub2/bootia32.efi";
        } else if option arch = 00:07 {
            filename "grub2/bootx64.efi";
        } else {
            filename "pxelinux.0";
        }
    }

    host example-ia32 {
        hardware ethernet XX:YY:ZZ:11:22:33;
        fixed-address 10.0.0.2;
    }
}
```

### [Foreman installer #12635](http://projects.theforeman.org/issues/12635) - Options to deploy Grub and PXELinux UEFI loaders in TFTP root

It looks like Grub2 is the best and most compatible way of PXE booting UEFI hardware and it is available in most Linux distributions. It makes sense to provide support in our Puppet classes and installer to deploy these. Instructions on where to find the firmware in RHEL7 are at [access.redhat.com](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/chap-installation-server-setup.html) (RHEL6 does not ship Grub2 so installer should fail or skip that). Similarly for Debian and Ubuntu it is available for [direct download](http://archive.ubuntu.com/ubuntu/dists/trusty/main/uefi/grub2-amd64/current/) or in debian-installer-8-netboot-amd64 package.

UEFI shim shall be installed if possible. Expected file paths are defined in the table above.

### [Foreman core #14921](http://projects.theforeman.org/issues/14921) - ComputeResource should pass host loader flag to VM

Since host will have "loader" (BIOS/UEFI/SecureBoot UEFI) flag, compute resources could pass this information when creating new VMs so they gets created with correct firmware and PXE booting will just work.

### [Foreman discovery #14922](http://projects.theforeman.org/issues/14922) - Discovery can detect BIOS/UEFI and report it back

Discovery can detect current OS loader via a [custom fact](https://github.com/jcpunk/puppet-efi/blob/master/lib/facter/efi.rb) and this can be used for provisioning to set the "loader" host attribute. Detection of secure boot should be possible too.

# Other ideas
[otherideas]: #other-ideas

* UEFI for ARM/ARM64 platform is emerging and it looks like Grub2 will support it - we should test this
* PPC/PPC64 uses yaboot or petitboot and it should be relatively easy to add support for this but we need someone to test this
* IA64 uses eLilo - ditto
* z360 can be booted via zPXE and Cobbler supports that - can we find someone from the community to help with testing that?

# Unresolved questions
[unresolved]: #unresolved-questions

* None.

# References
[references]: #references

* http://www.rodsbooks.com/efi-bootloaders/secureboot.html - about SecureBoot and loaders
* https://access.redhat.com/blogs/1169563/posts/1421543 - workaround via foreman_hooks
* https://beaker-project.org/docs/admin-guide/tftp.html - other ideas on PXE booting
* http://jk.ozlabs.org/blog/post/158/netbooting-petitboot/?cm_mc_uid=91489629740814486170120&cm_mc_sid_50200000=1448888937 - PPC
* http://lukas.zapletalovi.com/2014/09/efi-in-qemu-kvm-on-fedora-20.html - setup KVM/QEMU in Fedora for UEFI
