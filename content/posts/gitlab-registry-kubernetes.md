---
title: "Add Gitlab Registry in Kubernetes"
date: 2017-08-10
tags: ["containers"]
draft: false
---

Now that Gitlab offers its own image registry, it is possible to use it directly in our K8s environment!
If you missed the info (which is starting to date now), I refer you to [this article](https://about.gitlab.com/2016/05/23/gitlab-container-registry/).

To add the Gitlab private registry in Kubernetes you have to create a secret :

```
> kubectl create secret docker-registry regsecret --docker-server=registry.gitlab.xyz --docker-username='' --docker-password='' --docker-email=""
```

Where :
 
```
--docker-server         regitry gitlab
--docker-username       user gitlab autorisé à acceder au registry
--docker-password       son mot de passe
--docker-email          son email
```

We will check secret creation :

```
> kubectl get secret regsecret
NAME        TYPE                      DATA      AGE
regsecret   kubernetes.io/dockercfg   1         19h
```

Show details :

```
> kubectl get secret regsecret --output=yaml
apiVersion: v1
data:
  .dockercfg: eyJiXXXXXXteoirutXXXXetusrnXX=
kind: Secret
metadata:
  creationTimestamp: 2017-08-07T13:32:09Z
  name: regsecret
  namespace: default
  resourceVersion: "2783"
  selfLink: /api/v1/namespaces/default/secrets/regsecret
  uid: cfeXXXb7-7b74-XXX-XXX907-4iu8237492
type: kubernetes.io/dockercfg
```

It’s now possible to use images from your registry directly in your `deployment`.

Example

```yaml
apiVersion: apps/v1beta1 
kind: Deployment
metadata:
  name: test-registry
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: my-app
        version: v1
    spec:
      containers:
      - name: my-app
        image: registry.gitlab.xyz/images/my-app:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 80
      imagePullSecrets:
        - name: regsecret
```

See where we call `secret` : 

```yaml
imagePullSecrets:
  - name: regsecret
```

Enjoy !
