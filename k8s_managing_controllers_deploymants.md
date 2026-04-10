---
layout: default
title: "Ansible desired state – controller configuration as code"
permalink: /ansible-desired-state.html
---

[← Back to index](/)


# Managing Kubernetes Controllers and Deployments



## Kube System:
<img width="915" height="653" alt="image" src="https://github.com/user-attachments/assets/e9855951-285f-4eaf-95ac-93c7868876a1" />
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

## Maintaining Applications with Deployments

### Updating a Deployment and checking our rollout status:
<img width="1271" height="701" alt="image" src="https://github.com/user-attachments/assets/45cf4e58-6a9b-4463-835f-ce473a40ddfa" />

```
ssh aen@c1-cp1
cd ~/content/course/03/demos/

#Demo 1 - Updating a Deployment and checking our rollout status
#Let's start off with rolling out v1
kubectl apply -f deployment.yaml


#Check the status of the deployment
kubectl get deployment hello-world


#Now let's apply that deployment, run both this and line 18 at the same time.
kubectl apply -f deployment.v2.yaml


#Let's check the status of that rollout, while the command blocking your deployment is in the Progressing status.
kubectl rollout status deployment hello-world


#Expect a return code of 0 from kubectl rollout status...that's how we know we're in the Complete status.
echo $?


#Let's walk through the description of the deployment...
#Check out Replicas, Conditions and Events OldReplicaSet (will only be populated during a rollout) and NewReplicaSet
#Conditions (more information about our objects state):
#     Available      True    MinimumReplicasAvailable
#     Progressing    True    NewReplicaSetAvailable (when true, deployment is still progressing or complete)
kubectl describe deployments hello-world


#Both replicasets remain, and that will become very useful shortly when we use a rollback :)
kubectl get replicaset


#The NewReplicaSet, check out labels, replicas, status and pod-template-hash
kubectl describe replicaset hello-world-86666f466d


#The OldReplicaSet, check out labels, replicas, status and pod-template-hash
kubectl describe replicaset hello-world-75d856dc89
```
### Updating to a non-existent image:
<img width="1319" height="698" alt="image" src="https://github.com/user-attachments/assets/06b19d79-19f9-41ae-83fa-b1d2e7febb93" />

```
ssh aen@c1-cp1
cd ~/content/course/03/demos/

#Demo 2.1 - Updating to a non-existent image. 
#Delete any current deployments, because we're interested in the deploy state changes.
kubectl delete deployment hello-world
kubectl delete service hello-world

#Create our v1 deployment, then update it to v2
kubectl apply -f deployment.yaml
kubectl apply -f deployment.v2.yaml

```
<img width="1311" height="682" alt="image" src="https://github.com/user-attachments/assets/3f2c7681-b18f-4c0b-9d8f-d2471cbdc884" />

```
#Observe behavior since new image wasn’t available, the ReplicaSet doesn't go below maxUnavailable
kubectl apply -f deployment.broken.yaml


#Why isn't this finishing...? after progressDeadlineSeconds which we set to 10 seconds (defaults to 10 minutes)
kubectl rollout status deployment hello-world


#Expect a return code of 1 from kubectl rollout status...that's how we know we're in the failed status.
echo $?


#Let's check out Pods, ImagePullBackoff/ErrImagePull...ah an error in our image definition.
#Also, it stopped the rollout at 5, that's kind of nice isn't it?
#And 8 are online, let's look at why.
kubectl get pods


#What is maxUnavailable? 25%...So only two Pods in the ORIGINAL ReplicaSet are offline and 8 are online.
#What is maxSurge? 25%? So we have 13 total Pods, or 25% in addition to Desired number.
#Look at Replicas and OldReplicaSet 8/8 and NewReplicaSet 5/5.
#  Available      True    MinimumReplicasAvailable
#  Progressing    False   ProgressDeadlineExceeded
kubectl describe deployments hello-world 


#Let's sort this out now...check the rollout history, but which revision should we rollback to?
kubectl rollout history deployment hello-world


#It's easy in this example, but could be harder for complex systems.
#Let's look at our revision Annotation, should be 3
kubectl describe deployments hello-world | head

#We can also look at the changes applied in each revision to see the new pod templates.
kubectl rollout history deployment hello-world --revision=2
kubectl rollout history deployment hello-world --revision=3


#Let's undo our rollout to revision 2, which is our v2 container.
kubectl rollout undo deployment hello-world --to-revision=2
kubectl rollout status deployment hello-world
echo $?


#We're back to Desired of 10 and 2 new Pods where deployed using the previous Deployment Replicas/Container Image.
kubectl get pods


#Let's delete this Deployment and start over with a new Deployment.
kubectl delete deployment hello-world
kubectl delete service hello-world


```
<img width="1370" height="724" alt="image" src="https://github.com/user-attachments/assets/dc942f91-92db-4fda-8dbb-31a800303ed9" />
<img width="1347" height="671" alt="image" src="https://github.com/user-attachments/assets/e0f186b0-cc79-42fc-97ba-7b8fe9ebc5ef" />

