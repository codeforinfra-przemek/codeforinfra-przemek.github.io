---
layout: default
title: "Ansible desired state – controller configuration as code"
permalink: /ansible-desired-state.html
---

[← Back to index](/)


# Configuring and Managing Kubernetes Networking, Services, and Ingress

## Kubernetes Network Fundamentials

<img width="1295" height="610" alt="image" src="https://github.com/user-attachments/assets/a8277057-b33b-498d-8b68-2f3e0e30d551" />
<img width="1432" height="776" alt="image" src="https://github.com/user-attachments/assets/34b0f26b-abda-4961-afb0-0a73bb45afb9" />
<img width="1408" height="747" alt="image" src="https://github.com/user-attachments/assets/73137727-b230-4f72-8118-d821d796bfc3" />
<img width="1349" height="739" alt="image" src="https://github.com/user-attachments/assets/1130d3c5-9a66-4ab2-88c0-483ead769cac" />

### Investigate Network:
```
#1 - Investigating Kubernetes Networking
#Log into our local cluster
ssh aen@c1-cp1
cd ~/content/course/02/demos



#Local Cluster - Calico CNI Plugin
#Get all Nodes and their IP information, INTERNAL-IP is the real IP of the Node
kubectl get nodes -o wide


#Let's deploy a basic workload, hello-world with 3 replicas to create some pods on the pod network.
kubectl apply -f Deployment.yaml


#Get all Pods, we can see each Pod has a unique IP on the Pod Network.
#Our Pod Network was defined in the first course and we chose 192.168.0.0/16
kubectl get pods -o wide


#Let's hop inside a pod and check out it's networking, a single interface an IP on the Pod Network
#The line below will get a list of pods from the label query and return the name of the first pod in the list
PODNAME=$(kubectl get pods --selector=app=hello-world -o jsonpath='{ .items[0].metadata.name }')
echo $PODNAME
kubectl exec -it $PODNAME -- /bin/sh
ip addr
exit


#For the Pod on c1-node1, let's find out how traffic gets from c1-cp1 to c1-node1 to get to that Pod.

#Look at the annotations, specifically the annotation projectcalico.org/IPv4IPIPTunnelAddr: 192.168.19.64...your IP may vary
#Check out the Addresses: InternalIP, that's the real IP of the Node.
# Pod IPs are allocated from the network Pod Network which is configurable in Calico, it's controlling the IP allocation.
# Calico is using a tunnel interfaces to implement the Pod Network model. 
# Traffic going to other Pods will be sent into the tunnel interface and directly to the Node running the Pod.
# For more info on Calico's operations https://docs.projectcalico.org/reference/cni-plugin/configuration
kubectl describe node c1-cp1 | more


#Let's see how the traffic gets to c1-node1 from c1-cp1
#Via routes on the node, to get to c1-node1 traffic goes into tunl0/192.168.19.64...your IP may vary
#Calico handles the tunneling and sends the packet to the correct node to be send on into the Pod running on that Node based on the defined routes
#Follow each route, showing how to get to the Pod IP, it will need to go to the tun0 interface.
#There cali* interfaces are for each Pod on the Pod network, traffic destined for the Pod IP will have a 255.255.255.255 route to this interface.
kubectl get pods -o wide
route


#The local tunl0 is 192.168.19.64, packets destined for Pods running on c1-cp1 will be routed to this interface and get encapsulated
#Then send to the destination node for de-encapsulation.
ip addr


#Log into c1-node1 and look at the interfaces, there's tunl0 192.168.222.192...this is this node's tunnel interface
ssh aen@c1-node1


#This tunl0 is the destination interface, on this Node its 192.168.222.192, which we saw on the route listing on c1-cp1
ip addr


#All Nodes will have routes back to the other Nodes via the tunl0 interface
route


#Exit back to c1-cp1
exit







#Azure Kubernetes Service - kubenet
#Get all Nodes and their IP information, INTERNAL-IP is the real IP of the Node
kubectl config use-context 'CSCluster'


#Let's deploy a basic workload, hello-world with 3 replicas.
kubectl apply -f Deployment.yaml

#Note the INTERNAL-IP, these are on the virtual network in Azure, the real IPs of the underlying VMs
kubectl get nodes -o wide


#This time we're using a different network plugin, kubenet. It's based on routes/bridges rather than tunnels. Let's explore
#Check out Addresses and PodCIDR
kubectl describe nodes | more


#The Pods are getting IPs from their Node's PodCIDR Range
kubectl get pods -o wide


#Access an AKS Node via SSH so we can examine it's network config which uses kubenet
#https://docs.microsoft.com/en-us/azure/aks/ssh#configure-virtual-machine-scale-set-based-aks-clusters-for-ssh-access
NODENAME=$(kubectl get nodes -o jsonpath='{.items[0].metadata.name}')
kubectl debug node/$NODENAME -it --image=mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11


#Check out the routes, notice the route to the local Pod Network matching PodCIDR for this Node sending traffic to cbr0
#The routes for the other PodCIDR ranges on the other Nodes are implemented in the cloud's virtual network. 
route


#In Azure, these routes are implemented as route tables assigned to the virtual machine's for your Nodes.
#You'll find the routes implemented in the Resource Group as a Route Table assigned to the subnet the Nodes are on.
#This is a link to my Azure account, your's will vary.
#https://portal.azure.com/#@nocentinohotmail.onmicrosoft.com/resource/subscriptions/fd0c5e48-eea6-4b37-a076-0e23e0df74cb/resourceGroups/mc_kubernetes-cloud_cscluster_centralus/providers/Microsoft.Network/routeTables/aks-agentpool-89481420-routetable/overview

#Check out the eth0, actual Node interface IP, then cbr0 which is the bridge the Pods are attached to and 
#has an IP on the Pod Network.
#Each Pod has an veth interface on the bridge, which you see here, and and interface inside the container
#which will have the Pod IP.
ip addr 


#Let's check out the bridge's 'connections'
brctl show


#Exit the container on the node
exit


#Here is the Pod's interface and it's IP. 
#This interface is attached to the cbr0 bridge on the Node to get access to the Pod network. 
PODNAME=$(kubectl get pods -o jsonpath='{ .items[0].metadata.name }')
kubectl exec -it $PODNAME -- ip addr


#And inside the pod, there's a default route in the pod to the interface 10.244.0.1 which is the brige interface cbr0.
#Then the Node will route it on the Node network for reachability to other nodes.
kubectl exec -it $PODNAME -- route


#Delete the deployment in AKS, switch to the local cluster and delete the deployment too. 
kubectl delete -f Deployment.yaml 
kubectl config use-context kubernetes-admin@kubernetes
kubectl delete -f Deployment.yaml 

```
<img width="1393" height="743" alt="image" src="https://github.com/user-attachments/assets/5853d5bb-b449-468f-bf1e-7500533da48e" />

