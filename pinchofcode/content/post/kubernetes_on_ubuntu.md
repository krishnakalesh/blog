+++
title = 'Installing Kubernetes on Ubuntu: A Step-by-Step Guide 🛳'
date = 2024-03-12T17:23:31+05:30
draft = false
+++
# Installing Kubernetes on Ubuntu: A Step-by-Step Guide 🛳

Kubernetes has become the de facto standard for container orchestration, enabling organizations to deploy, scale, and manage containerized applications with ease. In this guide, we'll walk through the process of installing Kubernetes on Ubuntu, one of the most popular Linux distributions.
<!--more-->
## Prerequisites
Before we begin, ensure that you have the following prerequisites:

1. Ubuntu Linux distribution (version 18.04 or later recommended) This Example use - UBUNTU 22.04
2. A minimum of 2GB of RAM per node (4GB or more recommended)
3. A reliable internet connection

## Set up hostnames
Run below scripts as root user
```shell
sudo hostnamectl set-hostname "k8s-appmaster.xyzcorp"
```
Add below line to /etc/hosts

	{VM 1 IP 1} k8s-worker1.xyzcorp
	{VM 2 IP 1} k8s-appmaster.xyzcorp

In worker nodes do same as below,
```shell
sudo hostnamectl set-hostname "k8s-worker1.xyzcorp"
```
## Below steps to enable some network support for Kubernetes.
Run below scripts as root user

```shell	
#Kernel module Overlays 
printf "overlay\nbr_netfilter\n" >> /etc/modules-load.d/containerd.conf

sudo modprobe overlay
sudo modprobe br_netfilter

#IP Table Configurations 
printf "net.bridge.bridge-nf-call-iptables = 1\nnet.ipv4.ip_forward = 1\nnet.bridge.bridge-nf-call-ip6tables = 1\n" >> /etc/sysctl.d/99-kubernetes-cri.conf

sysctl --system
```
## Install Container run time. 

Kubernetes prefer containerd. so used that instead of Docker.

Run below scripts as root user

Here, we download the containerd tar.gz and enable containerd as a service.
Also, setting default configuration for containerd as well.

```shell
wget https://github.com/containerd/containerd/releases/download/v1.6.16/containerd-1.6.16-linux-amd64.tar.gz -P /tmp/
tar Cxzvf /usr/local /tmp/containerd-1.6.16-linux-amd64.tar.gz
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service -P /etc/systemd/system/
systemctl daemon-reload
systemctl enable --now containerd

wget https://github.com/opencontainers/runc/releases/download/v1.1.4/runc.amd64 -P /tmp/
install -m 755 /tmp/runc.amd64 /usr/local/sbin/runc

wget https://github.com/containernetworking/plugins/releases/download/v1.2.0/cni-plugins-linux-amd64-v1.2.0.tgz -P /tmp/
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin /tmp/cni-plugins-linux-amd64-v1.2.0.tgz

mkdir -p /etc/containerd
containerd config default | tee /etc/containerd/config.toml   <<<<<<<<<<< manually edit and change systemdCgroup to true
systemctl restart containerd
```

## Make Swap memory OFF
Run below scripts as root user

```shell
sudo -i
swapoff -a
exit	
free -h
#(swapoff -a  <<<<<<<<  disable it in /etc/fstab instead by commenting out the line mentioned swap)
```
## Download Kubernetes components.
Run below scripts as root user

Download Kubernetes repository and reboot the VM

```shell
    apt-get update
	apt-get install -y apt-transport-https ca-certificates curl
	
	*curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
	*echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list

    apt-get update

    reboot
```

## Install Kubernetes binaries (kubelet, kubeadm,kubectl ).
Run below scripts as root user

It is better to control the version manually and mark hold of the version.

```shell
#select version manually 
	apt-get install -y kubelet=1.26.9-00 kubeadm=1.26.9-00 kubectl=1.26.9-00
	apt-mark hold kubelet kubeadm kubectl
```

## All the above steps we need to do in master as well as worker nodes.

## Master Node Only - Initialize kubeadm
```shell
sudo kubeadm init --pod-network-cidr 10.10.0.0/16 --kubernetes-version 1.26.9 --node-name k8s-appmaster.xyzcorp
```
Now exit from root user mode and login as normal user to VM and execute below commands
```shell	
	mkdir -p $HOME/.kube
	sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	sudo chown $(id -u):$(id -g) $HOME/.kube/config	
```

