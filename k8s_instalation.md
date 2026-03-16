<img width="1373" height="215" alt="image" src="https://github.com/user-attachments/assets/fb2a1b41-35f8-4d72-937a-1b34331aabfd" /><img width="1377" height="214" alt="image" src="https://github.com/user-attachments/assets/f90a2772-517d-46c1-8e35-72a7a76d274c" /><img width="950" height="221" alt="image" src="https://github.com/user-attachments/assets/448f5c94-6b26-416a-b9d3-62892c89f18d" />---
layout: default
title: "Ansible desired state – controller configuration as code"
permalink: /ansible-desired-state.html
---

[← Back to index](/)


# Kubernetes Installation and Configuration Fundamentals

## Exploring the Kubernetes Architecture

<img width="1378" height="730" alt="image" src="https://github.com/user-attachments/assets/d91564c7-36db-4963-b692-3db3749e4c0c" />
<img width="1306" height="733" alt="image" src="https://github.com/user-attachments/assets/ea9d7cb0-d9c9-43fc-bbd4-94befc05160b" />
<img width="1363" height="737" alt="image" src="https://github.com/user-attachments/assets/af28b302-5f42-4333-9e38-cc65a404d257" />
<img width="1358" height="775" alt="image" src="https://github.com/user-attachments/assets/95395c90-199d-49d0-8046-72d315917543" />
<img width="1336" height="744" alt="image" src="https://github.com/user-attachments/assets/d4200ac2-5dc2-4b91-818b-7a4758628491" />
<img width="1381" height="779" alt="image" src="https://github.com/user-attachments/assets/71040039-c78e-41df-9269-5f15a5cb8f68" />
<img width="1373" height="760" alt="image" src="https://github.com/user-attachments/assets/d654ff0f-0ff4-4874-9b4f-6dcdd99c733d" />
<img width="1081" height="541" alt="image" src="https://github.com/user-attachments/assets/08e24ebb-d9ce-4d7e-9686-f2975213b715" />
<img width="1387" height="767" alt="image" src="https://github.com/user-attachments/assets/098dc01c-65c4-4ed7-8976-96cbe1774bc0" />
<img width="815" height="721" alt="image" src="https://github.com/user-attachments/assets/f302c91d-f5e3-42c4-91f0-e42c2338ad0b" />
<img width="1395" height="752" alt="image" src="https://github.com/user-attachments/assets/7d63fdc2-9813-4823-a70f-75ae121139bf" />
<img width="1287" height="731" alt="image" src="https://github.com/user-attachments/assets/98645b4b-9934-4ea9-97e9-57168f9e1ea0" />
<img width="1333" height="782" alt="image" src="https://github.com/user-attachments/assets/5ca8e7f7-ec6c-40da-a279-ce590162b624" />
<img width="1322" height="759" alt="image" src="https://github.com/user-attachments/assets/75cbd1fa-d776-4a68-9749-f999f444f1fc" />
<img width="1412" height="781" alt="image" src="https://github.com/user-attachments/assets/ee845641-4344-4dc1-aa14-ddf8f337b52d" />

## Installing and Configuring Kubernetes


```bash
apt-get install -y containerd

sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v_$VERSION/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v_$VERSION/deb/ /' | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list

apt-get update
apt-get install kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl containerd
```