### Configuring DNS:
```
ssh aen@c1-cp1
cd ~/content/course/02/demos


#1. Investigating the Cluster DNS Service
#It's Deployed as a Service in the cluster with a Deployment in the kube-system namespace
kubectl get service --namespace kube-system


#Two Replicas, Args injecting the location of the config file which is backed by ConfigMap mounted as a Volume.
kubectl describe deployment coredns --namespace kube-system | more
 

#The configmap defining the CoreDNS configuration and we can see the default forwarder is /etc/resolv.conf
kubectl get configmaps --namespace kube-system coredns -o yaml | more




#2. Configuring CoreDNS to use custom Forwarders, spaces not tabs!
#Defaults use the nodes DNS Servers for fowarders
#Replaces forward . /etc/resolv.conf
#with forward . 1.1.1.1
#Add a conditional domain forwarder for a specific domain
#ConfigMap will take a second to update the mapped file and the config to be reloaded
kubectl apply -f CoreDNSConfigCustom.yaml --namespace kube-system


#How will we know when the CoreDNS configuration file is updated in the pod?
#You can tail the log looking for the reload the configuration file...this can take a minute or two
#Also look for any errors post configuration. Seeing [WARNING] No files matching import glob pattern: custom/*.override is normal.
kubectl logs --namespace kube-system --selector 'k8s-app=kube-dns' --follow 


#Run some DNS queries against the kube-dns service cluster ip to ensure everything works...
SERVICEIP=$(kubectl get service --namespace kube-system kube-dns -o jsonpath='{ .spec.clusterIP }')
nslookup www.pluralsight.com $SERVICEIP
nslookup www.centinosystems.com $SERVICEIP


#On c1-cp1, let's put the default configuration back, using . forward /etc/resolv.conf 
kubectl apply -f CoreDNSConfigDefault.yaml --namespace kube-system



#3. Configuring Pod DNS client Configuration
kubectl apply -f DeploymentCustomDns.yaml


#Let's check the DNS configuration of a Pod created with that configuration
#This line will grab the first pod matching the defined selector
PODNAME=$(kubectl get pods --selector=app=hello-world-customdns -o jsonpath='{ .items[0].metadata.name }')
echo $PODNAME
kubectl exec -it $PODNAME -- cat /etc/resolv.conf


#Clean up our resources
kubectl delete -f DeploymentCustomDns.yaml



#Demo 3 - let's get a pods DNS A record and a Services A record
#Create a deployment and a service
kubectl apply -f Deployment.yaml


#Get the pods and their IP addresses
kubectl get pods -o wide


#Get the address of our DNS Service again...just in case
SERVICEIP=$(kubectl get service --namespace kube-system kube-dns -o jsonpath='{ .spec.clusterIP }')


#For one of the pods replace the dots in the IP address with dashes for example 192.168.206.68 becomes 192-168-206-68
#We'll look at some additional examples of Service Discovery in the next module too.
nslookup 192-168-206-[XX].default.pod.cluster.local $SERVICEIP


#Our Services also get DNS A records
#There's more on service A records in the next demo
kubectl get service 
nslookup hello-world.default.svc.cluster.local $SERVICEIP


#Clean up our resources
kubectl delete -f Deployment.yaml


#TODO for the viewer...you can use this technique to verify your DNS forwarder configuration from the first demo in this file. 
#Recreate the custom configuration by applying the custom configmap defined in CoreDNSConfigCustom.yaml
#Logging in CoreDNS will log the query, but not which forwarder it was sent to. 
#We can use tcpdump to listen to the packets on the wire to see where the DNS queries are being sent to.


#Find the name of a Node running one of the DNS Pods running...so we're going to observe DNS queries there.
DNSPODNODENAME=$(kubectl get pods --namespace kube-system --selector=k8s-app=kube-dns -o jsonpath='{ .items[0].spec.nodeName }')
echo $DNSPODNODENAME


#Let's log into THAT node running the dns pod and start a tcpdump to watch our dns queries in action.
#Your interface (-i) name may be different
ssh aen@$DNSPODNODENAME
sudo tcpdump -i ens33 port 53 -n 


#In a second terminal, let's test our DNS configuration from a pod to make sure we're using the configured forwarder.
#When this pod starts, it will point to our cluster dns service.
#Install dnsutils for nslookup and dig
ssh aen@c1-cp1
kubectl run -it --rm debian --image=debian
apt-get update && apt-get install dnsutils -y


#In our debian pod let's look at the dns config and run two test DNS queries
#The nameserver will be your cluster dns service cluster ip.
#We'll query two domains to generate traffic for our tcpdump
cat /etc/resolv.conf
nslookup www.pluralsight.com
nslookup www.centinosystems.com


#Switch back to our second terminal and review the tcpdump, confirming each query is going to the correct forwarder
#Here is some example output...www.pluralsight.com is going to 1.1.1.1 and www.centinosystems.com is going to 9.9.9.9
#172.16.94.13.63841 > 1.1.1.1.53: 24753+ A? www.pluralsight.com. (37)
#172.16.94.13.42523 > 9.9.9.9.53: 29485+ [1au] A? www.centinosystems.com. (63)

#Exit the tcpdump
ctrl+c


#Log out of the node, back onto c1-cp1
exit


#Switch sessions and break out of our pod and it will be deleted.
exit


#Exit out of our second SSH session and get a shell back on c1-cp1
exit
```