```
###Examine deployment.probes-1.yaml, review strategy settings, revisionhistory, and readinessProbe settings###

####QUICKLY run these two commands or as one block.####
#Demo 3 - Controlling the rate and update strategy of a Deployment update.
#Let's deploy a Deployment with Readiness Probes
kubectl apply -f deployment.probes-1.yaml --record


#Available is still 0 because of our Readiness Probe's initialDelaySeconds is 10 seconds.
#Also, look there's a new annotaion for our change-cause
#And check the Conditions, 
#   Progressing   True    NewReplicaSetCreated or ReplicaSetUpdated - depending on the state.
#   Available     False   MinimumReplicasUnavailable
kubectl describe deployment hello-world
####################################################

#Check again, Replicas and Conditions, all Pods should be online and ready.
#   Available      True    MinimumReplicasAvailable
#   Progressing    True    NewReplicaSetAvailable
kubectl describe deployment hello-world


#Let's update from v1 to v2 with Readiness Probes Controlling the rollout, and record our rollout
diff deployment.probes-1.yaml deployment.probes-2.yaml
kubectl apply -f deployment.probes-2.yaml --record


#Lots of pods, most are not ready yet, but progressing...how do we know it's progressing?
kubectl get replicaset


#Check again, Replicas and Conditions. 
#Progressing is now ReplicaSetUpdated, will change to NewReplicaSetAvailable when it's Ready
#NewReplicaSet is THIS current RS, OldReplicaSet is populated during a Rollout, otherwise it's <None>
#We used the update strategy settings of max unavailable and max surge to slow this rollout down.
#This update takes about a minute to rollout
kubectl describe deployment hello-world


#Let's update again, but I'm not going to tell you what I changed, we're going to troubleshoot it together
kubectl apply -f deployment.probes-3.yaml --record


#We stall at 4 out of 20 replicas updated...let's look
kubectl rollout status deployment hello-world


#Let's check the status of the Deployment, Replicas and Conditions, 
#22 total (20 original + 2 max surge)
#18 available (20 original - 2 (10%) in the old RS)
#4 Unavailable, (only 2 pods in the old RS are offline, 4 in the new RS are not READY)
#  Available      True    MinimumReplicasAvailable
#  Progressing    True    ReplicaSetUpdated 
kubectl describe deployment hello-world


#Let's look at our ReplicaSets, no Pods in the new RS hello-world-89579fd85 are READY, but 4 our deployed.
#That RS with Desired 0 is from our V1 deployment, 18 is from our V2 deployment.
kubectl get replicaset


#Ready...that sounds familiar, let's check the deployment again
#What keeps a pod from reporting ready? A Readiness Probe...see that Readiness Probe, wrong port ;)
kubectl describe deployment hello-world
 

#We can read the Deployment's rollout history, and see our CHANGE-CAUSE annotations
kubectl rollout history deployment hello-world
```
<img width="1370" height="719" alt="image" src="https://github.com/user-attachments/assets/21680e95-8d27-4a70-b2c1-3a7f213ac7da" />

```
#Let's rollback to revision 2 to undo that change...
kubectl rollout history deployment hello-world --revision=3
kubectl rollout history deployment hello-world --revision=2
kubectl rollout undo deployment hello-world --to-revision=2


#And check out our deployment to see if we get 20 Ready replicas
kubectl describe deployment | head
kubectl get deployment

#Let's clean up
kubectl delete deployment hello-world
kubectl delete service hello-world


```
<img width="1372" height="736" alt="image" src="https://github.com/user-attachments/assets/9384dc53-5bd4-4626-8105-e4a33176acbc" />

```
#Restarting a deployment. Create a fresh deployment so we have easier to read logs.
kubectl create deployment hello-world --image=psk8s.azurecr.io/hello-app:1.0 --replicas=5


#Check the status of the deployment
kubectl get deployment


#Check the status of the pods...take note of the pod template hash in the NAME and the AGE
kubectl get pods 


#Let's restart a deployment
kubectl rollout restart deployment hello-world 


#You get a new replicaset and the pods in the old replicaset are shutdown and the new replicaset are started up
kubectl describe deployment hello-world


#All new pods in the replicaset 
kubectl get pods 


#clean up from this demo
kubectl delete deployment hello-world

```
### Creating and Scaling a Deployment:
```
ssh aen@c1-cp1
cd ~/content/course/03/demos/

#Demo 1 - Creating and Scaling a Deployment.
#Let's start off imperatively creating a deployment and scaling it...
#To create a deployment, we need kubectl create deployment
kubectl create deployment hello-world --image=psk8s.azurecr.io/hello-app:1.0


#Check out the status of our deployment, we get 1 Replica
kubectl get deployment hello-world


#Let's scale our deployment from 1 to 10 replicas
kubectl scale deployment hello-world --replicas=10


#Check out the status of our deployment, we get 10 Replicas
kubectl get deployment hello-world


#But we're going to want to use declarative deployments in yaml, so let's delete this.
kubectl delete deployment hello-world


#Deploy our Deployment via yaml, look inside deployment.yaml first.
kubectl apply -f deployment.yaml 


#Check the status of our deployment
kubectl get deployment hello-world


#Apply a modified yaml file scaling from 10 to 20 replicas.
diff deployment.yaml deployment.20replicas.yaml
kubectl apply -f deployment.20replicas.yaml


#Check the status of the deployment
kubectl get deployment hello-world


#Check out the events...the replicaset is scaled to 20
kubectl describe deployment 


#Clean up from our demos
kubectl delete deployment hello-world
kubectl delete service hello-world
```



