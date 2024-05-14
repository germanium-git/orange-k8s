# Install K8s cluster

Follow the instructions from https://github.com/kunchalavikram1427/YouTube_Series/blob/main/Kubernetes/ClusterSetup/Kubernetes_on_aws_with_containerd.md

All steps are executed as root

```
sudo su -
```

### Disable Swap

```
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### Install Containerd

```
wget https://github.com/containerd/containerd/releases/download/v1.6.22/containerd-1.6.22-linux-arm64.tar.gz
tar Cxzvf /usr/local containerd-1.6.22-linux-arm64.tar.gz
rm containerd-1.6.22-linux-arm64.tar.gz
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
mkdir -p /usr/local/lib/systemd/system
mv containerd.service /usr/local/lib/systemd/system/containerd.service
systemctl daemon-reload
systemctl enable --now containerd
```

### Configure Containerd

Copy the config.toml from other nodes

```
scp -i ~/.ssh/orangePiMac petr@172.31.1.50:/etc/containerd/config.toml ./
scp -i ~/.ssh/orangePiMac config.toml petr@172.31.1.52:/home/petr
```
Create & overwrite the config file with the one copied from other nodes

```
sudo mkdir /etc/containerd/
containerd config default > /etc/containerd/config.toml
cp config.toml /etc/containerd/
```

Restart the containred service
```
sudo systemctl restart containerd.service
```


### Install Runc

```
wget https://github.com/opencontainers/runc/releases/download/v1.1.8/runc.arm64
install -m 755 runc.arm64 /usr/local/sbin/runc
rm runc.arm64
```

### Install CNI

```
wget https://github.com/containernetworking/plugins/releases/download/v1.3.0/cni-plugins-linux-arm64-v1.3.0.tgz
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-arm64-v1.3.0.tgz
rm cni-plugins-linux-arm64-v1.3.0.tgz
```

### Install CRICTL

```
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.26.0/crictl-v1.26.0-linux-arm64.tar.gz
tar zxvf crictl-v1.26.0-linux-arm64.tar.gz -C /usr/local/bin
rm -f crictl-v1.26.0-linux-arm64.tar.gz

cat <<EOF | sudo tee /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 2
debug: false
pull-image-on-create: false
EOF
```

### Forwarding IPv4 and letting iptables see bridged traffic

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sysctl --system

# verify
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward

modprobe br_netfilter
sysctl -p /etc/sysctl.conf
```

### Fix the issue with resolv.conf

There might be an issue indicated by the following error message: "Failed create pod sandbox: open /run/systemd/resolve/resolv.conf: no such file or directory"

https://github.com/ivanfioravanti/kubernetes-the-hard-way-on-azure/issues/30

```
mkdir -p /run/systemd/resolve
ln -s /etc/resolv.conf /run/systemd/resolve/resolv.conf
```

### Install containernetworking-plugins

```
apt install containernetworking-plugins
```



### Install kubectl, kubelet and kubeadm
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add

apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
apt update

apt install -y kubeadm=1.26.7-00 kubelet=1.26.7-00 kubectl=1.26.7-00
```


### !!! Missing spin up master node

```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```


### Join the worker node to the cluster

Run the command on master and use the output printed out in terminal on a worker node.
```
kubeadm token create --print-join-command --ttl=0
```

Take the output from the previous command and run it on the worker node.
```
kubeadm join 172.31.1.50:6443 --token 1wswqs.a7m37sq9ppir5xkx --discovery-token-ca-cert-hash sha256:xxxxxxxxxx
```


### Check if all containers are running

Restart the whole node and after the reboot
- disable swap
- configure the symlink for the resolv.conf as shown above again (the directory /run/systemd/resolve is removed after each reboot).

Check if teh containers are still rebooted
```
sudo crictl ps -a
```

Investigate the logs from the kubelet service

```
journalctl -u kubelet -n 100
```
