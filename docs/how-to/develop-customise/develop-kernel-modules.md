---
myst:
  html_meta:
    description: "Develop kernel modules for Ubuntu. Learn how to select and obtain kernel source, build modules the Ubuntu way, and what constraints apply for inclusion in Ubuntu."
---

(how-to-develop-kernel-modules)=

# How to develop kernel modules for Ubuntu

This guide is for developers who write kernel modules intended to run on
Ubuntu Classic, or to eventually land in the Ubuntu kernel.

It assumes you are familiar with Linux kernel development basics, and focuses on
Ubuntu-specific steps: Which source tree to use and where to get it, how to 
build it, and what rules apply if you want your work accepted into Ubuntu.

## Select a kernel source tree

Ubuntu offers many kernels, so it is important to understand which kernel you 
plan to target. Ubuntu kernel sources are organized by release codename and 
package name. Each combination is a kernel series, for example `noble:linux` or
`jammy:linux-azure`.
See {doc}`/reference/ubuntu-kernels` for the full list of kernel variants,
their target releases, and branching strategy.

Pick the series that matches your target:

- The Ubuntu release you want to support, such as Resolute Raccoon 26.04.
- The kernel package for that release. Use the generic `linux` package unless
  you target a specific variant such as `linux-aws` or `linux-oem`.

## Obtain the source

You have two options, depending on your goal:

- Git clone from Launchpad. Best for development and patch submission.
  - Within a series repository, the `master` branch contains the current
  released kernel, `master-next` has the changes staged for the next Stable
  Release Update (SRU). Previous releases are tagged with an `Ubuntu-*` tag.
  See {doc}`/how-to/source-code/obtain-kernel-source-git` for the full
  workflow, including tags, remotes, and reference clones.
- Source package via `apt`. Best for a quick build of the kernel you are
  running. Enable `deb-src` first as described in
  {doc}`/how-to/source-code/enable-source-repositories`, then run
  `apt source linux-image-unsigned-$(uname -r)`.

## Build the Ubuntu way

Build with the Ubuntu kernel packaging so your module matches the production
kernel configuration, signing setup, and ABI.

- To build a full kernel, follow {doc}`/how-to/develop-customise/build-kernel`.
- To iterate on one driver, rebuild only that module against the running
  kernel. Follow {doc}`/how-to/develop-customise/build-kernel-module`.
- To ship a module that survives kernel upgrades, package it with
  {manpage}`dkms(8)`.

```{important}
Locally built kernels and modules are for testing only.
They are not signed for Secure Boot and are not intended for production.
```

## Constraints for landing work in Ubuntu

If you intend your module to be accepted into the Ubuntu kernel, work under
these constraints from the start:

- **Upstream first.** Get your driver into the upstream Linux kernel before
  requesting it in Ubuntu. Ubuntu regularly follows upstream, so your module
  will also land in Ubuntu shortly after. Patches that are not upstreamed 
  (i.e. 'SAUCE' patches) are avoided as much as possible. See
  {doc}`/reference/patch-acceptance-criteria`.
- **Prefer dynamically loading your module.** Ubuntu prefers to 
  dynamically load kernel modules, as opposed to building them in. Ensure 
  that your module behaves well when dynamically loaded.
- **Test thoroughly.** Many kernel module bugs originate from race conditions 
  with dependencies, implicit dependencies on firmware or packages 
  providing them, or not gracefully handling cases where a device is 
  hotplugged, not connected, disconnected, etc.. 
- **Do not modify core kernel code.** Changes outside your own driver
  subsystem are unlikely to be accepted.
- **Do not change existing in-tree drivers** beyond what your patch requires.
  Unrelated changes will be rejected.
- **Do not rely on out-of-tree only interfaces.** Your module must build
  against the unmodified Ubuntu kernel headers for the kernel you target.
- **Follow upstream patch rules.** Run `scripts/checkpatch.pl` before
  submitting. Patches that fail upstream checks are rejected.
- **Follow the Ubuntu patch format.** Use the correct subject prefix,
  `BugLink`, and sign-off tags. See {doc}`/reference/stable-patch-format`.
- **One Launchpad bug per submission.** SRU patchsets require a dedicated bug with
  the SRU justification template. See
  {doc}`/reference/patch-acceptance-criteria`.
- **No new features directly to LTS releases.** New functionality must go
  through the development release or an HWE kernel first.

When your patch is ready, submit it to the Ubuntu kernel team mailing list as
described in {doc}`/how-to/source-code/send-patches`.

## Development and debugging suggestions

These are optional aids, not requirements:

- Use {doc}`/how-to/develop-customise/build-kernel-module` for fast
  build-test cycles on a single driver.
- Use {manpage}`dkms(8)` during development to rebuild your module
  automatically when the kernel changes.
- Test against pre-release kernels to catch breakage early.
  See {doc}`/how-to/testing-verification/test-pre-release-kernels`.

## Related topics

- {doc}`/explanation/ubuntu-linux-kernel-sources`
- {doc}`/explanation/stable-release-updates`
- {doc}`/reference/dkms-upload-rights`
