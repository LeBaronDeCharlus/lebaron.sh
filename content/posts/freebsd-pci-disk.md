---
title: "Freebsd Pci Disk Space"
date: 2018-03-04
tags: ["ovh", "linux"]
draft: false
---

When you create an OVH Public Cloud instance running FreeBSD with a given amount of disk space, say 50G, you'll find that it isn't fully applied to your partition.

First, let's look at what we have:
```
# gpart show
=>      40  10239920  da0  GPT  (50G) [CORRUPT]
        40      1024    1  freebsd-boot  (512K)
      1064       984       - free -  (492K)
      2048  10235904    2  freebsd-zfs  (4.9G)
  10237952      2008       - free -  (1.0M)
```

Notice that our volume da0 is tagged as CORRUPT. Don't panic, the FreeBSD handbook has the answer, as always. Here's the relevant excerpt:

> If the disk was formatted with the GPT partitioning scheme, it may show as “corrupted” because the GPT backup partition table is no longer at the end of the drive. Fix the backup partition table with gpart:
```
# gpart recover ada0
ada0 recovered
```


Let's apply this to our server, replacing `ada0` with `da0`:
```
# gpart recover da0
da0 recovered
```

Let's check:
```
# gpart show
=>       40  104857520  da0  GPT  (50G)
         40       1024    1  freebsd-boot  (512K)
       1064        984       - free -  (492K)
       2048   10235904    2  freebsd-zfs  (4.9G)
   10237952   94619608       - free -  (45G)
```

Much better! Our da0 "disk" now shows 50G. However, if we look more closely at the system, we see that not all of that space is actually available.
```
# df -h
Filesystem            Size    Used   Avail Capacity  Mounted on
zroot/ROOT/default    4.7G    493M    4.2G    10%    /
devfs                 1.0K    1.0K      0B   100%    /dev
zroot                 4.2G     96K    4.2G     0%    /zroot
```

Once again, no need to panic, the handbook has us covered.

Let's apply the 45G of free space:
```
# gpart resize -i 2 -a 4k -s 50G da0
da0p2 resized
```

Let's check:
```
# gpart show
=>       40  104857520  da0  GPT  (50G)
         40       1024    1  freebsd-boot  (512K)
       1064        984       - free -  (492K)
       2048   94371840    2  freebsd-zfs  (45G)
   94373888   10483672       - free -  (5.0G)
```

We're making progress, but the space still isn't usable, as confirmed by df:
```
# df -h
Filesystem            Size    Used   Avail Capacity  Mounted on
zroot/ROOT/default    4.7G    493M    4.2G    10%    /
devfs                 1.0K    1.0K      0B   100%    /dev
zroot                 4.2G     96K    4.2G     0%    /zroot
```

We need to tell our zpool to use this space.

Let's first check the pool's status.
```
# zpool status
  pool: zroot
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	zroot       ONLINE       0     0     0
	  da0p2     ONLINE       0     0     0
```

Enable autoexpand on zroot:
```
# zpool set autoexpand=on zroot
```

Apply it to da0p2:
```
# zpool online -e zroot /dev/da0p2
```

One last check:
```
# df -h
Filesystem            Size    Used   Avail Capacity  Mounted on
zroot/ROOT/default     44G    493M     43G     1%    /
devfs                 1.0K    1.0K      0B   100%    /dev
zroot                  43G     96K     43G     0%    /zroot
```

And that's it!
