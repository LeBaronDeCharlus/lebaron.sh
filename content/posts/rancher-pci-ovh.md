---
title: "Setup Rancher cluster on OVH Public Cloud"
date: 2017-08-04
tags: ["containers", "ovh"]
draft: false
---

The world of hosting is changing, and so is the world of application development. Today we are turning less and less to dedicated hosting for a single application, but more to build the infrastructure that will support it. 

In this sense, we prefer to use an Iaas solution on dedicated "bare metal" for our application overlay than pure "bare metal" per service.

Application deployment missions pushed by developers must fit with the technology and logic of production. Which `pipelines` should we use for our `CI`, `CD` ?

This post will not aim at answering the question of the pipelines to be implemented, it will be a question of `Ranching`, coupled with its `Cattle` orchestrator, on Public Cloud OVH `Openstack`.

### Installation :


The installation of `PCI` (Public Cloud) instances is the fastest step. We need to start with 5 instances:

- 1 LoadBalancer/ReverseProxy (`HA-Proxy` or `Nginx`) for €2.99
- 3 RancherServer in Cluster under `Galera` at 5€99
- 1 Node Worker (for our applications) at the price you want to put for your performance

#### LoadBalancer Nginx :

For Nginx installation, nothing very complex, a €2.99 instance will be more than enough since the only job of the server will be to forward the request to our rancher cluster.

The conf file will be :

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

Replace the `upstream` by the ips of your servers.

Note that it is also possible to dockerize this service. This way, the day you put a second LB on the front end, there will only be one container to place on your instance.

#### Rancher installation :

For Rancher part, there are no great difficulties either. 5€99 instances will be more than enough for the needs.

I recommend the [official Rancher documentation](http://rancher.com/docs/rancher/v1.0/en/installing-rancher/installing-server/multi-nodes/) on this subject.

We will start by deploying on each Rancher server:

```bash
sudo docker run -d --restart=unless-stopped -p 8080:8080 rancher/server
```


Then we will add the servers to each other via the UI. (Infrastructure > Hosts > Add Host)

You should get the following code to run on each node :

```bash
sudo docker run -e CATTLE_AGENT_IP="1.2.3.4"  -e CATTLE_HOST_LABELS='galera=true'  --rm --privileged -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/rancher:/var/lib/rancher rancher/agent:v1.2.5 http://1.2.3.4:8080/v1/scripts/4956918455D4D9BE3AF1:1483142400000:Fscj9CvRSrx0mS05E4kdWDkb0E
```

Once this step is done, it will then be possible to use the `Galera` image proposed by Rancher (in the catalog) on our 3 servers.
A node will then be present on each server.

We will be able to initialize the cluster on a Galera node:

```
> CREATE DATABASE IF NOT EXISTS cattle COLLATE = 'utf8_general_ci' CHARACTER SET = 'utf8';
> GRANT ALL ON cattle.* TO 'cattle'@'%' IDENTIFIED BY 'cattle';
> GRANT ALL ON cattle.* TO 'cattle'@'localhost' IDENTIFIED BY 'cattle';
```

Then check on the other two servers that the basic entries are also present.

Then let's stop one Rancher server and start it using Galera for persistence of its data :

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

Do the same for the other two servers so each Rancher launches using the Galera database.

It is possible to check if Rancher is `clustered` via the UI in: Admin > High Availability.

#### Addition of Worker Nodes.

You now have your `Rancher` cluster with a front-end LB, it is now possible to add `Worker` nodes to your cluster.
It's very easy to add them directly with the Rancher utility. Now it's up to you to see which type of instance best suits your resource needs.

Rancher's default environment uses the `Cattle` orchestrator, so once your Workers nodes are configured, you can deploy your `Docker` containers directly from your cluster.

### Conclusion

The installation of a Rancher environment is fast, the Public Cloud OVH allows to quickly deploy the necessary instances of Rancher.
The ease of use offered by the Rancher/Cattle duo allows an efficient and fluid commissioning.

We will see in a next article how to set up an HA environment with `Kubernetes`, still under the OVH PCI and using the environment template proposed by Rancher.
