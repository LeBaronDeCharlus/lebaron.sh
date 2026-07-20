---
title: "Migrate Openvz To Lxc"
date: 2017-07-31
tags: ["containers", "linux"]
draft: false
---

I recently had to migrate containers from a proxmox3 (under OpenVZ) to a proxmox4 (under LXC).

Problem, there are a lot of containers to migrate/"convert" to run under LXC. So I needed a way to automate the procedure as much as possible.

Luckily, the [migration documentation](https://pve.proxmox.com/wiki/Convert_OpenVZ_to_LXC) is very well detailed. So I used it to "bash" the operation.

I cut the operation under two scripts, an export script and an import script.

#### The export :

The export script takes two parameters as input: `the container ID` and `the IP` of the destination server.

It is also necessary to define some variables that will be used to send data via `scp`.

The *export* procedure is done as follows:

```bash
# Stop & Dump
sudo vzctl stop $ID && \
    echo "$ID stopped [OK]" && \
    sudo vzdump $ID -dumpdir /home/$USER/vzdump && \
    echo "$ID : dump [OK]" && \
```

We check dump is present :

```bash
# DumpName
vzDumpName=$(ls /home/$USER/vzdump/)

if [ -z $vzDumpName ] ; then
    echo "No dump found in /home/$USER/vzdump/"
    exit
fi
```

Then we send it to `Proxmox 4` :

```bash
sudo scp -i /home/$USER/.ssh/id_rsa "-P $rPort" $vzDumpName $rUSER@$rIP:$rPath && \
    echo "Sending Dump [OK]" && \
    sudo scp -i /home/$USER/.ssh/id_rsa "-P $rPort" vz.log $rUSER@$rIP:$rPath && \
    echo "Sending Log [OK]" && \
    sudo rm $vzDumpName && \
    sudo rm vz.log && \
    echo "SCP $vzDumpName on $rIP [OK]"
```

#### Import :

As for the import, only one argument is passed as input, which is the container's "ID".

Then we start the **conversion** and **restoration** part.

```bash
sudo pct restore $ID $dumpPath/$dumpName && \
    echo "pct $ID restoring [OK]"

# At this poin, you can set network configuration
# exemple : pct set 101 -net0 name=eth0,bridge=vmbr0,ip=192.168.15.144/24,gw=192.168.15.1
# I prefer doing it manually

sudo pct start $ID
sudo pct enter $ID
```

I omitted to tell you that there is a small logging system for the operations in order to be able to trace the process a bit.
