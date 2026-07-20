---
title: "Freebsd Pci Disk Space"
date: 2018-03-04
tags: ["ovh", "linux"]
draft: false
---

When you create an OVH Public Cloud instance under Freebsd with a certain amount of disk space, let’s say 50G, you will find that it is not applied on your partition.

First let's look at what we have:
```
# gpart show
=>      40  10239920  da0  GPT  (50G) [CORRUPT]
        40      1024    1  freebsd-boot  (512K)
      1064       984       - free -  (492K)
      2048  10235904    2  freebsd-zfs  (4.9G)
  10237952      2008       - free -  (1.0M)
```

We note that our volume da0 is tagged as CORRUPT. Don't panic, everyone knows that the Freebsd handbook is great. I quote:

> If the disk was formatted with the GPT partitioning scheme, it may show as “corrupted” because the GPT backup partition table is no longer at the end of the drive. Fix the backup partition table with gpart:
```
# gpart recover ada0
ada0 recovered
```


Well, let's apply this to our server by replacing `ada0` by `da0` :
```
# gpart recover da0
da0 recovered
```

Check :
```
# gpart show
=>       40  104857520  da0  GPT  (50G)
         40       1024    1  freebsd-boot  (512K)
       1064        984       - free -  (492K)
       2048   10235904    2  freebsd-zfs  (4.9G)
   10237952   94619608       - free -  (45G)
```

Much better!
We see that our da0 "disk" has 50G. However if we look more closely at our system, we see that not all the space is present.
```
# df -h
Filesystem            Size    Used   Avail Capacity  Mounted on
zroot/ROOT/default    4.7G    493M    4.2G    10%    /
devfs                 1.0K    1.0K      0B   100%    /dev
zroot                 4.2G     96K    4.2G     0%    /zroot
```

Once again, don't panic. The handbook is our friend.

Let's apply the 45G free on our score :
```
# gpart resize -i 2 -a 4k -s 50G da0
da0p2 resized
```

Check :
```
# gpart show
=>       40  104857520  da0  GPT  (50G)
         40       1024    1  freebsd-boot  (512K)
       1064        984       - free -  (492K)
       2048   94371840    2  freebsd-zfs  (45G)
   94373888   10483672       - free -  (5.0G)
```

Well, we are moving forward, however, the space is not yet usable as the return from df :
```
# df -h
Filesystem            Size    Used   Avail Capacity  Mounted on
zroot/ROOT/default    4.7G    493M    4.2G    10%    /
devfs                 1.0K    1.0K      0B   100%    /dev
zroot                 4.2G     96K    4.2G     0%    /zroot
```

We must ask to our zpool to use this space.

Let’s first check our pool.
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

Ask it we want to autoexpand on zroot
```
# zpool set autoexpand=on zroot
```

Apply it on da0p2 :
```
# zpool online -e zroot /dev/da0p2
```

Last check :
```
# df -h
Filesystem            Size    Used   Avail Capacity  Mounted on
zroot/ROOT/default     44G    493M     43G     1%    /
devfs                 1.0K    1.0K      0B   100%    /dev
zroot                  43G     96K     43G     0%    /zroot
```

This is it !
