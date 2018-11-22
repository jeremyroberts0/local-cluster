# Instance Setup
## Master
[ ] Install Fedora Server
[ ] SSH with keys only
[ ] 

## Node

# Setup Steps
## SSH
Disable Password based login and require a public key.  Public key is copied from client to server.

While still on password based auth, Use ssh-copy-id to copy public key from client to server:

```sh
ssh-copy-id
```
(copies default ssh keys in home dir by default)

Change ssh config on server to:
```
PubkeyAuthentication yes
PasswordAuthentication no
```

## Kubernetes
From https://kubernetes.io/docs/getting-started-guides/fedora/fedora_manual_config/

Install Kubernetes with dnf
```sh
dnf -y install kubernetes
```

Install etcd
```
dnf -y install etcd
```

Set master in Kube config in `/etc/kubernetes/config`:
```
KUBE_MASTER="--master=http://master.chuperstein.local:8080"
```

Disable firewall (not present on default Fedora Server):
```
systemctl mask firewalld.service
systemctl stop firewalld.service

systemctl disable iptables.service
systemctl stop iptables.service
```

Setup kube API server in `/etc/kubernetes/apiserver`:
```
# The address on the local server to listen to.
KUBE_API_ADDRESS="--address=0.0.0.0"

# Comma separated list of nodes in the etcd cluster
KUBE_ETCD_SERVERS="--etcd-servers=http://127.0.0.1:2379"

# Address range to use for services
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"

# Add your own!
KUBE_API_ARGS=""
```

Edit `/etc/etcd/etcd.conf` to let etcd listen on all available IPs instead of 127.0.0.1. If you have not done this, you might see an error such as “connection refused”.
```
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
```

Start k8 services
```
systemctl restart etcd
systemctl enable etcd
systemctl status etcd
systemctl restart kube-apiserver
systemctl enable kube-apiserver
systemctl status kube-apiserver
systemctl restart kube-controller-manager
systemctl enable kube-controller-manager
systemctl status kube-controller-manager
systemctl restart kube-scheduler
systemctl enable kube-scheduler
systemctl status kube-scheduler
```