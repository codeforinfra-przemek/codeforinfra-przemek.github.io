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

###