```bash
#0 - Install Packages
#containerd prerequisites, and load two modules and configure them to load on boot
#https://kubernetes.io/docs/setup/production-environment/container-runtimes/
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

```bash
# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```
```bash
#Install containerd...
sudo apt-get install -y containerd

#Create a containerd configuration file
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```
```bash
# https://github.com/containerd/containerd/blob/master/docs/ops.md

#At the end of this section, change SystemdCgroup = false to SystemdCgroup = true
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
    ...
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true

#You can use sed to swap in true
sudo sed -i 's/            SystemdCgroup = false/            SystemdCgroup = true/' /etc/containerd/config.toml

#Verify the change was made
grep 'SystemdCgroup = true' /etc/containerd/config.toml

#Restart containerd with the new configuration
sudo systemctl restart containerd
```
```bash
#Install Kubernetes packages - kubeadm, kubelet and kubectl
#Add k8s.io's apt repository gpg key, this will likely change for each version of kubernetes release.
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

#Add the Kubernetes apt repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

#Update the package list and use apt-cache policy to inspect versions available in the repository
sudo apt-get update
apt-cache policy kubelet | head -n 20

#Install the required packages, if needed we can request a specific version.
#Use this version because in a later course we will upgrade the cluster to a newer version.
#Try to pick one version back because later in this series, we'll run an upgrade
VERSION=1.29.1-1.1
sudo apt-get install -y kubelet=$VERSION kubeadm=$VERSION kubectl=$VERSION
sudo apt-mark hold kubelet kubeadm kubectl containerd

#To install the latest, omit the version parameters. I have tested all demos with the version above, if you use the latest it ...
#sudo apt-get install kubelet kubeadm kubectl
#sudo apt-mark hold kubelet kubeadm kubectl containerd

#1 - systemd Units
#Check the status of our kubelet and our container runtime, containerd.
#The kubelet will enter a inactive (dead) state until a cluster is created or the node is joined to an existing cluster.
sudo systemctl status kubelet.service
sudo systemctl status containerd.service
```
<img width="1368" height="761" alt="image" src="https://github.com/user-attachments/assets/e30ac96e-520c-4ce3-9961-1ccd3715312f" />
<img width="1422" height="764" alt="image" src="https://github.com/user-attachments/assets/79fa2956-abc1-4419-bdd4-13e5c9a7c0eb" />
<img width="1350" height="759" alt="image" src="https://github.com/user-attachments/assets/ec6d8584-e5f6-4ef8-9849-cb30263e9916" />
<img width="1379" height="765" alt="image" src="https://github.com/user-attachments/assets/0dc35474-d181-4677-8fc5-15800c78cdcf" />
<img width="1394" height="730" alt="image" src="https://github.com/user-attachments/assets/4ecde67a-aae5-4d73-9116-8532bbe52219" />

## Creating control plane:
<img width="1382" height="722" alt="image" src="https://github.com/user-attachments/assets/819d6106-d86c-4efa-9f19-42dedb1cab28" />

```
bash
wget https://raw.githubusercontent.com/projectcalico/calico/master/manifests/calico.yaml

sudo kubeadm init

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl apply -f calico.yaml
kubeadm join 172.16.94.10:6443 \
  --token i0pr88.pbid2af0071xhuo1 \
  --discovery-token-ca-cert-hash \
  sha256:9a56f13bbae1f77e3a01fecc2bf8c59e6977d9c71c2d3482b988fa47767353d7
```

### demo:
```
#0 - Creating a Cluster
# Log into our control plane node
ssh aen@c1-cp1

#Create our kubernetes cluster, specify a pod network range matching that in calico.yaml!
#Only on the Control Plane Node, download the yaml files for the pod network.
wget https://raw.githubusercontent.com/projectcalico/calico/master/manifests/calico.yaml

#Look inside calico.yaml and find the setting for Pod Network IP address range CALICO_IPV4POOL_CIDR,
#adjust if needed for your infrastructure to ensure that the Pod network IP
#range doesn't overlap with other networks in our infrastructure.
vi calico.yaml

#You can now just use kubeadm init to bootstrap the cluster
sudo kubeadm init --kubernetes-version v1.29.1

#remove the kubernetes-version parameter if you want to use the latest.
#sudo kubeadm init

#Before moving on review the output of the cluster creation process including the kubeadm init phases,
#the admin.conf setup and the node join command

#Configure our account on the Control Plane Node to have admin access to the API server from a non-privileged account.
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

#1 - Creating a Pod Network
#Deploy yaml file for your pod network.
kubectl apply -f calico.yaml
```
```
#Look for the all the system pods and calico pods to change to Running.
#The DNS pod won't start (pending) until the Pod network is deployed and Running.
kubectl get pods --all-namespaces

#Gives you output over time, rather than repainting the screen on each iteration.
kubectl get pods --all-namespaces --watch
aen@c1-cp1:~$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS              RESTARTS   AGE
kube-system   calico-kube-controllers-57758d645c-59fj6   0/1     ContainerCreating   0          27s
kube-system   calico-node-fl9h7                          0/1     Running             0          27s
kube-system   coredns-76f75df574-pr547                   0/1     ContainerCreating   0          3m57s
kube-system   coredns-76f75df574-stc84                   0/1     ContainerCreating   0          3m57s
kube-system   etcd-c1-cp1                                1/1     Running             0          4m3s
kube-system   kube-apiserver-c1-cp1                      1/1     Running             0          4m3s
kube-system   kube-controller-manager-c1-cp1             1/1     Running             0          4m3s
kube-system   kube-proxy-wvtm8                           1/1     Running             0          3m57s
kube-system   kube-scheduler-c1-cp1                      1/1     Running             0          4m3s
aen@c1-cp1:~$

#Gives you output over time, rather than repainting the screen on each iteration.
kubectl get pods --all-namespaces --watch

#All system pods should be Running
kubectl get pods --all-namespaces

#Get a list of our current nodes, just the Control Plane Node Node...should be Ready.
kubectl get nodes
```

