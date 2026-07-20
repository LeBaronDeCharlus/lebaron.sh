---
title: "Bring Your Own Image"
date: 2020-07-20
tags: ["linux", "ovh"]
draft: false
---

When I was working at **OVHCloud** company, I've developed a feature called **BYOI** (Bring Your Own Image).

**Bring Your Own Image** technology allows you to boot any cloud (or not) images on a baremetal.

Even if today not all editors are ready to provide completely agnostic images (and I mean **UEFI** ready and/or **legacy boot**), by triturating (**packer**) a bit the whole, we can have something functional.

This mark the first step towards the **Hybrid IAC**.

[The documentation is here](https://docs.ovh.com/gb/en/dedicated/bringyourownimage/)

**What does it really do?**

- API deployment (+automation)
- post-install scripting (+automation)
- transparent installation and system (+security)
- template customization (+security)
- nova boot custom (+tech)

A little warning, as I said above, not all images are ready to boot automatically yet, there are many bug-tracks opened on different editors because of unwanted behaviors at **boot time**, **grub config**, **init phase** etc

If you see these errors via a **KVM/IPMI**, it means that the installation went well, but the sticking point is not the deployment technology itself, but the image installed on the disks.
