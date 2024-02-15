
##### Documentation Link:

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

##### Step 1: Setup containerd
```sh
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```
```sh
modprobe overlay
modprobe br_netfilter
```
```sh
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```
```sh
sysctl --system
```
```sh
apt-get install -y containerd
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
```
```sh
nano /etc/containerd/config.toml
```
  --> SystemdCgroup = true

```sh
systemctl restart containerd
```

##### Step 2: Kernel Parameter Configuration
```sh
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```
```sh
sudo sysctl --system
```

##### Step 3: Configuring Repo and Installation
```sh
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
```sh
sudo apt-get update
apt-cache madison kubeadm
sudo apt-get install -y kubelet=1.24.2-00 kubeadm=1.24.2-00 kubectl=1.24.2-00 cri-tools=1.24.2-00
sudo apt-mark hold kubelet kubeadm kubectl
```

#### Step 4: Initialize Cluster with kubeadm (Only master node)
```sh
kubeadm init --pod-network-cidr=10.244.0.0/16 --kubernetes-version=1.24.2
```
```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
##### Step 5: Install Network Addon (flannel) (master node)
```sh
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

##### Step 6: Untaint node
```sh
kubectl taint nodes --all node-role.kubernetes.io/master-
```
#### Location of Manifest files for K8s components:
```sh
cd /etc/kubernetes/manifests
```