```
#2 - systemd Units...again!
#Check out the systemd unit...it's no longer inactive (dead)...its active(running) because it has static pods to start
#Remember the kubelet starts the static pods, and thus the control plane pods
sudo systemctl status kubelet.service

#3 - Static Pod manifests
#Let's check out the static pod manifests on the Control Plane Node
ls /etc/kubernetes/manifests

#And look more closely at API server and etcd's manifest.
sudo more /etc/kubernetes/manifests/etcd.yaml
sudo more /etc/kubernetes/manifests/kube-apiserver.yaml
#Check out the directory where the kubeconfig files live for each of the control plane pods.
ls /etc/kubernetes
```

### Demo: adding nodes to claster
```
#For this demo ssh into c1-node1
ssh aen@c1-node1

#Disable swap, swapoff then edit your fstab removing any entry for swap partitions
#You can recover the space with fdisk. You may want to reboot to ensure your config is ok.
swapoff -a
vi /etc/fstab

#0 - Joining Nodes to a Cluster
#Install a container runtime - containerd
#containerd prerequisites, and load two modules and configure them to load on boot
#https://kubernetes.io/docs/setup/production-environment/container-runtimes/
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
# Apply sysctl params without reboot
sudo sysctl --system

#Install containerd...
sudo apt-get install -y containerd

#Configure containerd
sudo mkdir -p /etc/containerd

#Configure containerd
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml

# https://github.com/containerd/containerd/blob/master/docs/ops.md

#At the end of this section, change SystemdCgroup = false to SystemdCgroup = true
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
    ...
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true

#You can use sed to swap in true
sudo sed -i 's/            SystemdCgroup = false/            SystemdCgroup = true/' /etc/containerd/config.toml

#Verify the change was made
grep 'SystemdCgroup = true' /etc/containerd/config.toml

#Restart containerd with the new configuration
sudo systemctl restart containerd
#Install Kubernetes packages - kubeadm, kubelet and kubectl
#Add Google's apt repository gpg key
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

#Add the Kubernetes apt repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

#Update the package list and use apt-cache policy to inspect versions available in the repository
sudo apt-get update
apt-cache policy kubelet | head -n 20

#Install the required packages, if needed we can request a specific version.
#Use this version because in a later course we will upgrade the cluster to a newer version.
#Try to pick one version back because later in this series, we'll run an upgrade
VERSION=1.29.1-1.1
#Install the required packages, if needed we can request a specific version.
#Use this version because in a later course we will upgrade the cluster to a newer version.
#Try to pick one version back because later in this series, we'll run an upgrade
VERSION=1.29.1-1.1
sudo apt-get install -y kubelet=$VERSION kubeadm=$VERSION kubectl=$VERSION
sudo apt-mark hold kubelet kubeadm kubectl containerd

#To install the latest, omit the version parameters
#sudo apt-get install kubelet kubeadm kubectl
#sudo apt-mark hold kubelet kubeadm kubectl

#Check the status of our kubelet and our container runtime.
#The kubelet will enter a inactive/dead state until it's joined
sudo systemctl status kubelet.service
sudo systemctl status containerd.service

#You can also use print-join-command to generate token and print the join command in the proper format
#COPY THIS INTO YOUR CLIPBOARD
kubeadm token create --print-join-command

#Back on the worker node c1-node1, using the Control Plane Node (API Server) IP address or name, the token and the cert has, ...
ssh aen@c1-node1
#PASTE_JOIN_COMMAND_HERE be sure to add sudo
sudo kubeadm join 172.16.94.10:6443 \
  --token km25f3.q19pwleiisqzkhoo \
  --discovery-token-ca-cert-hash sha256:f1c5a8de4018095eb938b189ef6d687b8e68e55db59ef7acdcf12c71a8fa5adf

#Log out of c1-node1 and back on to c1-cp1
exit
#Back on Control Plane Node, this will say NotReady until the networking pod is created on the new node.
#Has to schedule the pod, then pull the container.
kubectl get nodes

#On the Control Plane Node, watch for the calico pod and the kube-proxy to change to Running on the newly added nodes.
kubectl get pods --all-namespaces --watch
```
<img width="1406" height="742" alt="image" src="https://github.com/user-attachments/assets/9d6d1291-4dd2-4784-8311-57b70199481d" />

