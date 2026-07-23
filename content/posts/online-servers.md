---
title: "Online servers availability"
date: 2017-08-01
tags: ["shell"]
draft: false
---

We needed servers at Online, but there was no availability! So they came to ask me if I had some kind of magic solution...

A bit of `bash`, a `notify` (in this case `Slack`), and here we go!

#### Dirty way

To get alerted via Slack, you need to create an [incoming-webhook](https://my.slack.com/services/new/incoming-webhook/), which will generate a link.

For an XC 2016 series server:

```bash
text="DISPO : https://www.online.net/fr/serveur-dedie/dedibox-xc"; json="{\"channel\": \"#infra\", \"text\": \"$text\"}" ; while true ; do curl --silent https://www.online.net/fr/serveur-dedie  | grep '<button class="btn btn--primary js-order-dedibox"' | grep -i 'xc 2016' | grep -i 'victime' || curl -s -d "payload=$json" "https://hooks.slack.com/services/XXX/XXXX/XXXX/XXXX" ; sleep 5 ; done
```

The curl only fires if a server is actually available.

#### Clean way

Just some code formatting:
```bash
#!/bin/bash

webhook="https://hooks.slack.com/services/XXX/XXXX/XXXX/XXXX"
url="https://www.online.net/fr/serveur-dedie/XXXXX"
text="DISPO : $url"
channel="test"
json="{\"channel\": \"#$channel\", \"text\": \"$text\"}"
server="XC 2016"

while true
    do curl --silent https://www.online.net/fr/serveur-dedie | \
    grep '<button class="btn btn--primary js-order-dedibox"' | \
    grep -i "$server" | grep -i 'victime' || \
    curl -s -d "payload=$json" "$webhook" ; \
    sleep 5 ; \
done
```

You can also go the extra mile by filtering on **region** and **disk type**.

#### Pig mode

Because, why not?

```bash
text="DISPO https://www.online.net/fr/serveur-dedie/dedibox-xc"; json="{\"channel\": \"#test\", \"text\": \"$text\"}" ; while true ; do curl --silent https://www.online.net/fr/serveur-dedie/dedibox-xc | egrep -io '<option value="\w+">ssd / france / dc2</option>' && curl -s -d "payload=$json" "https://hooks.slack.com/services/XXX/XXXX/XXXX" ; sleep 5 ; done
```

Next step: simulate an order?
