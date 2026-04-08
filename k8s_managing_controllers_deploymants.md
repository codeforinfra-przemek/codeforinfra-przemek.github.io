---
layout: default
title: "Ansible desired state – controller configuration as code"
permalink: /ansible-desired-state.html
---

[← Back to index](/)


# Managing Kubernetes Controllers and Deployments



## Kube System:
<img width="915" height="653" alt="image" src="https://github.com/user-attachments/assets/e9855951-285f-4eaf-95ac-93c7868876a1" />
<img width="915" height="653" alt="image" src="https://github.com/user-attachments/assets/c7ad65dc-e280-4091-a499-a7cbbe91a6e4" /><img width="1373" height="215" alt="image" src="https://github.com/user-attachments/assets/fb2a1b41-35f8-4d72-937a-1b34331aabfd" /><img width="1377" height="214" alt="image" src="https://github.com/user-attachments/assets/f90a2772-517d-46c1-8e35-72a7a76d274c" /><img width="950" height="221" alt="image" src="https://github.com/user-attachments/assets/448f5c94-6b26-416a-b9d3-62892c89f18d" />
```
#Log into the Control Plane Node to drive these demos.
ssh aen@c1-cp1
cd ~/content/course/02/demos


#Demo 1 - Examining System Pods and their Controllers

#Inside the kube-system namespace, there's a collection of controllers supporting parts of the cluster's control plane
#How'd they get started since there's no cluster when they need to come online? Static Pod Manifests
kubectl get --namespace kube-system all 


#Let's look more closely at one of those deployments, requiring 2 pods up and runnning at all times.
kubectl get --namespace kube-system deployments coredns


#Daemonset Pods run on every node in the cluster by default, as new nodes are added these will be deployed to those nodes.
#There's a Pod for our Pod network, calico and one for the kube-proxy.
kubectl get --namespace kube-system daemonset


#We have 4 nodes, that's why for each daemonset they have 4 Pods.
kubectl get nodes
```
<img width="1397" height="718" alt="image" src="https://github.com/user-attachments/assets/65add623-4651-4d4b-9269-be0e9dcf3b28" />

```
#Log into the Control Plane Node to drive these demos.
ssh aen@c1-cp1
cd ~/content/course/02/demos


#Demo 2 Creating a Deployment Imperatively, with kubectl create,
#you have lot's of options available to you such as image, container ports, and replicas
kubectl create deployment hello-world --image=psk8s.azurecr.io/hello-app:1.0
kubectl scale deployment hello-world --replicas=5


#These two commands can be combined into one command if needed
#kubectl create deployment hello-world --image=psk8s.azurecr.io/hello-app:1.0 --replicas=5


#Check out the status of our imperative deployment
kubectl get deployment 


#Now let's delete that and move towards declarative configuration.
kubectl delete deployment hello-world




#Demo 1.b - Declaratively
#Simple Deployment
#Let's start off declaratively creating a deployment with a service.
kubectl apply -f deployment.yaml


#Check out the status of our deployment, which creates the ReplicaSet, which creates our Pods
kubectl get deployments hello-world


#The first replica set created in our deployment, which has the responsibility of keeping
#of maintaining the desired state of our application but starting and keeping 5 pods online. 
#In the name of the replica set is the pod-template-hash
kubectl get replicasets


#The actual pods as part of this replicaset, we know these pods belong to the replicaset because of the
#pod-template-hash in the name
kubectl get pods


#But also by looking at the 'Controlled By' property
kubectl describe pods | head -n 20


#It's the job of the deployment-controller to maintain state. Let's look at it a litte closer
#The selector defines which pods are a member of this deployment.
#Replicas define the current state of the deployment, we'll dive into what each one of these means later in the course.
#In Events, you can see the creation and scaling of the replica set to 5
kubectl describe deployment


#Remove our resources
kubectl delete deployment hello-world
kubectl delete service hello-world

```

```
#Log into the Control Plane Node to drive these demos.
ssh aen@c1-cp1
cd ~/content/course/02/demos


#Demo 1 - Deploy a Deployment which creates a ReplicaSet
kubectl apply -f deployment.yaml
kubectl get replicaset


#Let's look at the selector for this one...and the labels in the pod template
kubectl describe replicaset hello-world


#Let's delete this deployment which will delete the replicaset
kubectl delete deployment hello-world
kubectl get replicaset



#Deploy a ReplicaSet with matchExpressions
kubectl apply -f deployment-me.yaml


#Check on the status of our ReplicaSet
kubectl get replicaset


#Let's look at the Selector for this one...and the labels in the pod template
kubectl describe replicaset hello-world


#Demo 2 - Deleting a Pod in a ReplicaSet, application will self-heal itself
kubectl get pods
kubectl delete pods hello-world-[tab][tab]
kubectl get pods




#Demo 3 - Isolating a Pod from a ReplicaSet
#For more coverage on this see, Managing the Kubernetes API Server and Pods - Module 2 - Managing Objects with Labels, Annotations, and Namespaces
kubectl get pods --show-labels


#Edit the label on one of the Pods in the ReplicaSet, the replicaset controller will create a new pod
kubectl label pod hello-world-[tab][tab] app=DEBUG --overwrite
kubectl get pods --show-labels




#Demo 4 - Taking over an existing Pod in a ReplicaSet, relabel that pod to bring 
#it back into the scope of the replicaset...what's kubernetes going to do?
kubectl label pod hello-world-[tab][tab] app=hello-world-pod-me --overwrite


#One Pod will be terminated, since it will maintain the desired number of replicas at 5
kubectl get pods --show-labels
kubectl describe ReplicaSets




#Demo 5 - Node failures in ReplicaSets
#Shutdown a node
ssh c1-node3
sudo shutdown -h now


#c1-node3 Status Will go NotReady...takes about 1 minute.
kubectl get nodes --watch


#But there's a Pod still on c1-node3...wut? 
#Kubernetes is protecting against transient issues. Assumes the Pod is still running...
kubectl get pods -o wide


#Start up c1-node3, break out of watch when Node reports Ready, takes about 15 seconds
kubectl get nodes --watch


#That Pod that was on c1-node3 goes to Status Unknown then it will be restarted on that Node.
kubectl get pods -o wide 


#It will start the container back up on the Node c1-node3...see Restarts is now 1, takes about 10 seconds
#The pod didn't get rescheduled, it's still there, the container restart policy restarts the container which 
#starts at 10 seconds and defaults to Always. We covered this in detail in my course "Managing the Kuberentes API Server and Pods"
kubectl get pods -o wide --watch

#Shutdown a node again...
ssh c1-node3
sudo shutdown -h now


#Let's set a watch and wait...about 5 minutes and see what kubernetes will do.
#Because of the --pod-eviction-timeout duration setting on the kube-controller-manager, this pod will get killed after 5 minutes.
kubectl get pods --watch


#Orphaned Pod goes Terminating and a new Pod will be deployed in the cluster.
#If the Node returns the Pod will be deleted, if the Node does not, we'll have to delete it
kubectl get pods -o wide


#And go start c1-node3 back up again and see if those pods get deleted :)


#let's clean up...
kubectl delete deployment hello-world
kubectl delete service hello-world
```

## 
