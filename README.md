# How to Setting up Kubernetes Cluster on Ubuntu 22.04

Are you seeking a straightforward tutorial to set up a Kubernetes Cluster on Ubuntu 22.04 (Jammy Jellyfish)?

This step-by-step guide will walk you through the process of installing a Kubernetes cluster on Ubuntu 22.04 using the Kubeadm command.

Kubernetes has become the leading container orchestration platform, enabling developers and system administrators to effortlessly manage and scale containerized applications. If you're an Ubuntu 22.04 user eager to leverage the capabilities of Kubernetes, you've found the perfect resource.

### Prerequisites
In this guide, we are using one master node and two worker nodes.  

### Lab Setup  
Master Node:  192.168.100.97  
Worker Node1:  192.168.100.98  
Worker Node2:  192.168.100.99  

### 1) Disable swap & Add kernel Parameters  

```
sudo swapoff -a
```  
```
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```  

**Load the following kernel modules on all the nodes**  
```
sudo tee /etc/modules-load.d/containerd.conf <<EOF  
overlay  
br_netfilter  
EOF
```  
$ sudo modprobe overlay  
$ sudo modprobe br_netfilter  

**Set the following Kernel parameters for Kubernetes**   

$ sudo nano /etc/sysctl.d/kubernetes.conf  
net.bridge.bridge-nf-call-ip6tables = 1  
net.bridge.bridge-nf-call-iptables = 1  
net.ipv4.ip_forward = 1  

**then save and exit**  

**Reload the above changes**  

$ sudo sysctl --system  

### 2) Install Containerd Runtime  

**In this guide, we are using containerd runtime for our Kubernetes cluster. So, to install containerd, first install its dependencies**  

$ sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates  

**Enable docker repository**  

$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg  

$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"  

**Now, run following apt command to install containerd**  

$ sudo apt update
$ sudo apt install -y containerd.io  

**Configure containerd so that it starts using systemd as cgroup.**  

$ containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1  
$ sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml  

**Restart and enable containerd service**  

$ sudo systemctl restart containerd  
$ sudo systemctl enable containerd  

### 3) Add Apt Repository for Kubernetes  

$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/kubernetes-xenial.gpg  
$ sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"  

### 4) Install Kubectl, Kubeadm and Kubelet  
**Post adding the repositories, install Kubernetes components like kubectl, kubelet and Kubeadm utility on all the nodes. Execute following set of commands**  

$ sudo apt update  
$ sudo apt install -y kubelet kubeadm kubectl  
$ sudo apt-mark hold kubelet kubeadm kubectl  

### 5) Initialize Kubernetes Cluster with Kubeadm  
**Now, we are all set to initialize Kubernetes cluster. Run the following Kubeadm command on the master node only.**  

$ sudo kubeadm init --control-plane-endpoint=192.168.100.97  

**After the initialization is complete, you will see a message with instructions on how to join worker nodes to the cluster. Make a note of the kubeadm join command for future reference.**  

**So, to start interacting with cluster, run following commands on the master node**  

$ mkdir -p $HOME/.kube  
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config  

**next, try to run following kubectl commands to view cluster and node status** 

$ kubectl cluster-info  
$ kubectl get nodes  

### 6) Join Worker Nodes to the Cluster  
**On each worker node, use the kubeadm join command you noted down earlier after initializing the master node on step 6. It should look something like this:**  
kubeadm join 192.168.100.97:6443 --token paln2p.cffblq6zz1myjnd0 \  
--discovery-token-ca-cert-hash sha256:55e72b90eb18d20e170ea8e4a820770aec7b76bf911ce4670747dc09a0b2dd05  

### 7) Install Calico Network Plugin
**Above output from worker nodes confirms that both the nodes have joined the cluster.Check the nodes status from master node using kubectl command**  

$ kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml  

**Verify the status of pods in kube-system namespace**  

$ kubectl get pods -n kube-system  

**Perfect, check the nodes status as well.**

$ kubectl get nodes  


### 8) Test Your Kubernetes Cluster Installation  
**To test Kubernetes installation, let’s try to deploy nginx based application and try to access it.**  

$ kubectl create deployment nginx-app --image=nginx --replicas=2  

Check the status of nginx-app deployment  

$ kubectl get deployment nginx-app  


## Congratulations! You have successfully set up a Kubernetes cluster on Ubuntu 22.04.