## Configuring and Managing Application Access with Services:

<img width="1185" height="726" alt="image" src="https://github.com/user-attachments/assets/ea8e991f-9f8b-4d63-a220-7709454d4765" />
<img width="1369" height="730" alt="image" src="https://github.com/user-attachments/assets/00088626-9836-47cb-a90f-76fd47d0ad8b" />
<img width="1288" height="598" alt="image" src="https://github.com/user-attachments/assets/9d29f51b-dc6e-49f1-8f5a-993bade4872b" />
<img width="1430" height="771" alt="image" src="https://github.com/user-attachments/assets/bc95d4b5-53a1-4c89-be8f-b7524520eecd" />
<img width="1434" height="778" alt="image" src="https://github.com/user-attachments/assets/1e0b7bdd-a18d-48d2-a840-18f1dd6a4f52" />
<img width="1430" height="772" alt="image" src="https://github.com/user-attachments/assets/9e629183-9c0c-4fb1-814f-ad9cc43d6f4a" />

### Service:
```
ssh aen@c1-cp1
cd ~/content/course/03/demos/

#1 - Exposing and accessing applications with Services on our local cluster
#ClusterIP

#Imperative, create a deployment with one replica
kubectl create deployment hello-world-clusterip \
    --image=psk8s.azurecr.io/hello-app:1.0


#When creating a service, you can define a type, if you don't define a type, the default is ClusterIP
kubectl expose deployment hello-world-clusterip \
    --port=80 --target-port=8080 --type ClusterIP


#Get a list of services, examine the Type, CLUSTER-IP and Port
kubectl get service


#Get the Service's ClusterIP and store that for reuse.
SERVICEIP=$(kubectl get service hello-world-clusterip -o jsonpath='{ .spec.clusterIP }')
echo $SERVICEIP


#Access the service inside the cluster
curl http://$SERVICEIP


#Get a listing of the endpoints for a service, we see the one pod endpoint registered.
kubectl get endpoints hello-world-clusterip
kubectl get pods -o wide


#Access the pod's application directly on the Target Port on the Pod, not the service's Port, useful for troubleshooting.
#Right now there's only one Pod and its one Endpoint
kubectl get endpoints hello-world-clusterip
PODIP=$(kubectl get endpoints hello-world-clusterip -o jsonpath='{ .subsets[].addresses[].ip }')
echo $PODIP
curl http://$PODIP:8080


#Scale the deployment, new endpoints are registered automatically
kubectl scale deployment hello-world-clusterip --replicas=6
kubectl get endpoints hello-world-clusterip


#Access the service inside the cluster, this time our requests will be load balanced...whooo!
curl http://$SERVICEIP


#The Service's Endpoints match the labels, let's look at the service and it's selector and the pods labels.
kubectl describe service hello-world-clusterip
kubectl get pods --show-labels


#Clean up these resources for the next demo
kubectl delete deployments hello-world-clusterip
kubectl delete service hello-world-clusterip




#2 - Creating a NodePort Service
#Imperative, create a deployment with one replica
kubectl create deployment hello-world-nodeport \
    --image=psk8s.azurecr.io/hello-app:1.0


#When creating a service, you can define a type, if you don't define a type, the default is ClusterIP
kubectl expose deployment hello-world-nodeport \
    --port=80 --target-port=8080 --type NodePort


#Let's check out the services details, there's the Node Port after the : in the Ports column. It's also got a ClusterIP and Port
#This NodePort service is available on that NodePort on each node in the cluster
kubectl get service


CLUSTERIP=$(kubectl get service hello-world-nodeport -o jsonpath='{ .spec.clusterIP }')
PORT=$(kubectl get service hello-world-nodeport -o jsonpath='{ .spec.ports[].port }')
NODEPORT=$(kubectl get service hello-world-nodeport -o jsonpath='{ .spec.ports[].nodePort }')

#Let's access the services on the Node Port...we can do that on each node in the cluster and 
#from outside the cluster...regardless of where the pod actually is

#We have only one pod online supporting our service
kubectl get pods -o wide


#And we can access the service by hitting the node port on ANY node in the cluster on the Node's Real IP or Name.
#This will forward to the cluster IP and get load balanced to a Pod. Even if there is only one Pod.
curl http://c1-cp1:$NODEPORT
curl http://c1-node1:$NODEPORT
curl http://c1-node2:$NODEPORT
curl http://c1-node3:$NODEPORT


#And a Node port service is also listening on a Cluster IP, in fact the Node Port traffic is routed to the ClusterIP
echo $CLUSTERIP:$PORT
curl http://$CLUSTERIP:$PORT


#Let's delete that service
kubectl delete service hello-world-nodeport
kubectl delete deployment hello-world-nodeport




#3 - Creating LoadBalancer Services in Azure or any cloud
#Switch contexts into AKS, we created this cluster together in 'Kubernetes Installation and Configuration Fundamentals'
#I've added a script to create a GKE and AKS cluster this course's downloads.
kubectl config use-context 'CSCluster'


#Let's create a deployment
kubectl create deployment hello-world-loadbalancer \
    --image=psk8s.azurecr.io/hello-app:1.0


#When creating a service, you can define a type, if you don't define a type, the default is ClusterIP
kubectl expose deployment hello-world-loadbalancer \
    --port=80 --target-port=8080 --type LoadBalancer


#Can take a minute for the load balancer to provision and get an public IP, you'll see EXTERNAL-IP as <pending>
kubectl get service


LOADBALANCERIP=$(kubectl get service hello-world-loadbalancer -o jsonpath='{ .status.loadBalancer.ingress[].ip }')
curl http://$LOADBALANCERIP:$PORT


#The loadbalancer, which is 'outside' your cluster, sends traffic to the NodePort Service which sends it to the ClusterIP to get to your pods!
#Your cloud load balancer will have health probes checking the health of the node port service on the real node IPs.
#This isn't the health of our application, that still needs to be configured via readiness/liveness probes and maintained by your Deployment configuration
kubectl get service hello-world-loadbalancer



#Clean up the resources from this demo
kubectl delete deployment hello-world-loadbalancer
kubectl delete service hello-world-loadbalancer


#Let's switch back to our local cluster
kubectl config use-context kubernetes-admin@kubernetes



#Declarative examples
kubectl config use-context kubernetes-admin@kubernetes
kubectl apply -f service-hello-world-clusterip.yaml
kubectl get service


#Creating a NodePort with a predefined port, first with a port outside of the NodePort range then a corrected one.
kubectl apply -f service-hello-world-nodeport-incorrect.yaml
kubectl apply -f service-hello-world-nodeport.yaml
kubectl get service


#Switch contexts to Azure to create a cloud load balancer
kubectl config use-context 'CSCluster'
kubectl apply -f service-hello-world-loadbalancer.yaml
kubectl get service


#Clean up these resources
kubectl delete -f service-hello-world-loadbalancer.yaml
kubectl config use-context kubernetes-admin@kubernetes
kubectl delete -f service-hello-world-nodeport.yaml
kubectl delete -f service-hello-world-clusterip.yaml

```

