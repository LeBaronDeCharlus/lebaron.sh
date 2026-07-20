---
title: "Galera Monitoring"
date: 2017-07-30
tags: ["devops"]
draft: false
---

Monitoring a database in standalone mode is one thing, but when it comes to clustering, it's a little more complex.

This is the case with Galera clustering (mariaDB/mysql). Zabbix (&co) offered me simple solutions for single database servers, but I didn't find a really interesting template for monitoring a Galera cluster for production.

So several questions, what to monitor, how to alert, what's the best method?

I based myself on [the official Galera documentation](http://galeracluster.com/documentation-webpages/monitoringthecluster.html) to have all the important elements to monitor.
For the choice of the Golang language, it seemed to me that it provided me with the necessary functionalities.
As for the choice of alerting, I decided to use a Slack App.

Example of a healthy output :

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


You have to use the `slackApp` package in which I placed an exposed function `PayloadSlack()` to then customize your alerts.

The whole output is not displayed in the **STDOUT** when it is healthy. Nevertheless you just have to uncomment the println in the `main.go` to be able to perform your tests.
