---
layout: post
title:  "Kubernetes Draft"
date:   2019-12-27 08:25:31 +0200
categories: infrastructure devops kubernetes
---

# Build your own Kubernetes Cluster from Scratch

Needed:
 - 2 Kubernetes controllers
 - 2 Kubernetes worker nodes
 - 1 Kubernetes API load balancer


1. Install cfssl and kubectl on your workstation.

[cfssl](https://github.com/cloudflare/cfssl):

```bash
$ wget -q --show-progress --https-only --timestamping \
  https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 \
  https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64

$ chmod +x cfssl_linux-amd64 cfssljson_linux-amd64
$ sudo mv cfssl_linux-amd64 /usr/local/bin/cfssl
$ sudo mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
$ cfssl version # check the installation
```

[kubectl]():

```bash
$ wget https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubectl
$ chmod +x kubectl
$ sudo mv kubectl /usr/local/bin/
$ kubectl version --client # check the installation
```

### References

https://github.com/kelseyhightower/kubernetes-the-hard-way

https://linuxacademy.com/course/kubernetes-the-hard-way/
