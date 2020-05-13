---
layout: post
title:  "Kubernetes Cluster from Scratch - Part 3"
date:   2019-12-27 08:25:31 +0200
categories: infrastructure devops kubernetes
---

## What is the Kubernetes Control Plane?

The kubernetes control plane is a set of services that control the Kubernetes cluster.

Control Plane components "make global decisions about the cluster (e.g. scheduling) and detect and respond to cluster events (e.g. starting up a new pod when a replication controlle's replicas field is unsatisfied)"

Control Plane components:

- `kube-apiserver`: serves the kubernetes API, this allows users to interact with the cluster (kubectl communicates with the cluster using the API)

- `etcd`: Not part of Kubernetes, but it is essential to Kubernetes Control Plane, as Kubernetes uses it as a distributed datastore.

- `kube-scheduler`: Schedules pods on available worker nodes.

- `kube-controller-manager`: Runs a series of controllers that provide a wide range of functionality. It combines all of those into a single process.

- `cloud-controller-manager`: Handles interaction with underlying cloud providers (Azure, AWS, Google Cloud, hybrid, etc...). (Not going to be used)

These control plane components need to be installed on each controller node.

### Installing Kubernetes Control Plane

First, let's install the control plane binaries on each control node:

```bash
sudo mkdir -p /etc/kubernetes/config

wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubectl"

chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl

sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
```

### Setting up the Kubernetes API Server

This is going to be the directory for us to store some important files kubernetes is going to need.

```bash
sudo mkdir -p /var/lib/kubernetes/

sudo cp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
  service-account-key.pem service-account.pem \
  encryption-config.yaml /var/lib/kubernetes/
```

Then, let's setup some environment variables that will be used to create the `systemd` unit file.

```bash
INTERNAL_IP=$(curl http://169.254.169.254/latest/meta-data/local-ipv4) # this could have been set manually
CONTROLLER0_IP=10.0.1.48
CONTROLLER1_IP=10.0.1.239
```

Then, generate the `kube-apiserver` unit file for `systemd`:

```bash
cat << EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-admission-plugins=Initializers,NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --enable-swagger-ui=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\
  --etcd-servers=https://$CONTROLLER0_IP:2379,https://$CONTROLLER1_IP:2379 \\
  --event-ttl=1h \\
  --experimental-encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \\
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \\
  --kubelet-https=true \\
  --runtime-config=api/all \\
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \\
  --v=2 \\
  --kubelet-preferred-address-types=InternalIP,InternalDNS,Hostname,ExternalIP,ExternalDNS
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

Last line was added to add the order of priority of the servers that determines how the kubeletes return their ip addresses, we give preference to the internal ip, otherwise they might report an IP address that might not work. Things may break without that flag:

```
--kubelet-preferred-address-types=InternalIP,InternalDNS,Hostname,ExternalIP,ExternalDNS
```

### Setting up the Kubernetes Controller Manager

First thing, let's move this file to a location since it's going to be needed by the kubernetes controller manager

```bash
sudo cp kube-controller-manager.kubeconfig /var/lib/kubernetes/
```

Then, let's generate the kube-controller-manager systemd unit file:

```bash
cat << EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --address=0.0.0.0 \\
  --cluster-cidr=10.200.0.0/16 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Setting up the Kubernetes Scheduler

Move the file of the kube-scheduler into place:

```bash
sudo cp kube-scheduler.kubeconfig /var/lib/kubernetes/
```

Now, we need to create a yaml configuration file for the scheduler:

```bash
cat << EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: componentconfig/v1alpha1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF
```

And finally, the systemd unit file:

```bash
cat << EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

Lastly, start and enable all of the service:

```bash
sudo systemctl daemon-reload # required every time we change systemd unit files

sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
```

You can check the statues of the services:

```bash
sudo systemctl status kube-apiserver kube-controller-manager kube-scheduler
```

They should be all in `active (running)` status.
You can also check the status of the components through kubectl by running:

```bash
kubectl get componentstatuses --kubeconfig admin.kubeconfig
```

At this point, the basics components of the kubernetes control place are up and running.

### Enabling HTTP health checks

#### Why do we need to enable HTTP Health Checks?

In the original `Kubernetes The Hard Way`, he uses a Google Cloud Platform load balancer, to avoid sending traffic against nodes that are unhealthy.

So we create a nginx proxy to propagate those status over HTTP (setting HTTPS health checks is too much trouble)

This step is not needed, but it's a good practice.

Kubernetes only provides those via HTTPS, but sometimes you might need it through HTTP.

```bash
sudo apt-get install -y nginx
```

Create an nginx configuration file for the health check proxy:

```bash
cat > kubernetes.default.svc.cluster.local << EOF
server {
  listen      80;
  server_name kubernetes.default.svc.cluster.local;

  location /healthz {
     proxy_pass                    https://127.0.0.1:6443/healthz;
     proxy_ssl_trusted_certificate /var/lib/kubernetes/ca.pem;
  }
}
EOF
```

Set up the configuration:

```bash
sudo mv kubernetes.default.svc.cluster.local /etc/nginx/sites-available/kubernetes.default.svc.cluster.local
sudo ln -s /etc/nginx/sites-available/kubernetes.default.svc.cluster.local /etc/nginx/sites-enabled/
sudo systemctl restart nginx
sudo systemctl enable nginx
```

You can verify it's working by:

```bash
curl -H "Host: kubernetes.default.svc.cluster.local" -i http://127.0.0.1/healthz
```

### Set up RBAC for Kubelet Authorization

RBAC is Role-Based Access Control and is the mechanism to create roles and assign permissions to different users.

We need to make sure that the Kubernetes API has permission to access the Kubelet API on each node and perform certain common tasks, without this some functionalities might not work.

So let's create a ClusterRose with the necessary permissions and assign that role to the Kubernetes user with a ClusterRoleBinding:

We only need to run this command on one of the servers, since now we are communicating to the Kubernetes cluster itself and the changes will be propagated.

```bash
cat << EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
```

And then, bind the role to the kubernetes user:

```bash
cat << EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF
```

### Setting up a Kube API Frontend Load Balancer

Log in into the API Load Balancer server and first, install nginx:

```bash
sudo apt-get install -y nginx
sudo systemctl enable nginx
sudo mkdir -p /etc/nginx/tcpconf.d
```

We need to include the following at the end of the nginx.conf:

```bash
sudo vi /etc/nginx/nginx.conf
```

```
include /etc/nginx/tcpconf.d/*;
```

That way, we cna add our configuration files in this folder and nginx will pick it up.

For the next step, we can set the following environment variables to help us:

```bash
CONTROLLER0_IP=10.0.1.73
CONTROLLER1_IP=10.0.1.35
```

And now we create the load balancer nginx config file:

```bash
cat << EOF | sudo tee /etc/nginx/tcpconf.d/kubernetes.conf
stream {
    upstream kubernetes {
        server $CONTROLLER0_IP:6443;
        server $CONTROLLER1_IP:6443;
    }

    server {
        listen 6443;
        listen 443; # not really necessary
        proxy_pass kubernetes;
    }
}
EOF
```

Then, we reload the nginx configuration:

```bash
sudo nginx -s reload
```

And we can verify it's working by running:

```bash
curl -k https://localhost:6443/version
```

(The `-k` is to make sure we don't have problems with certificates since they were just generated)

#### Resources

[Kubernetes components](https://kubernetes.io/docs/concepts/overview/components/#master-components)