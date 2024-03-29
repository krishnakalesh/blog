+++
title = 'MetalLB as a Load Balancer Service for On-Premises Kubernetes 🎡'
date = 2024-03-13T16:40:55+05:30
draft = false
+++
# MetalLB as a Load Balancer Service for On-Premises Kubernetes 🎡

Unlike cloud providers that offer built-in load balancers, on-premises Kubernetes clusters often lack this functionality out-of-the-box. In this guide, we'll walk through the process of configuring MetalLB as a load balancer service for your on-premises Kubernetes cluster, enabling you to distribute traffic to your applications effectively.
<!--more-->
## Prerequisites
1. Kubernetes cluster set up and running on-premises.
2. Access to the Kubernetes control plane with appropriate permissions.
3. Basic knowledge of networking concepts and Kubernetes resources.

## Steps

**1. Install MetalLB**
When we use on-premises environment, we need a tool similar to MetalLB to create VIPs.
This is in the case where we have chosen to expose our apps to clients using 'Load Balancer' service over the
NodePort way as 'Load Balancer' is the preferred way.

To install MettalLB, we can use helm commands (Default L2 mode)

```shell
kubectl create ns metallb-system
helm repo add metallb https://metallb.github.io/metallb	
helm upgrade --install -n metallb-system metallb oci://registry-1.docker.io/bitnamicharts/metallb
	or
helm install metallb -n metallb-system metallb/metallb
```
**2. Add IP range pool**
Add IP range pool in a yaml

Sample [Range Allocation yaml](https://krishnakalesh.github.io/pinchofcode/data/L2-range-allocation.yaml)
```python
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: xyzcorplb
  namespace: metallb-system
spec:
  addresses:
  - 198.144.1.4/32
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: empty
  namespace: metallb-system
```
Replace 122.122.1.11 with your desired CIDR range. The "/32" denotes the subnet mask in CIDR notation. 
It indicates that all 32 bits of the IPv4 address are part of the network address, 
leaving no bits for host addresses. 

Essentially, it specifies a network with just one host address, which is the IP address itself.

Once changes are done, apply it using 
```shell
$kubectl apply -f L2-range-allocation.yaml
```
Metal LB pods will start appearing 
![Alt text](/pinchofcode/images/metal1.png)

**2. Deploy a Load-Balanced Service**

Here, we deploy a sample application, such as Nginx, and obtain a VIP for it
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template: 
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: LoadBalancer
```

Save this to a file named nginx-deployment.yaml and do
```shell
kubectl apply -f nginx-deployment.yaml
```
**2. Verify Service**

![Alt text](https://raw.githubusercontent.com/krishnakalesh/devops-diagrams/main/metallb/pictures/metal-lb_1.png)

```shell
$kubectl get service
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP      10.55.0.1       <none>        443/TCP        2d
nginx        LoadBalancer   10.55.144.12   198.144.1.4   80:31186/TCP   50s
```
We can access the application using URL - http://198.144.1.4:80

