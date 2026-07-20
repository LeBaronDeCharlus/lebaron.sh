---
title: "Dockerize Nova client"
date: 2017-08-08
tags: ["containers", "ovh"]
draft: false
---

With OVH Public Cloud, it is possible for you to control your OpenStack instances directly from the Nova client.

It is rather practical to go through the command line rather than having to access the manager which is often, let us say it, slow.

But to avoid having to prepare each time a working environment compatible with Nova, I find more interesting to directly create a Docker image for this purpose.
In this way, no more need to install anything except the Docker daemon on its workstation.

The DockerFile looks like :

```dockerfile
FROM debian:latest

RUN apt-get update && \
    apt-get install -y \
    python-glanceclient \
    python-novaclient

env OS_AUTH_URL=""
env OS_TENANT_ID=""
env OS_TENANT_NAME=""
env OS_USERNAME=""
env OS_PASSWORD=""
env OS_REGION_NAME=""
```

Note that I use the basic debian image because I already have it locally, but you can replace the bone with the one you want and adapt the content of RUN.

You also need to fill the environment variables with information about your OpenStack env.

Let’s go with :

```bash
sudo docker build -t clientNova .
```

Then run your container as follow :

```bash
sudo docker run --rm -it novaClient bash
```

You can add alias to your `.bashrc` / `.zshrc`

```bash
alias nova="sudo docker run --rm -it novaClient bash"
```

Enjoy !
