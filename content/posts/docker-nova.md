---
title: "Dockerize Nova client"
date: 2017-08-08
tags: ["containers", "ovh"]
draft: false
---

With OVH Public Cloud, you can control your OpenStack instances directly from the Nova client.

Working from the command line is much more practical than going through the manager, which, let's be honest, is often slow.

To avoid setting up a working Nova environment every time, I found it more convenient to package everything into a Docker image. That way, the only thing you need installed on your workstation is the Docker daemon.

The Dockerfile looks like this:

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

I used a plain Debian image here because I already had it locally, but feel free to swap in whichever base image you prefer and adjust the RUN instructions accordingly.

You'll also need to fill in the environment variables with your OpenStack credentials.

Let's build it:

```bash
sudo docker build -t clientNova .
```

Then run your container like this:

```bash
sudo docker run --rm -it novaClient bash
```

You can also add an alias to your `.bashrc` / `.zshrc`:

```bash
alias nova="sudo docker run --rm -it novaClient bash"
```

Enjoy!
