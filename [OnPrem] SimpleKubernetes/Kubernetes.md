# Setup Kubernetes Cluster

1.
```Bash
sudo apt -y update && sudo apt -y upgrade
```

2. Disable swap memory on all nodes
```Bash
sudo swapoff -a
```

3. Load the required kernel modules and configure the necessary network parameters
```Bash
sudo modprobe overlay
sudo modprobe br_netfilter
echo -e "overlay\nbr_netfilter" | sudo tee /etc/modules-load.d/k8s.conf

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
sudo sysctl --system
```

4. Install the container runtime
```Bash
sudo apt -y install containerd
```

5. COnfigure the runtime
```Bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

6. Install the required Kubernetes components
```Bash
sudo apt install -y apt-transport-https ca-certificates curl
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

7. Initialize the master node
```Bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

8. Set up the local kubeconfig for kubectl use
```Bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

9. Join the worker nodes to the newly initialized Kubernetes cluster
```Bash

```

10. Deploy a network add-on
```Bash
kubectl apply -f <network_addon_manifest_url>
```

