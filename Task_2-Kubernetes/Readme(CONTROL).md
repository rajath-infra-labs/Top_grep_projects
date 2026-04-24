# 🚀 Kubernetes Control Plane Setup (Kubeadm)

This document provides a clean, production-style setup guide for initializing a Kubernetes control plane using kubeadm.

---

# 🧾 Environment Details

| Component         | Value            |
| ----------------- | ---------------- |
| OS                | Ubuntu 24.04 LTS |
| Kubernetes        | v1.30.x          |
| Container Runtime | containerd       |
| CNI               | Flannel          |
| Control IP        | 151.185.43.141   |
| Worker IP         | 151.185.43.142   |

---

# ⚙️ Step 1: System Preparation

## Disable Swap

```
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab
```

## Load Kernel Modules

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

## Apply sysctl settings

```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```

---

# 📦 Step 2: Install Container Runtime (containerd)

```
sudo apt update
sudo apt install -y containerd

sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

### ⚠️ IMPORTANT: Enable SystemdCgroup

Edit:

```
sudo nano /etc/containerd/config.toml
```

Change:

```
SystemdCgroup = true
```

Restart:

```
sudo systemctl restart containerd
sudo systemctl enable containerd
```

---

# ☸️ Step 3: Install Kubernetes Components

```
sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg


echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | \
sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable kubelet
```

---

# 🌐 Step 4: Host Configuration

## Set Hostname

```
sudo hostnamectl set-hostname k8s-control
exec bash
```

## Update /etc/hosts

```
sudo nano /etc/hosts
```

Ensure:

```
127.0.0.1 localhost
127.0.1.1 k8s-control

151.185.43.141   k8s-control
151.185.43.142   k8s-worker
```

---

# 🔧 Step 5: Install Helm & Helmfile

## Helm

```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

## Helmfile

```
wget https://github.com/helmfile/helmfile/releases/download/v0.162.0/helmfile_0.162.0_linux_amd64.tar.gz

tar -xzf helmfile_0.162.0_linux_amd64.tar.gz
sudo mv helmfile /usr/local/bin/
```

Verify:

```
helm version
helmfile --version
```

---

# 🔥 Step 6: Initialize Kubernetes Control Plane

```
sudo kubeadm init \
  --apiserver-advertise-address=151.185.43.141 \
  --pod-network-cidr=10.244.0.0/16
```

---

# ⚙️ Step 7: Configure kubectl

```
export KUBECONFIG=/etc/kubernetes/admin.conf
```

Verify:

```
kubectl get nodes
```

Expected:

```
NotReady (before CNI)
```

---

# 🌐 Step 8: Install CNI (Flannel)

```
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

---

# ✅ Step 9: Verify Cluster

```
kubectl get pods -n kube-system
```

All should be:

```
Running
```

Then:

```
kubectl get nodes
```

Expected:

```
k8s-control   Ready
```

---

# 🔑 Step 10: Worker Join Command

Example:

```
kubeadm join 151.185.43.141:6443 --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

---

# 🧠 Notes / Best Practices

* Always disable swap
* Ensure `SystemdCgroup=true`
* Use public IP if no VPC
* Keep security groups strict (no 0.0.0.0/0 except 80/443 if needed)
* Take snapshot before init

---

# 📌 Version Update Guide

| Component  | Current | Change To (Future) |
| ---------- | ------- | ------------------ |
| Kubernetes | v1.30   | Update repo URL    |
| Helmfile   | 0.162   | Update release URL |
| Flannel    | Latest  | Check GitHub repo  |

---

# 🎯 Outcome

✔ Control plane initialized
✔ Networking configured
✔ Cluster ready for worker join

---

# 🚀 Next Steps

* Setup Worker Node
* Join cluster
* Install monitoring stack (Prometheus,

