<img width="1373" height="215" alt="image" src="https://github.com/user-attachments/assets/fb2a1b41-35f8-4d72-937a-1b34331aabfd" /><img width="1377" height="214" alt="image" src="https://github.com/user-attachments/assets/f90a2772-517d-46c1-8e35-72a7a76d274c" /><img width="950" height="221" alt="image" src="https://github.com/user-attachments/assets/448f5c94-6b26-416a-b9d3-62892c89f18d" />---
layout: default
title: "Ansible desired state – controller configuration as code"
permalink: /ansible-desired-state.html
---

[← Back to index](/)


# Running and managing pods

## Pods:

```
ssh aen@c1-cp1
cd ~/content/course/04/demos/

#Start up kubectl get events --watch and background it.
kubectl get events --watch &

#Create a pod...we can see the scheduling, container pulling and container starting.
kubectl apply -f pod.yaml

#Start a Deployment with 1 replica. We see the deployment created, scaling the replica set and the replica set starting the first pod
kubectl apply -f deployment.yaml

#Scale a Deployment to 2 replicas. We see the scaling the replica set and the replica set starting the second pod
kubectl scale deployment hello-world --replicas=2

#We start off with the replica set scaling to 1, then  Pod deletion, then the Pod killing the container 
kubectl scale deployment hello-world --replicas=1

kubectl get pods

#Let's use exec a command inside our container, we can see the GET and POST API requests through the API server to reach the pod.
kubectl -v 6 exec -it PASTE_POD_NAME_HERE -- /bin/sh
ps
exit

#Let's look at the running container/pod from the process level on a Node.
kubectl get pods -o wide
ssh aen@c1-node[xx]
ps -aux | grep hello-app
exit

#Now, let's access our Pod's application directly, without a service and also off the Pod network.
kubectl port-forward PASTE_POD_NAME_HERE 80:8080

#Let's do it again, but this time with a non-priviledged port
kubectl port-forward PASTE_POD_NAME_HERE 8080:8080 &

#We can point curl to localhost, and kubectl port-forward will send the traffic through the API server to the Pod
curl http://localhost:8080

#Kill our port forward session.
fg
ctrl+c

kubectl delete deployment hello-world
kubectl delete pod hello-world-pod

#Kill off the kubectl get events
fg
ctrl+c


#Static pods
#Quickly create a Pod manifest using kubectl run with dry-run and -o yaml...copy that into your clipboard
kubectl run hello-world --image=psk8s.azurecr.io/hello-app:2.0 --dry-run=client -o yaml --port=8080 

#Log into a node...
ssh aen@c1-node1

#Find the staticPodPath:
sudo cat /var/lib/kubelet/config.yaml


#Create a Pod manifest in the staticPodPath...paste in the manifest we created above
sudo vi /etc/kubernetes/manifests/mypod.yaml
ls /etc/kubernetes/manifests

#Log out of c1-node1 and back onto c1-cp1
exit

#Get a listing of pods...the pods name is podname + node name
kubectl get pods -o wide


#Try to delete the pod...
kubectl delete pod hello-world-c1-node1


#Its still there...
kubectl get pods 


#Remove the static pod manifest on the node
ssh aen@c1-node1
sudo rm /etc/kubernetes/manifests/mypod.yaml

#Log out of c1-node1 and back onto c1-cp1
exit

#The pod is now gone.
kubectl get pods 

```
## Container:
```
ssh aen@c1-cp1
cd ~/content/course/04/demos/


#Use a watch to watch the progress
#Each init container run to completion then the app container will start and the Pod status changes to Running.
kubectl get pods --watch &


#Create the Pod with 2 init containers...
#each init container will be processed serially until completion before the main application container is started
kubectl apply -f init-containers.yaml


#Review the Init-Containers section and you will see each init container state is 'Teminated and Completed' and the main app container is Running
#Looking at Events...you should see each init container starting, serially...
#and then the application container starting last once the others have completed
kubectl describe pods init-containers | more 


#Delete the pod
kubectl delete -f init-containers.yaml

#Kill the watch
fg
ctrl+c

```
## Multicontainer pods:
```
ssh aen@c1-cp1
cd ~/content/course/04/demos/

#Review the code for a multi-container pod, the volume webcontent is an emptyDir...essentially a temporary file system.
#This is mounted in the containers at mountPath, in two different locations inside the container.
#As producer writes data, consumer can see it immediatly since it's a shared file system.
more multicontainer-pod.yaml

#Let's create our multi-container Pod.
kubectl apply -f multicontainer-pod.yaml

#Let's connect to our Pod...not specifying a name defaults to the first container in the configuration
kubectl exec -it multicontainer-pod -- /bin/sh
ls -la /var/log
tail /var/log/index.html
exit

#Let's specify a container name and access the consumer container in our Pod
kubectl exec -it multicontainer-pod --container consumer -- /bin/sh
ls -la /usr/share/nginx/html
tail /usr/share/nginx/html/index.html
exit

#This application listens on port 80, we'll forward from 8080->80
kubectl port-forward multicontainer-pod 8080:80 &
curl http://localhost:8080

#Kill our port-forward.
fg
ctrl+c

kubectl delete pod multicontainer-pod
```

