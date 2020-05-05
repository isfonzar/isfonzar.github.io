---
layout: post
title:  "Kubernetes Draft"
date:   2019-12-27 08:25:31 +0200
categories: infrastructure devops kubernetes
---

# Kubernetes Cluster from Scratch - Part 1 - Certificates

This post is based on both [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) by [Kelsey Hightower](https://github.com/kelseyhightower) and the homonymous material from [William Boyd](https://linuxacademy.com/course/kubernetes-the-hard-way/).

This is not intended as a substitute for the aforementioned content, and I would strongly advise to use them as reference instead, as they are both more complete and more didactic than the content present here.

With that out of the way, the intent of this post is, first, a cheatsheet so I can quickly refer to in the future, and second, because I've found out that I learn best by writting down what I'm doing and explaining it "to someone else", in this case, the reader.

### Why?

I decided to learn how to set up a kubernetes cluster from scratch because I was unsatisfied of using it as part of my job and not understanding how it works.

## Get Started

- To set-up a kubernetes cluster from scratch you'll need 5 servers in total
  - 2 will be used for the kubernetes controllers
  - 2 for the kubernetes worker nodes
  - 1 for the kubernetes API load balancer

It's possible, for the sake of learning, to use less servers. The steps present here work just as fine with 1 of each kind of server as well as 10 or more. Also for the sake of learning, it's good to have at least some redundancy in the controllers and in the workers. Kelsey Hightower recommends 3 of each, while William Boyd recommends 2 of each, I decided to go with 2 as I believe it's the minimum to get the most of the concepts in those guide.

After provisioning all the servers, I recommend creating environment variables, as most of the commands will use those and it will save you the work of having to replace them in the commands.

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
```

TODO: Create users for all the instances.

## Setting up the Certificates

Certificate Authority (CA)

We will provision a certificate authority

We use certificates to confirm identity, they are used to prove that you are who you say you are.

A certificate authority provides the ability to confirm that a certificate is valid.

### Why are certificates needed?

Kubernetes uses certificates for a variety of security functions. (rephrase, expand)

### What certificates are needed?

- Client certificates: these certificates will provide client authentication for various users: adminm kube-controller-manager, kube-proxy, etc. All of these components need to authenticate securarily with the Kubernetes API.

- Kubernetes API Server: The API also needs to confirm its identity to the clients. So both client and server are able to verify each other.

- Service Account Key Pair: Allow users to authenticate.

### Requirements

<details>
<summary>Install <a href="https://github.com/cloudflare/cfssl" >cfssl</a> on your workstation.</summary>
<p>

```bash
$ wget -q --show-progress --https-only --timestamping \
  https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 \
  https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64

$ chmod +x cfssl_linux-amd64 cfssljson_linux-amd64

$ sudo mv cfssl_linux-amd64 /usr/local/bin/cfssl
$ sudo mv cfssljson_linux-amd64 /usr/local/bin/cfssljson

$ cfssl version # check the installation
```

</p>
</details>  

### Provisioning the Certificate Authority

<details>
<summary>Create the config files for the certificate authority</summary>
<p>

```bash
# Config file for the certificate authority
$ cat > ca-config.json << EOF
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
$ cat > ca-csr.json << EOF
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


$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```

</p>
</details>  

As a result, we should now have the files `ca.csr`, `ca-key.pem` and `ca.pem`, the certiificate authority, the private certificate and the public certificate respectively. 

#### Generate the admin certificate

<details>
<summary>Generate the admin certificate</summary>
<p>

```bash
$ cat > admin-csr.json << EOF
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

# Using the previous created certificate, we will sign the new ceretificate
$ cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
```

</p>
</details>  

As a result, we should now have the files `admin.csr`, `admin-key.pem` and `admin.pem`.

Now, let's generate the certificates for the clients.

<details>
<summary>Generate certificates for the workers</summary>
<p>

```bash
$ cat > ${WORKER0_HOST}-csr.json << EOF
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

$ cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${WORKER0_IP},${WORKER0_HOST} \
  -profile=kubernetes \
  ${WORKER0_HOST}-csr.json | cfssljson -bare ${WORKER0_HOST}

$ cat > ${WORKER1_HOST}-csr.json << EOF
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

$ cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${WORKER1_IP},${WORKER1_HOST} \
  -profile=kubernetes \
  ${WORKER1_HOST}-csr.json | cfssljson -bare ${WORKER1_HOST}

```

</p>
</details> 

After run, we should have the public and private keys for both of the workers.

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