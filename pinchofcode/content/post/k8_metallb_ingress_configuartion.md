+++
title = 'Ingress and Load Balancer binding in bare-metal Kubernetes🚀'
date = 2024-03-29T15:53:47+05:30
draft = false
+++
# Ingress and Load Balancer binding in bare-metal Kubernetes🚀

In our previous post we have seen how we can add MetalLB to our bare-metal Kubernetes. 
It helps us with a single point of contact between client network and cluster.
 
MetalLB helps for load-balancing external services we can combine it with an ingress controller, 
to provide a stateful layer between the network and the services.
<!--more-->
## Prerequisites
1. Kubernetes cluster set up and running on-premises.
2. A CNI plugin compatible with MetalLB running (Canal/ Cilium/ Flannel/ Kube-ovn/Calico / kube-router / Weave-net)
2. Unique IPv4 addresses from the network of your cluster to configure IP address pools
3. UDP and TDP traffic on port 7946 must be allowed between nodes as MetalLB

## What is Ingress Controller?

In the previous post whith MetalLB config setup, we can deploy a service and make it accessible outside of the cluster. However, it is not satisfactory because a different IP address is allocated every time we deploy a service. We wish to be able to access every service from a single IP address. To achieve this, we use nginx-ingress.

Ingresses are a way to expose services outside of the cluster. It allows for hosts and routes to be created, thus allowing DNS resolutions of IP addresses. nginx-ingress listens and manages every service exposed on ports 80 and 443. It allows for two or more services to be exposed on the same IP address, using different hosts which means that a single IP is effectively used.

Ingress Controllers extend Kubernetes' capabilities by providing HTTP and HTTPS routing to services.
They act as a layer 7 (application layer) load balancer, directing traffic based on rules defined in Ingress resources.
Popular Ingress Controllers include NGINX Ingress Controller, Traefik, and HAProxy. We will go with NGINX in our
scenario.

Using both MetalLB and nginx-ingress allow us to get even closer to a proper load-balancer by layering the access to the services. The IP address remains the same, even if a node goes down thanks to the failover mechanism of MetalLB, and the services are now centralized thanks to nginx-ingress.

**How does it work**

![Alt text](/pinchofcode/images/metal_ingress.png)

The ingress controller generates the load-balancer IP of a load-balancer type service by selecting the first unused address of the provided IP address pool. The IP address pool must be composed of one or more unique IP addresses from the Host Network.

When data comes through the load-balancer IP, it is redirected to the elected speaker pod. In this example, the metallb-speaker of one of the worker-node will be elected, allowing the pod to spread the information with kube-proxy, looking for the pod to access.

If the speaker pod node goes down, another speaker assumes its place. If we decided to add a second service of load-balancer type, its load-balancer IP would be the next directly available IP.

Step 1: Installation of nginx-ingress

```yaml
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm search repo ingress-nginx --versions
helm repo update

helm template ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx > {YOUR_DESIRED_SERVER_LOCATION}/ingress-nginx.yaml

kubectl apply -f  ingress-nginx.yaml -n ingress-nginx
```

Step 2: Verification
```yaml
kubectl get service ingress-nginx-controller --namespace=ingress-nginx
```

Using this command, we check if the EXTERNAL-IP field is pending or not.
Normally, the first address of the config-map should be allocated to the ingress-nginx-controller.

Step 3: Intall Your application and access it

The next steps is deploy the pods as required, expose the pods as services. Once your 
application services are available, configure ingress rules.

Once rules are applied, traffic will start flowing through  