## Pod lifecycle:
```
ssh aen@c1-cp1
cd ~/content/course/04/demos/

#Start up kubectl get events --watch and background it.
kubectl get events --watch &
clear

#Create a pod...we can see the scheduling, container pulling and container starting.
kubectl apply -f pod.yaml

#We've used exec to launch a shell before, but we can use it to launch ANY program inside a container.
#Let's use killall to kill the hello-app process inside our container
kubectl exec -it hello-world-pod -- /bin/sh 
ps
exit

#We still have our kubectl get events running in the background, so we see if re-create the container automatically.
kubectl exec -it hello-world-pod -- /usr/bin/killall hello-app

#Our restart count increased by 1 after the container needed to be restarted.
kubectl get pods

#Look at Containers->State, Last State, Reason, Exit Code, Restart Count and Events
#This is because the container restart policy is Always by default
kubectl describe pod hello-world-pod

#Cleanup time
kubectl delete pod hello-world-pod

#Kill our watch
fg
ctrl+c

#Remember...we can ask the API server what it knows about an object, in this case our restartPolicy
kubectl explain pods.spec.restartPolicy

#Create our pods with the restart policy
more pod-restart-policy.yaml
kubectl apply -f pod-restart-policy.yaml

#Check to ensure both pods are up and running, we can see the restarts is 0
kubectl get pods 

#Let's kill our apps in both our pods and see how the container restart policy reacts
kubectl exec -it hello-world-never-pod -- /usr/bin/killall hello-app
kubectl get pods

#Review container state, reason, exit code, ready and contitions Ready, ContainerReady
kubectl describe pod hello-world-never-pod

#let's use killall to terminate the process inside our container. 
kubectl exec -it hello-world-onfailure-pod -- /usr/bin/killall hello-app

#We'll see 1 restart on the pod with the OnFailure restart policy.
kubectl get pods 

#Let's kill our app again, with the same signal.
kubectl exec -it hello-world-onfailure-pod -- /usr/bin/killall hello-app

#Check its status, which is now Error too...why? The backoff.
kubectl get pods 

#Let's check the events, we hit the backoff loop. 10 second wait. Then it will restart.
#Also check out State and Last State.
kubectl describe pod hello-world-onfailure-pod 

#Check its status, should be Running...after the Backoff timer expires.
kubectl get pods 

#Now let's look at our Pod statuses
kubectl delete pod hello-world-never-pod
kubectl delete pod hello-world-onfailure-pod

```

## Probes:
```
ssh aen@c1-cp1
cd ~/content/course/04/demos/

#Start a watch to see the events associated with our probes.
kubectl get events --watch &
clear

#We have a single container pod app, in a Deployment that has both a liveness probe and a readiness probe
more container-probes.yaml

#Send in our deployment, after 10 seconds, our liveness and readiness probes will fail.
#The liveness probe will kill the current pod, and recreate one.
kubectl apply -f container-probes.yaml

#kill our watch
fg
ctrl+c

#We can see that our container isn't ready 0/1 and it's Restarts are increasing.
kubectl get pods

#Let's figure out what's wrong
#1. We can see in the events. The Liveness and Readiness probe failures.
#2. Under Containers, Liveness and Readiness, we can see the current configuration. And the current probe configuration. Both are pointing to 8081.
#3. Under Containers, Ready and Container Contidtions, we can see that the container isn't ready.
#4. Our Container Port is 8080, that's what we want our probes, probings. 
kubectl describe pods

#So let's go ahead and change the probes to 8080
vi container-probes.yaml

#And send that change into the API Server for this deployment.
kubectl apply -f container-probes.yaml

#Confirm our probes are pointing to the correct container port now, which is 8080.
kubectl describe pods

#Let's check our status, a couple of things happened there.
#1. Our Deployment ReplicaSet created a NEW Pod, when we pushed in the new deployment configuration.
#2. It's not immediately ready because of our initialDelaySeconds which is 10 seconds.
#3. If we wait long enough, the livenessProbe will kill the original Pod and it will go away.
#4. Leaving us with the one pod in our Deployment's ReplicaSet
kubectl get pods 

kubectl delete deployment hello-world



#Let's start up a watch on kubectl get events
kubectl get events --watch &
clear

#Create our deployment with a faulty startup probe...
#You'll see failures since the startup probe is looking for 8081...
#but you won't see the liveness or readiness probes executed
#The container will be restarted after 1 failures failureThreshold defaults to 3...this can take up to 30 seconds
#The container restart policy default is Always...so it will restart.
kubectl apply -f container-probes-startup.yaml


#Do you see any container restarts?  You should see 1.
kubectl get pods


#Change the startup probe from 8081 to 8080
kubectl apply -f container-probes-startup.yaml


#Our pod should be up and Ready now.
kubectl get pods

fg
ctrl+c

kubectl delete -f container-probes-startup.yaml
```


