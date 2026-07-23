---
title: "Galera Monitoring"
date: 2017-07-30
tags: ["devops"]
draft: false
---

Monitoring a database in standalone mode is one thing, but clustering makes it a lot more complex.

That's exactly the case with Galera clustering (MariaDB/MySQL). Zabbix and similar tools gave me simple solutions for single database servers, but I couldn't find a template I actually liked for monitoring a Galera cluster in production.

So a few questions came up: what to monitor, how to alert, and what the best approach would be.

I used [the official Galera documentation](http://galeracluster.com/documentation-webpages/monitoringthecluster.html) as my reference for all the important elements to keep an eye on. I picked Go because it gave me the functionality I needed, and for alerting I went with a Slack app.

Example of a healthy output:

```
> go run main.go

### Version ####
2017/06/16 14:19:48 Serveur n1 - version 5.6.35-1xenial
2017/06/16 14:19:48 Serveur n2 - version 5.6.35-1xenial
2017/06/16 14:19:48 Serveur n3 - version 5.6.35-1xenial
### UUID ###
2017/06/16 14:19:48 n1 a1c404a9-51b4-11e7-b057-237cc5970d38
2017/06/16 14:19:48 n2 a1c404a9-51b4-11e7-b057-237cc5970d38
2017/06/16 14:19:48 n3 a1c404a9-51b4-11e7-b057-237cc5970d38
### Nodes ###
2017/06/16 14:19:48 Total Nodes : 3
2017/06/16 14:19:48 Number of Nodes counts : 3
2017/06/16 14:19:48 Number of Nodes counts : 3
2017/06/16 14:19:48 Number of Nodes counts : 3
### STATUS ###
2017/06/16 14:19:48 n1 status : Primary
2017/06/16 14:19:48 n2 status : Primary
2017/06/16 14:19:48 n3 status : Primary
2017/06/16 14:19:48 n1 is ready : [ON]
2017/06/16 14:19:48 n2 is ready : [ON]
2017/06/16 14:19:48 n3 is ready : [ON]
2017/06/16 14:19:48 n1 is connected : [ON]
2017/06/16 14:19:48 n2 is connected : [ON]
2017/06/16 14:19:48 n3 is connected : [ON]
### Average Replication ###
2017/06/16 14:19:48 Average on n2 : 0.000000
2017/06/16 14:19:48 Average on n3 : 0.000000
2017/06/16 14:19:48 Average on n1 : 0.100000
```


You need to use the `slackApp` package, where I've exposed a `PayloadSlack()` function so you can customize your alerts.

The full output isn't printed to **STDOUT** when everything is healthy. If you want to run your own tests, just uncomment the println calls in `main.go`.