### Service Discovery:

<img width="1414" height="775" alt="image" src="https://github.com/user-attachments/assets/58d2f9fd-5289-49c2-a374-fc27037ea2a2" />
<img width="1349" height="773" alt="image" src="https://github.com/user-attachments/assets/e840410d-41f4-4d18-9bc7-0e57e66d418c" />

```
ssh aen@c1-cp1
cd ~/content/course/03/demos/


#Service Discovery
#Cluster DNS

#Let's create a deployment in the default namespace
kubectl create deployment hello-world-clusterip \
    --image=psk8s.azurecr.io/hello-app:1.0


#Let's create a deployment in the default namespace
kubectl expose deployment hello-world-clusterip \
    --port=80 --target-port=8080 --type ClusterIP


#We can use nslookup or dig to investigate the DNS record, it's CNAME @10.96.0.10 is the cluser IP of our DNS Server
kubectl get service kube-dns --namespace kube-system


#Each service gets a DNS record, we can use this in our applications to find services by name.
#The A record is in the form <servicename>.<namespace>.svc.<clusterdomain>
nslookup hello-world-clusterip.default.svc.cluster.local 10.96.0.10
kubectl get service hello-world-clusterip


#Create a namespace, deployment with one replica and a service
kubectl create namespace ns1


#Let's create a deployment with the same name as the first one, but in our new namespace
kubectl create deployment hello-world-clusterip --namespace ns1 \
    --image=psk8s.azurecr.io/hello-app:1.0


kubectl expose deployment hello-world-clusterip --namespace ns1 \
    --port=80 --target-port=8080 --type ClusterIP


#Let's check the DNS record for the service in the namespace, ns1. See how ns1 is in the DNS record?
#<servicename>.<namespace>.svc.<clusterdomain>
nslookup hello-world-clusterip.ns1.svc.cluster.local 10.96.0.10


#Our service in the default namespace is still there, these are completely unique services.
nslookup hello-world-clusterip.default.svc.cluster.local 10.96.0.10


#Get the environment variables for the pod in our default namespace
#More details about the lifecycle of variables in "Configuring and Managing Kubernetes Storage and Scheduling"
#Only the kubernetes service is available? Why? I created the deployment THEN I created the service
PODNAME=$(kubectl get pods -o jsonpath='{ .items[].metadata.name }')
echo $PODNAME
kubectl exec -it $PODNAME -- env | sort


#Environment variables are only created at pod start up, so let's delete the pod
kubectl delete pod $PODNAME


#And check the enviroment variables again...
PODNAME=$(kubectl get pods -o jsonpath='{ .items[].metadata.name }')
echo $PODNAME
kubectl exec -it $PODNAME -- env | sort


#ExternalName
kubectl apply -f service-externalname.yaml


#The record is in the form <servicename>.<namespace>.<clusterdomain>. You may get an error that says ** server can't find hello-world.api.example.com: NXDOMAIN this is ok.
nslookup hello-world-api.default.svc.cluster.local 10.96.0.10




#Let's clean up our resources in this demo
kubectl delete service hello-world-api
kubectl delete service hello-world-clusterip
kubectl delete service hello-world-clusterip --namespace ns1
kubectl delete deployment hello-world-clusterip
kubectl delete deployment hello-world-clusterip --namespace ns1
kubectl delete namespace ns1

```
## Configuring and Managing Kubernetes Networking, Services, and Ingress:

