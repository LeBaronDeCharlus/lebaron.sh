---
title: "Setup Rancher cluster on OVH Public Cloud"
date: 2017-08-04
tags: ["containers", "ovh"]
draft: false
---

The hosting world is changing, and so is the way we build applications. We're turning less and less to dedicated hosting for a single application, and more toward building the infrastructure that will support it.

In practice, this means favoring an IaaS layer on top of dedicated "bare metal" for our application overlay, rather than provisioning pure "bare metal" per service.

The deployments developers push need to fit the technology and logic of production. Which `pipelines` should we use for `CI`/`CD`?

This post won't try to answer the pipeline question. Instead, it's about `Rancher`, paired with its `Cattle` orchestrator, running on OVH Public Cloud `OpenStack`.

### Installation

Spinning up the `PCI` (Public Cloud) instances is the fastest part. We'll start with 5 instances:

- 1 LoadBalancer/ReverseProxy (`HA-Proxy` or `Nginx`) for €2.99
- 3 RancherServer instances clustered under `Galera` at €5.99 each
- 1 Worker node (for our applications), sized to whatever performance you need

#### LoadBalancer Nginx

Setting up Nginx is straightforward: a €2.99 instance is more than enough, since the server's only job is to forward requests to our Rancher cluster.

Here's the config file:

```nginx
upstream rancher {
    server rancher-server1:8080;
    server rancher-server2:8080;
    server rancher-server3:8080;
}

map $http_upgrade $connection_upgrade {
    default Upgrade;
    ''      close;
}

server {
    listen 443 ssl spdy;
    server_name <server>;
    ssl_certificate <cert_file>;
    ssl_certificate_key <key_file>;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Port $server_port;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://rancher;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_read_timeout 900s;
    }
}

server {
    listen 80;
    server_name <server>;
    return 301 https://$server_name$request_uri;
}
```

Replace the `upstream` block with your servers' IPs.

You can also dockerize this service. That way, if you later add a second load balancer on the front end, it's just one more container to deploy on your instance.

#### Rancher installation

The Rancher side isn't much harder either. The €5.99 instances are more than enough for what we need.

I'd recommend following the [official Rancher documentation](http://rancher.com/docs/rancher/v1.0/en/installing-rancher/installing-server/multi-nodes/) for this part.

Start by deploying this on each Rancher server:

```bash
sudo docker run -d --restart=unless-stopped -p 8080:8080 rancher/server
```


Then add the servers to each other through the UI (Infrastructure > Hosts > Add Host).

You'll get a command like this to run on each node:

```bash
sudo docker run -e CATTLE_AGENT_IP="1.2.3.4"  -e CATTLE_HOST_LABELS='galera=true'  --rm --privileged -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/rancher:/var/lib/rancher rancher/agent:v1.2.5 http://1.2.3.4:8080/v1/scripts/4956918455D4D9BE3AF1:1483142400000:Fscj9CvRSrx0mS05E4kdWDkb0E
```

Once that's done, you can deploy the `Galera` image from Rancher's catalog on all 3 servers, giving each server its own node.

We can now initialize the cluster from one Galera node:

```
> CREATE DATABASE IF NOT EXISTS cattle COLLATE = 'utf8_general_ci' CHARACTER SET = 'utf8';
> GRANT ALL ON cattle.* TO 'cattle'@'%' IDENTIFIED BY 'cattle';
> GRANT ALL ON cattle.* TO 'cattle'@'localhost' IDENTIFIED BY 'cattle';
```

Then check the other two servers to confirm the same entries are present.

Now stop one Rancher server and restart it using Galera for data persistence:

```bash
sudo docker run -d --restart=unless-stopped -p 8080:8080 rancher/server \
    --db-host myhost.example.com --db-port 3306 --db-user username --db-pass password --db-name cattle
```

Where :

```
--db-host               IP du serveur MySQL
--db-port               port du serveur MySQL (default: 3306)
--db-user               username MySQL login (default: cattle)
--db-pass               password MySQL login (default: cattle)
--db-name               nom de la base MySQL (default: cattle)
```

Do the same on the other two servers so each Rancher instance starts using the Galera database.

You can confirm Rancher is `clustered` via the UI, under Admin > High Availability.

#### Adding Worker Nodes

With your `Rancher` cluster and front-end LB now in place, you can add `Worker` nodes to your cluster. This is easy to do directly from the Rancher utility, so it's really just up to you to pick the instance type that best suits your resource needs.

Rancher's default environment uses the `Cattle` orchestrator, so once your Worker nodes are configured, you can deploy `Docker` containers directly from your cluster.

### Conclusion

Setting up a Rancher environment is fast: OVH Public Cloud lets you deploy the instances you need in no time. The ease of use the Rancher/Cattle duo offers makes for smooth, efficient commissioning.

In a future article, we'll look at setting up an HA environment with `Kubernetes`, still on OVH PCI, using the environment template Rancher provides.
