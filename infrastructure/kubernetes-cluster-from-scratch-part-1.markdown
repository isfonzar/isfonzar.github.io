---
layout: post
title:  "Kubernetes Cluster from Scratch - Part 1"
date:   2020-05-05 16:13:31 +0200
categories: infrastructure devops kubernetes
---

# Kubernetes Cluster from Scratch - Part 1

This post is based on both [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) by [Kelsey Hightower](https://github.com/kelseyhightower) and the homonymous material from [William Boyd](https://linuxacademy.com/course/kubernetes-the-hard-way/).

This is not intended as a substitute for the aforementioned content, and I would strongly advise to use them as reference instead, as they are both more complete than the content present here.

### Why set up a Kubernetes Cluster from Scratch?

> "Kubernetes The Hard Way is optimized for learning, which means taking the long route to ensure you understand each task required to bootstrap a Kubernetes cluster" 
>
> \- Kelsey Hightower, Kubernetes the Hard Way

I decided to learn how to set up a kubernetes cluster from scratch because I was unsatisfied of using it without understanding exactly how it works. I believe that setting everything up from scratch is a great exercise and opportunity to learn more about how Kubernetes works.

## Geting Started

- To set-up a kubernetes cluster you'll need 5 servers in total
  - 2 will be used for the kubernetes controllers
  - 2 for the kubernetes worker nodes
  - 1 for the kubernetes API load balancer

It's possible, for the sake of learning, to use less servers. The steps present here work just as fine with 1 of each kind of server as well as 10 or more. Also for the sake of learning, it's good to have at least some redundancy in the controllers and in the workers. Kelsey Hightower recommends 3 of each, while William Boyd works with 2 of each, I decided to go with 2 as I believe it's the minimum to get the most of the concepts.

After provisioning all the servers and creating the users in them, I recommend creating the following environment variables on your workstation, as most of the commands will use those and it will save you the work of having to replace them.

```bash
CONTROLLER0_HOST=controller0.mylabserver.com
CONTROLLER0_IP=172.34.0.0
CONTROLLER1_HOST=controller1.mylabserver.com
CONTROLLER1_IP=172.34.0.1
WORKER0_HOST=worker0.mylabserver.com
WORKER0_IP=172.34.1.0
WORKER1_HOST=worker1.mylabserver.com
WORKER1_IP=172.34.1.1
API_HOST=kubernetes.mylabserver.com
API_IP=172.34.2.0

# Specify the user you will be connecting to those servers with
CLOUD_USER=cloud_user
```

## Setting up the Certificates

The first step to set up our Kubernetes Cluster is to create the [Public key infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure) to manage the security and authentication in the cluster so nodes can communicate with each other and also with the Kubernetes API.

### Requirements

Install Cloudflare's [cfssl](https://github.com/cloudflare/cfssl) on your workstation.

```bash
$ wget -q --show-progress --https-only --timestamping \
  https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 \
  https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64

$ chmod +x cfssl_linux-amd64 cfssljson_linux-amd64

$ sudo mv cfssl_linux-amd64 /usr/local/bin/cfssl
$ sudo mv cfssljson_linux-amd64 /usr/local/bin/cfssljson

$ cfssl version # check the installation
```

`cfssl` is Cloudflare's open-source PKI tool that will provide us with all the functionalities we need to set up the Certificate Authority (CA) and the certificates themselves.

### Provisioning the Certificate Authority

Create the config files for the certificate authority

```bash
# Config file for the certificate authority
cat > ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

# Certificate signing request
cat > ca-csr.json << EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Oregon"
    }
  ]
}
EOF

# Generate the certificates
cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```

As a result, we should now have the files `ca-key.pem` and `ca.pem`, the private certificate and the public certificate respectively. 

### Client and Server Certificates

Now that we have our CA, we can use it to generate the client and server certificates.

#### Generate the admin certificate

```bash
# Create admin certificate signing request
cat > admin-csr.json << EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:masters",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

# Generate admin certificate
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
```

As a result, we should now have the files `admin-key.pem` and `admin.pem`.

#### Generate the clients certificates

Generate certificates for the workers, in case you haven't set up the environment variables in the beginning of this post, replace them in the following commands.

```bash
# Certificate signing request for the first worker
cat > ${WORKER0_HOST}-csr.json << EOF
{
  "CN": "system:node:${WORKER0_HOST}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

# Generate first worker's certificate
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${WORKER0_IP},${WORKER0_HOST} \
  -profile=kubernetes \
  ${WORKER0_HOST}-csr.json | cfssljson -bare ${WORKER0_HOST}

# Certificate signing request for the second worker
cat > ${WORKER1_HOST}-csr.json << EOF
{
  "CN": "system:node:${WORKER1_HOST}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

# Generate second worker's certificate
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${WORKER1_IP},${WORKER1_HOST} \
  -profile=kubernetes \
  ${WORKER1_HOST}-csr.json | cfssljson -bare ${WORKER1_HOST}
```

After executed, we should have the public and private keys for both of the workers.

<details>
<summary>Generate certificate for the Kube Controller Manager</summary>
<p>

```bash
$ cat > kube-controller-manager-csr.json << EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

$ cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
```

</p>
</details> 

<details>
<summary>Generate certificate for the Kube Proxy</summary>
<p>

```bash
$ cat > kube-proxy-csr.json << EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:node-proxier",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

$ cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy
```

</p>
</details> 

<details>
<summary>Generate certificate for the Kube Scheduler Client</summary>
<p>

```bash
$ cat > kube-scheduler-csr.json << EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

$ cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler
```

</p>
</details> 

```bash
CERT_HOSTNAME=10.32.0.1,${CONTROLLER0_IP},${CONTROLLER0_HOST},${CONTROLLER1_IP},${CONTROLLER1_HOST},${API_IP},${API_HOST},127.0.0.1,localhost,kubernetes.default
```

```bash
$ cat > kubernetes-csr.json << EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

$ cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${CERT_HOSTNAME} \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes
```

You should get as the result, the files `kubernetes.pem` and `kubernetes-key.pem`.

### Generating the Service Account Key Pair

This is the certificate that Kubernetes will use to authenticate service accounts connected to the cluster.

```bash
$ cat > service-account-csr.json << EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

$ cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account
```

### Distributing the certificate files

```bash
# Copying certificates to workers
$ scp ca.pem ${WORKER0_HOST}-key.pem ${WORKER0_HOST}.pem cloud_user@${WORKER0_IP}:~/
$ scp ca.pem ${WORKER1_HOST}-key.pem ${WORKER1_HOST}.pem cloud_user@${WORKER1_IP}:~/

# Copying certificates to controllers
$ scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem cloud_user@${CONTROLLER0_IP}:~/
$ scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem cloud_user@${CONTROLLER1_IP}:~/
```

## Conclusion

You should now have all the certificates present in the workers and in the controllers. This can be verified by logging into those nodes and checking for the presence of them.