<img width="641" height="350" alt="image" src="https://github.com/user-attachments/assets/7ef9f61a-a3b7-4e54-948c-05d1cb8493d3" />
<img width="643" height="353" alt="image" src="https://github.com/user-attachments/assets/4b7fa040-0210-40dc-ae51-ecbfbbea71d4" />
<img width="646" height="344" alt="image" src="https://github.com/user-attachments/assets/ba74a27e-e9b1-45cd-ab0f-9dbb8a98a3d4" />

```
ssh aen@c1-cp1
cd ~/content/course/04/demos/

#Check out 1-ingress-loadbalancer.sh for the cloud demos

#Demo 1 - Deploying an ingress controller
#For our Ingress Controller, we're going to go with nginx, widely available and easy to use. 
#Follow this link here to find a manifest for nginx Ingress Controller for various infrastructures, Cloud, Bare Metal, EKS and more.
#We have to choose a platform to deploy in...we can choose Cloud, Bare-metal (which we can use in our local cluster) and more.
https://kubernetes.github.io/ingress-nginx/deploy/


#Bare-metal: On our on prem cluster: Bare Metal (NodePort)
#Let's make sure we're in the right context and deploy the manifest for the Ingress Controller found in the link just above (around line 9).
kubectl config use-context kubernetes-admin@kubernetes
kubectl apply -f ./baremetal/deploy.yaml


#Using this manifest, the Ingress Controller is in the ingress-nginx namespace but 
#It will monitor for Ingresses in all namespaces by default. If can be scoped to monitor a specific namespace if needed.


#Check the status of the pods to see if the ingress controller is online.
kubectl get pods --namespace ingress-nginx


#Now let's check to see if the service is online. This of type NodePort, so do you have an EXTERNAL-IP?
kubectl get services --namespace ingress-nginx


#Check out the ingressclass nginx...we have not set the is-default-class so in each of our Ingresses we will need 
#specify an ingressclassname
kubectl describe ingressclasses nginx
#kubectl annotate ingressclasses nginx "ingressclass.kubernetes.io/is-default-class=true"


#Demo 2 - Single Service
#Create a deployment, scale it to 2 replicas and expose it as a serivce. 
#This service will be ClusterIP and we'll expose this service via the Ingress.
kubectl create deployment hello-world-service-single --image=psk8s.azurecr.io/hello-app:1.0
kubectl scale deployment hello-world-service-single --replicas=2
kubectl expose deployment hello-world-service-single --port=80 --target-port=8080 --type=ClusterIP



#Create a single Ingress routing to the one backend service on the service port 80 listening on all hostnames
kubectl apply -f ingress-single.yaml


#Get the status of the ingress. It's routing for all host names on that public IP on port 80
#This is a NodePort service so there's no public IP, its the NodePort Serivce that you'll use for access or integration into load balancing.
#If you don't define an ingressclassname and don't have a default ingress class the address won't be updated.
kubectl get ingress --watch #Wait for the Address to be populated before proceeding
kubectl get services --namespace ingress-nginx


#Notice the backends are the Service's Endpoints...so the traffic is going straight from the Ingress Controller to the Pod cutting out the kube-proxy hop.
#Also notice, the default back end is the same service, that's because we didn't define any rules and
#we just populated ingress.spec.backend. We're going to look at rules next...
kubectl describe ingress ingress-single


#Access the application via the exposed ingress that's listening the NodePort and it's static port, let's get some variables so we can reused them
INGRESSNODEPORTIP=$(kubectl get ingresses ingress-single -o jsonpath='{ .status.loadBalancer.ingress[].ip }')
NODEPORT=$(kubectl get svc -n ingress-nginx ingress-nginx-controller -o jsonpath='{ .spec.ports[?(@.name=="http")].nodePort }')
echo $INGRESSNODEPORTIP:$NODEPORT
curl http://$INGRESSNODEPORTIP:$NODEPORT




#Demo 3 - Multiple Services with path based routing
#Let's create two additional services
kubectl create deployment hello-world-service-blue --image=psk8s.azurecr.io/hello-app:1.0
kubectl create deployment hello-world-service-red  --image=psk8s.azurecr.io/hello-app:1.0

kubectl expose deployment hello-world-service-blue --port=4343 --target-port=8080 --type=ClusterIP
kubectl expose deployment hello-world-service-red  --port=4242 --target-port=8080 --type=ClusterIP


#Let's create an ingress with paths each routing to different backend services.
kubectl apply -f ingress-path.yaml


#We now have two, one for all hosts and the other for our defined host with two paths
#The Ingress controller is implementing these ingresses and we're sharing the one public IP, don't proceed until you see 
#the address populated for your ingress
kubectl get ingress --watch


#We can see the host, the path, and the backends.
kubectl describe ingress ingress-path


#Our ingress on all hosts is still routing to service single, since we're accessing the URL with an IP and not a domain name or host header.
curl http://$INGRESSNODEPORTIP:$NODEPORT


#Our paths are routing to their correct services, if we specify a host header or use a DNS name to access the ingress. That's how the rule will route the request.
curl http://$INGRESSNODEPORTIP:$NODEPORT/red  --header 'Host: path.example.com'
curl http://$INGRESSNODEPORTIP:$NODEPORT/blue --header 'Host: path.example.com'


#Example Prefix matches...these will all match and get routed to red
curl http://$INGRESSNODEPORTIP:$NODEPORT/red/1  --header 'Host: path.example.com'
curl http://$INGRESSNODEPORTIP:$NODEPORT/red/2  --header 'Host: path.example.com'


#Example Exact mismatches...these will all 404
curl http://$INGRESSNODEPORTIP:$NODEPORT/Blue  --header 'Host: path.example.com'
curl http://$INGRESSNODEPORTIP:$NODEPORT/blue/1  --header 'Host: path.example.com'
curl http://$INGRESSNODEPORTIP:$NODEPORT/blue/2  --header 'Host: path.example.com'


#If we don't specify a path we'll get a 404 while specifying a host header. 
#We'll need to configure a path and backend for / or define a default backend for the service
curl http://$INGRESSNODEPORTIP:$NODEPORT/     --header 'Host: path.example.com'


#Add a backend to the ingress listenting on path.example.com pointing to the single service
kubectl apply -f ingress-path-backend.yaml


#We can see the default backend, and in the Rules, the host, the path, and the backends.
kubectl describe ingress ingress-path


#Now we'll hit the default backend service, single for the undefined path.
curl http://$INGRESSNODEPORTIP:$NODEPORT/ --header 'Host: path.example.com'




#Demo 4 - Name based virtual hosts
#Now, let's route traffic to the services using named based virtual hosts rather than paths
kubectl apply -f ingress-namebased.yaml
kubectl get ingress --watch #Wait for the Address to be populated before proceeding

curl http://$INGRESSNODEPORTIP:$NODEPORT/ --header 'Host: red.example.com'
curl http://$INGRESSNODEPORTIP:$NODEPORT/ --header 'Host: blue.example.com'




#Demo 5 - TLS Example
#1 - Generate a certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout tls.key -out tls.crt -subj "/C=US/ST=ILLINOIS/L=CHICAGO/O=IT/OU=IT/CN=tls.example.com"


#2 - Create a secret with the key and the certificate
kubectl create secret tls tls-secret --key tls.key --cert tls.crt


#3 - Create an ingress using the certificate and key. This uses HTTPS for both / and /red 
kubectl apply -f ingress-tls.yaml


#Check the status...do we have an IP?
kubectl get ingress --watch #Wait for the Address to be populated before proceeding


#Test access to the hostname...we need --resolve because we haven't registered the DNS name
#TLS is a layer lower than host headers, so we have to specify the correct DNS name. 
kubectl get service -n ingress-nginx ingress-nginx-controller
NODEPORTHTTPS=$(kubectl get svc -n ingress-nginx ingress-nginx-controller -o jsonpath='{ .spec.ports[?(@.name=="https")].nodePort }')
echo $NODEPORTHTTPS
curl https://tls.example.com:$NODEPORTHTTPS/ \
    --resolve tls.example.com:$NODEPORTHTTPS:$INGRESSNODEPORTIP \
    --insecure --verbose


#Clean up from our demo
kubectl delete ingresses ingress-path
kubectl delete ingresses ingress-tls
kubectl delete ingresses ingress-namebased
kubectl delete deployment hello-world-service-single
kubectl delete deployment hello-world-service-red
kubectl delete deployment hello-world-service-blue
kubectl delete service hello-world-service-single
kubectl delete service hello-world-service-red
kubectl delete service hello-world-service-blue
kubectl delete secret tls-secret
rm tls.crt
rm tls.key

#Delete the ingress, ingress controller and other configuration elements
kubectl delete -f ./baremetal/deploy.yaml
```
