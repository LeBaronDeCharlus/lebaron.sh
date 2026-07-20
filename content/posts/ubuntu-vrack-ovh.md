---
title: "Ubuntu Vrack Ovh Fix"
date: 2018-03-04
tags: ["ovh", "linux"]
draft: false
---

You may have noticed this already: when you spin up an OVH Public Cloud (PCI) instance running Ubuntu with Vrack enabled, your VM won't have its private IP at boot time.
So yes, I'm not a fan of Ubuntu, but sometimes you don't get to choose.

Anyway, long story short: no private IP, and that's a shame. (RT)

The fix trick is stupid.
Very stupid.

Add:

```
allow-hotplug ens4
iface ens4 inet dhcp
```

In the `/etc/network/interface` file.

Then:

```bash
systemctl restart network
```

and

```bash
ifup ens4
```

That’s it

I sent an email to OVH's mailing list ([cloud]), because I still find it strange that this bug exists at all.

That was back on 21 November 2017.

```
Hello la team,

Juste une petite remarque sur l'installation d'un Ubuntu PCI avec Vrack.
En fait je suis obligé d'ajouter :

allow-hotplug ens4
iface ens4 inet dhcp

Dans /etc/network/interface puis

systemctl restart network
et
ifup ens4

Il me semble que sur les autres distrib' les confs network s'ajoutent automatiquement à l'install non ?


Je passe peut-être à côté d'un truc... en même temps je ne connais pas bien les systèmes en .deb...

Bises à tout le monde

F00b4rch
```

Answer:

```
21/11/2017
	
À cloud
Hello,
Exact, sur debian 9 (voir 8), les interfaces sont montée automatiquement...
Je crois qu'il a un bug d'ouvert cote ubuntu pour ça, si je le retrouve je te l'envoi.
```