Installation can be verified using commands
```shell	
	$kubectl cluster-info
	$kubectl get nodes
	$kubectl get namespaces
	$kubectl get pods -n kube-system
```

## Master Node Only - Add CNI plugin

```shell	
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
wget https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml
vi custom-resources.yaml <<<<<< edit the CIDR for pods as it's custom - 10.10.0.0/16
kubectl apply -f custom-resources.yaml
```
Refer : https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart

## Remove the taints on the master node
Remove the taints on the master node so that you can schedule pods on it.

```shell
	kubectl taint nodes --all node-role.kubernetes.io/control-plane-
	kubectl taint nodes --all node-role.kubernetes.io/master-
```
## Generate token for worker nodes

When we execute below command in master node, it will give us the discovery token that
can be used by worker nodes to join control-plane.
```shell
kubeadm token create --print-join-command
```
## For worker nodes to join master

Use the command received from above step from worker node and then worker will
automatically join the cluster.

![Alt text](/pinchofcode/images/k81.png)

## Common Issues

**Below steps can be used to reset and install Kubernetes again**
```shell
kubectl get pods --all-namespaces
sudo systemctl disable kubelet
sudo systemctl stop kubelet  
sudo systemctl stop kube-proxy
sudo systemctl stop containerd
sudo systemctl status containerd 
kubectl delete configmap -n kube-system calico-config
kubectl delete networkpolicy --all -n kube-system  
sudo kubeadm reset -f  
sudo rm -rf /usr/bin/kubelet /usr/bin/kubeadm /usr/bin/kubectl  
sudo rm -rf /etc/kubernetes/
sudo rm -rf /var/lib/kubelet/
sudo rm -rf /var/lib/kube-proxy/
sudo rm -rf /var/lib/kubelet/
sudo rm -rf /opt/cni/bin/calico*
sudo rm -rf /etc/cni/net.d/10-calico.conflist
sudo rm -rf /etc/cni/net.d/calico-kubeconfig
sudo rm -rf /home/{USER_YOURS}/.kube/
sudo rm -rf /etc/systemd/system/kubelet.service.d
sudo rm -rf /var/lib/etcd
sudo rm -rf /var/log/kubernetes/
sudo rm -rf /var/log/kube-proxy/
sudo rm -f /etc/cni/net.d/calico-*
sudo apt-get purge kubeadm kubectl kubelet kubernetes-cni 
sudo apt-get autoremove  
sudo journalctl --vacuum-time=1d  
sudo reboot 
```
Once everything removed to re apply, we need to repeat steps from 
kubernetes installation to reinstate Kubernetes setup.

**Below steps can be used to remove a worker node from cluster**
```shell
  kubectl drain k8s-worker1.xyzcorp --ignore-daemonsets
  kubectl delete node k8s-worker1.xyzcorp
```
**Below steps can be used to reset kubeadm**

Once everything removed to re apply, we need to repeat steps from 
kubernetes installation to reinstate Kubernetes setup.

```shell
sudo systemctl stop kubelet
sudo kubeadm reset
sudo apt remove -y kubectl kubelet kubeadm
sudo apt autoremove -y
sudo reboot
chekc if any process running
ps aux | grep kube-
```
**Complete removal of containerd**

Once everything removed to re apply, we need to repeat steps from 
kubernetes installation to reinstate Kubernetes setup.
```shell
sudo systemctl disable containerd
sudo apt-get remove containerd.io
sudo apt autoremove
sudo apt-get purge containerd.io
sudo rm -rf /etc/containerd
sudo rm -rf /var/lib/containerd
sudo apt autoremove
sudo apt-get clean
dpkg -l | grep containerd
sudo dpkg --purge containerd
sudo apt autoremove
```

**Some helpful scripts for installation verifications**
```shell
kubectl get pod -n tigera-operator | grep Error | awk '{print $1}' 
    | xargs kubectl delete pod -n tigera-operator -->>>>> to remove tigera-operator
kubectl cluster-info
kubectl get nodes
kubectl get namespaces
kubectl get pods -n kube-system
kubectl get pods --all-namespaces
```