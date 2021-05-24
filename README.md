# Piep-Kubernetes
## Installation
### Kubernetes Setup
#### containerd:
```
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
> overlay
> br_netfilter
> EOF
```
```
sudo modprobe overlay
sudo modprobe br_netfilter
```

*# Setup required sysctl params, these persist across reboots.*
```
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
> net.bridge.bridge-nf-call-iptables  = 1
> net.ipv4.ip_forward                 = 1
> net.bridge.bridge-nf-call-ip6tables = 1
> EOF
```


*# Apply sysctl params without reboot*
```
sudo sysctl --system
```
#### Install containerd
```
sudo apt install containerd  
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
```
#### Use *systemd* for cgroup driver in /etc/containerd/config.toml
```
sudo nano /etc/containerd/config.toml

[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```
```
sudo systemctl restart containerd
```

### Installing kubeadm, kubelet and kubectl
#### Debian-based distributions
*# Update the apt package index and install packages needed to use the Kubernetes apt repository: *
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

*# Download the Google Cloud public signing key:*
```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

*# Add the Kubernetes apt repository:*
```
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

*# Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:*
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### Setup Configuration File
#### Create configuration file *config.yaml*

```
sudo nano config.yaml
```
```
apiVersion : kubelet.config.k8s.io/v1beta1
kind : KubeletConfiguration
cgroupDriver : systemd
---
apiVersion : kubeadm.k8s.io/v1beta2
kind : ClusterConfiguration
networking :
  podSubnet : "192.168.0.0/16"
```
  


