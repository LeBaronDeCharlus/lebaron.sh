---
title: "Migrate Openvz To Lxc"
date: 2017-07-31
tags: ["containers", "linux"]
draft: false
---

I recently had to migrate containers from a Proxmox 3 host (running OpenVZ) to a Proxmox 4 host (running LXC).

The problem was the sheer number of containers to migrate and "convert" to run under LXC, so I needed a way to automate the process as much as possible.

Luckily, the [migration documentation](https://pve.proxmox.com/wiki/Convert_OpenVZ_to_LXC) is very well detailed, so I used it as the basis for a couple of scripts.

I split the operation into two scripts: an export script and an import script.

#### The export

The export script takes two parameters: `the container ID` and `the IP` of the destination server.

You'll also need to define a few variables used to send data via `scp`.

Here's the *export* procedure:

```bash
# Stop & Dump
sudo vzctl stop $ID && \
    echo "$ID stopped [OK]" && \
    sudo vzdump $ID -dumpdir /home/$USER/vzdump && \
    echo "$ID : dump [OK]" && \
```

We check that the dump exists:

```bash
# DumpName
vzDumpName=$(ls /home/$USER/vzdump/)

if [ -z $vzDumpName ] ; then
    echo "No dump found in /home/$USER/vzdump/"
    exit
fi
```

Then we send it to `Proxmox 4`:

```bash
sudo scp -i /home/$USER/.ssh/id_rsa "-P $rPort" $vzDumpName $rUSER@$rIP:$rPath && \
    echo "Sending Dump [OK]" && \
    sudo scp -i /home/$USER/.ssh/id_rsa "-P $rPort" vz.log $rUSER@$rIP:$rPath && \
    echo "Sending Log [OK]" && \
    sudo rm $vzDumpName && \
    sudo rm vz.log && \
    echo "SCP $vzDumpName on $rIP [OK]"
```

#### Import

The import script only takes one argument: the container's "ID".

Then we kick off the **conversion** and **restoration** step.

```bash
sudo pct restore $ID $dumpPath/$dumpName && \
    echo "pct $ID restoring [OK]"

# At this poin, you can set network configuration
# exemple : pct set 101 -net0 name=eth0,bridge=vmbr0,ip=192.168.15.144/24,gw=192.168.15.1
# I prefer doing it manually

sudo pct start $ID
sudo pct enter $ID
```

One more thing I forgot to mention: there's a small logging system for these operations, so you can trace the process a bit.
