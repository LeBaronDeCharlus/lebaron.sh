---
title: "Additional Volume Public Cloud Ovh"
date: 2017-07-31
tags: ["ovh", "linux"]
draft: false
---

Adding an extra disk or volume to your OVH Public Cloud instance takes a few manual steps. Here's how to do it.

First, identify your new disk:

```bash
fdisk -l
```

The output will vary depending on your system (`sd{x}`, `vd{x}`).

Then create a new partition:

```
# parted /dev/{{disk}}
mktable gpt
mkpart primary ext4 512 100%
quit
```

Format it:

```bash
mkfs -t ext4 -L rootfs /dev/{{disk}}1
```

Mount it:

```bash
mount /dev/{{disk}}1 /mnt
```

To make this persistent across reboots, we'll need the volume's **UUID**. Let's grab the block ID.

```
# blkid
/dev/sdb1: LABEL="rootfs" UUID="6b75bbb4-b311-4b9d-a8fd-6e6ff23c401f" TYPE="ext4" PARTLABEL="primary" 
PARTUUID="e20dc227-9d10-41c4-a714-2fb53d190c11"
/dev/sda1: UUID="9abb590f-8a5e-496f-ad2a-2c877415bdc5" TYPE="ext4" PARTUUID="71036eb1-01"
```

Add it to `/etc/fstab` along with the mount information.

```
# vim /etc/fstab
[...]
UUID=6b75bbb4-b311-4b9d-a8fd-6e6ff23c401f /mnt ext4 errors=remount-ro,discard 0 1
```

Reboot once to confirm everything is configured correctly.