```bash
# This demo will be run from c1-cp1 since kubectl is already installed there.
# This can be run from any system that has the Azure CLI client installed.

#Ensure Azure CLI command line utilitles are installed
#https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-apt?view=azure-cli-latest
AZ_REPO=$(lsb_release -cs)
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | sudo tee /etc/apt/sources.list.d/azure-cli.list

#Install the gpg key for Microsoft's repository
curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null
#Install the gpg key for Microsoft's repository
curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null

sudo apt-get update
sudo apt-get install azure-cli

#Log into our subscription
#Free account - https://azure.microsoft.com/en-us/free/
az login
az account set --subscription "Demonstration Account"

#Create a resource group for the serivces we're going to create
az group create --name "Kubernetes-Cloud" --location centralus

#Let's get a list of the versions available to us
az aks get-versions --location centralus -o table

#Let's create our AKS managed cluster. Use --kubernetes-version to specify a version.
az aks create \
  --resource-group "Kubernetes-Cloud" \
  --generate-ssh-keys \
  --name CSCluster \
  --node-count 3  #default Node count is 3

#If needed, we can download and install kubectl on our local system.
az aks install-cli

#Get our cluster credentials and merge the configuration into our existing config file.
#This will allow us to connect to this system remotely using certificate based user authentication.
az aks get-credentials --resource-group "Kubernetes-Cloud" --name CSCluster

#Get our cluster credentials and merge the configuration into our existing config file.
#This will allow us to connect to this system remotely using certificate based user authentication.
az aks get-credentials --resource-group "Kubernetes-Cloud" --name CSCluster

#List our currently available contexts
kubectl config get-contexts

#set our current context to the Azure context
kubectl config use-context CSCluster

#run a command to communicate with our cluster.
kubectl get nodes

#Get a list of running pods, we'll look at the system pods since we don't have anything running.
#Since the API Server is HTTP based...we can operate our cluster over the internet...esentially the same as if it wa...
kubectl get pods --all-namespaces

#Let's set to the kubectl context back to our local custer
kubectl config use-context kubernetes-admin@kubernetes
```

## Certified Kubernetes Administrator: Using kubeadm to Install a Basic Cluster

