---
title: "Bring Your Own Image"
date: 2020-07-20
tags: ["linux", "ovh"]
draft: false
---

While working at **OVHCloud**, I developed a feature called **BYOI** (Bring Your Own Image).

**Bring Your Own Image** technology lets you boot any cloud (or non-cloud) image on bare metal.

Not every vendor ships images that are fully agnostic yet (by that I mean ready for both **UEFI** and **legacy boot**), but with a bit of tinkering (**packer**), we can get something functional.

This marks a first step towards **Hybrid IAC**.

[The documentation is here](https://docs.ovh.com/gb/en/dedicated/bringyourownimage/)

**What does it actually do?**

- API deployment (+automation)
- post-install scripting (+automation)
- transparent installation and system (+security)
- template customization (+security)
- custom nova boot (+tech)

One caveat: as mentioned above, not all images boot automatically yet. Several bug trackers are open across different vendors for issues at **boot time**, in **grub config**, during the **init phase**, and so on.

If you see these errors through **KVM/IPMI**, it actually means the deployment went well: the sticking point isn't the deployment technology itself, but the image installed on the disks.