package installation container d"
```
#Setup 
#   1. 4 VMs Ubuntu 22.04, 1 control plane, 3 nodes.
#   2. Static IPs on individual VMs
#   3. /etc/hosts hosts file includes name to IP mappings for VMs
#   4. Swap is disabled
#   5. Take snapshots prior to installation, this way you can install 
#       and revert to snapshot if needed
#
ssh aen@c1-cp1


#0 - Disable swap, swapoff then edit your fstab removing any entry for swap partitions
#You can recover the space with fdisk. You may want to reboot to ensure your config is ok. 
sudo swapoff -a
vi /etc/fstab


###IMPORTANT####
#I will keep the code in the course downloads up to date with the latest method.
################


#0 - Install Packages 
#containerd prerequisites, and load two modules and configure them to load on boot
#https://kubernetes.io/docs/setup/production-environment/container-runtimes/
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF


# Apply sysctl params without reboot
sudo sysctl --system


#Install containerd...
sudo apt-get install -y containerd


#Create a containerd configuration file
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml


#Set the cgroup driver for containerd to systemd which is required for the kubelet.
#For more information on this config file see:
# https://github.com/containerd/cri/blob/master/docs/config.md and also
# https://github.com/containerd/containerd/blob/master/docs/ops.md

#At the end of this section, change SystemdCgroup = false to SystemdCgroup = true
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
        ...
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true

#You can use sed to swap in true
sudo sed -i 's/            SystemdCgroup = false/            SystemdCgroup = true/' /etc/containerd/config.toml


#Verify the change was made
grep 'SystemdCgroup = true' /etc/containerd/config.toml


#Restart containerd with the new configuration
sudo systemctl restart containerd




#Install Kubernetes packages - kubeadm, kubelet and kubectl
#Add k8s.io's apt repository gpg key, this will likely change for each version of kubernetes release. 
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg


#Add the Kubernetes apt repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list


#Update the package list and use apt-cache policy to inspect versions available in the repository
sudo apt-get update
apt-cache policy kubelet | head -n 20 


#Install the required packages, if needed we can request a specific version. 
#Use this version because in a later course we will upgrade the cluster to a newer version.
#Try to pick one version back because later in this series, we'll run an upgrade
VERSION=1.29.1-1.1
sudo apt-get install -y kubelet=$VERSION kubeadm=$VERSION kubectl=$VERSION 
sudo apt-mark hold kubelet kubeadm kubectl containerd


#To install the latest, omit the version parameters. I have tested all demos with the version above, if you use the latest it may impact other demos in this course and upcoming courses in the series
#sudo apt-get install kubelet kubeadm kubectl
#sudo apt-mark hold kubelet kubeadm kubectl containerd


#1 - systemd Units
#Check the status of our kubelet and our container runtime, containerd.
#The kubelet will enter a inactive (dead) state until a cluster is created or the node is joined to an existing cluster.
sudo systemctl status kubelet.service
sudo systemctl status containerd.service
```
create control plane:
```
#0 - Creating a Cluster
# Log into our control plane node
ssh aen@c1-cp1


#Create our kubernetes cluster, specify a pod network range matching that in calico.yaml! 
#Only on the Control Plane Node, download the yaml files for the pod network.
wget https://raw.githubusercontent.com/projectcalico/calico/master/manifests/calico.yaml


#Look inside calico.yaml and find the setting for Pod Network IP address range CALICO_IPV4POOL_CIDR, 
#adjust if needed for your infrastructure to ensure that the Pod network IP
#range doesn't overlap with other networks in our infrastructure.
vi calico.yaml


#You can now just use kubeadm init to bootstrap the cluster
sudo kubeadm init --kubernetes-version v1.29.1


#remove the kubernetes-version parameter if you want to use the latest.
#sudo kubeadm init


#Before moving on review the output of the cluster creation process including the kubeadm init phases, 
#the admin.conf setup and the node join command


#Configure our account on the Control Plane Node to have admin access to the API server from a non-privileged account.
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


#1 - Creating a Pod Network
#Deploy yaml file for your pod network.
kubectl apply -f calico.yaml


#Look for the all the system pods and calico pods to change to Running. 
#The DNS pod won't start (pending) until the Pod network is deployed and Running.
kubectl get pods --all-namespaces


#Gives you output over time, rather than repainting the screen on each iteration.
kubectl get pods --all-namespaces --watch


#All system pods should be Running
kubectl get pods --all-namespaces


#Get a list of our current nodes, just the Control Plane Node Node...should be Ready.
kubectl get nodes 




#2 - systemd Units...again!
#Check out the systemd unit...it's no longer inactive (dead)...its active(running) because it has static pods to start
#Remember the kubelet starts the static pods, and thus the control plane pods
sudo systemctl status kubelet.service 


#3 - Static Pod manifests
#Let's check out the static pod manifests on the Control Plane Node
ls /etc/kubernetes/manifests


#And look more closely at API server and etcd's manifest.
sudo more /etc/kubernetes/manifests/etcd.yaml
sudo more /etc/kubernetes/manifests/kube-apiserver.yaml


#Check out the directory where the kubeconfig files live for each of the control plane pods.
ls /etc/kubernetes
```
creat nodes:
```
#For this demo ssh into c1-node1
ssh aen@c1-node1


#Disable swap, swapoff then edit your fstab removing any entry for swap partitions
#You can recover the space with fdisk. You may want to reboot to ensure your config is ok. 
swapoff -a
vi /etc/fstab


#0 - Joining Nodes to a Cluster

#Install a container runtime - containerd
#containerd prerequisites, and load two modules and configure them to load on boot
#https://kubernetes.io/docs/setup/production-environment/container-runtimes/
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system


#Install containerd...
sudo apt-get install -y containerd


#Configure containerd
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml


#Set the cgroup driver for containerd to systemd which is required for the kubelet.
#For more information on this config file see:
# https://github.com/containerd/cri/blob/master/docs/config.md and also
# https://github.com/containerd/containerd/blob/master/docs/ops.md

#At the end of this section, change SystemdCgroup = false to SystemdCgroup = true
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
        ...
#          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true

#You can use sed to swap in true
sudo sed -i 's/            SystemdCgroup = false/            SystemdCgroup = true/' /etc/containerd/config.toml


#Verify the change was made
grep 'SystemdCgroup = true' /etc/containerd/config.toml


#Restart containerd with the new configuration
sudo systemctl restart containerd



#Install Kubernetes packages - kubeadm, kubelet and kubectl
#Add Google's apt repository gpg key
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg


#Add the Kubernetes apt repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list


#Update the package list and use apt-cache policy to inspect versions available in the repository
sudo apt-get update
apt-cache policy kubelet | head -n 20 


#Install the required packages, if needed we can request a specific version. 
#Use this version because in a later course we will upgrade the cluster to a newer version.
#Try to pick one version back because later in this series, we'll run an upgrade
VERSION=1.29.1-1.1
sudo apt-get install -y kubelet=$VERSION kubeadm=$VERSION kubectl=$VERSION 
sudo apt-mark hold kubelet kubeadm kubectl containerd



#To install the latest, omit the version parameters
#sudo apt-get install kubelet kubeadm kubectl
#sudo apt-mark hold kubelet kubeadm kubectl


#Check the status of our kubelet and our container runtime.
#The kubelet will enter a inactive/dead state until it's joined
sudo systemctl status kubelet.service 
sudo systemctl status containerd.service 


#Log out of c1-node1 and back on to c1-cp1
exit



#You can also use print-join-command to generate token and print the join command in the proper format
#COPY THIS INTO YOUR CLIPBOARD
kubeadm token create --print-join-command


#Back on the worker node c1-node1, using the Control Plane Node (API Server) IP address or name, the token and the cert has, let's join this Node to our cluster.
ssh aen@c1-node1


#PASTE_JOIN_COMMAND_HERE be sure to add sudo
sudo kubeadm join 172.16.94.10:6443 \
  --token yn8tkx.f5ssw0qn1ycqskt2 \
  --discovery-token-ca-cert-hash sha256:66ff307c46617ca400060e54b0db58f1597419f0a54bd971ed074b1a12067ee0 

#Log out of c1-node1 and back on to c1-cp1
exit


#Back on Control Plane Node, this will say NotReady until the networking pod is created on the new node. 
#Has to schedule the pod, then pull the container.
kubectl get nodes 


#On the Control Plane Node, watch for the calico pod and the kube-proxy to change to Running on the newly added nodes.
kubectl get pods --all-namespaces --watch


#Still on the Control Plane Node, look for this added node's status as ready.
kubectl get nodes


#GO BACK TO THE TOP AND DO THE SAME FOR c1-node2 and c1-node3
#Just SSH into c1-node2 and c1-node3 and run the commands again.

```


