---
layout: default
title: "Ansible desired state – controller configuration as code"
permalink: /ansible-desired-state.html
---

[← Back to index](/)


# Configuring and Managing Kubernetes Storage and Scheduling



## Implement Kubernetes Persistent Storage Concepts and Management:

<img width="1357" height="744" alt="image" src="https://github.com/user-attachments/assets/b6238601-3dcc-4269-9048-d66b744f2acb" />

```
cat > pv-demo.yaml <<'EOF'
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-demo
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  hostPath:
    path: "/mnt/data"
EOF

kubectl apply -f pv-demo.yaml
```
```
cat > pvc-demo.yaml <<'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-demo
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  resources:
    requests:
      storage: 1Gi
EOF

kubectl apply -f pvc-demo.yaml
```

```
kubectl get pv
kubectl get pvc
kubectl describe pv pv-demo
kubectl describe pvc pvc-demo
```

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
spec:
  containers:
    - name: demo-container
      image: busybox
      command: ["sleep", "3600"]
      volumeMounts:
        - name: demo-storage
          mountPath: /data
  volumes:
    - name: demo-storage
      persistentVolumeClaim:
        claimName: pvc-demo
```

### Storage classes and provisioning:

<img width="1349" height="751" alt="image" src="https://github.com/user-attachments/assets/66ba4f63-8ad8-4b66-bda6-5d4e11b37b3c" />
<img width="1377" height="744" alt="image" src="https://github.com/user-attachments/assets/0ce57e62-b90a-493c-a085-d036b7788658" />
<img width="997" height="638" alt="image" src="https://github.com/user-attachments/assets/d733eed8-2d00-4ca7-b393-b2e316bb7778" />
<img width="975" height="539" alt="image" src="https://github.com/user-attachments/assets/0039ace9-35da-4427-a421-26f3bb1e2ce1" />

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: demo-storage
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Delete
volumeBindingMode: Immediate

kubectl apply -f storageclass-demo.yaml
kubectl get storageclass
kubectl describe storageclass demo-storage
```
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: demo-storage
  resources:
    requests:
      storage: 1Gi

kubectl apply -f demo-pvc.yaml
kubectl get pvc
kubectl describe pvc demo-pvc
```
## Implement Advanced Pod Scheduling Techniques:

<img width="1354" height="757" alt="image" src="https://github.com/user-attachments/assets/9bc929fb-c014-4140-952c-382b71f11aca" />
<img width="1366" height="786" alt="image" src="https://github.com/user-attachments/assets/21a5437e-8619-4480-b2cd-fa24cb83ac68" />

```
kubectl taint nodes desktop-worker dedicated=analytics:NoSchedule
kubectl describe node desktop-worker | grep Taints -A1

kubectl taint nodes desktop-worker dedicated=analytics:NoSchedule-
kubectl get nodes --show-labels
kubectl label node desktop-worker workload=analytics
kubectl taint nodes desktop-worker dedicated=analytics:NoSchedule
kubectl label node desktop-worker workload=analytics

apiVersion: apps/v1
kind: Deployment
metadata:
  name: analytics-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: analytics-app
  template:
    metadata:
      labels:
        app: analytics-app
    spec:
      nodeSelector:
        workload: analytics
      tolerations:
      - key: "dedicated"
        operator: "Equal"
        value: "analytics"
        effect: "NoSchedule"
      containers:
      - name: app
        image: nginx

kubectl apply -f analytics-app.yaml
kubectl get pods -o wide
kubectl describe pod <pod-name>

apiVersion: v1
kind: Pod
metadata:
  name: analytics-test
spec:
  nodeSelector:
    workload: analytics
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "analytics"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx

kubectl apply -f analytics-test.yaml
```
<img width="1358" height="731" alt="image" src="https://github.com/user-attachments/assets/f7a80b61-1194-4c08-9ab6-82c7fe048dba" />
<img width="1358" height="745" alt="image" src="https://github.com/user-attachments/assets/cd343d1c-21ee-4663-bc47-b30455ffaf18" />

```
KUBERNETES QUICK SUMMARY

1) NODE AFFINITY
Used on Pods to control which nodes they prefer or require.

Required node affinity:
- Pod must run only on nodes matching the label.

Example:
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: workload
          operator: In
          values:
          - analytics

Preferred node affinity:
- Scheduler tries to place pod there, but it is not mandatory.

Example:
affinity:
  nodeAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      preference:
        matchExpressions:
        - key: workload
          operator: In
          values:
          - analytics

Useful node label commands:
kubectl get nodes --show-labels
kubectl label node desktop-worker workload=analytics
kubectl label node desktop-worker workload-

--------------------------------------------------

2) POD AFFINITY
Used on Pods to schedule close to other pods.

Pod affinity:
- Place this pod on a node where matching pods already exist.

Example:
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - backend
      topologyKey: kubernetes.io/hostname

Meaning:
- Schedule this pod onto a node that already has a pod with label app=backend.

Pod anti-affinity:
- Keep this pod away from matching pods.

Example:
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - frontend
      topologyKey: kubernetes.io/hostname

Meaning:
- Do not schedule this pod on the same node as pods with app=frontend.

Useful pod commands:
kubectl get pods -o wide
kubectl describe pod <pod-name>

--------------------------------------------------

3) RESOURCE REQUESTS AND LIMITS
Set on containers.

requests:
- Minimum resources Kubernetes reserves for the container
- Scheduler uses requests to decide placement

limits:
- Maximum resources container can use

Example:
resources:
  requests:
    cpu: "250m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"

Meaning:
- requests.cpu: reserve 0.25 CPU
- limits.cpu: container can use up to 0.5 CPU
- requests.memory: reserve 128Mi
- limits.memory: container can use up to 256Mi

Notes:
- If memory exceeds limit, container may be killed (OOMKilled)
- CPU over limit is throttled
- Scheduler places pod based mostly on requests, not limits

Useful commands:
kubectl top nodes
kubectl top pods
kubectl describe pod <pod-name>

--------------------------------------------------

4) FULL EXAMPLE YAML
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
  labels:
    app: demo
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: workload
            operator: In
            values:
            - analytics
    podAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 50
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - backend
          topologyKey: kubernetes.io/hostname
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 50
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - frontend
          topologyKey: kubernetes.io/hostname
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: "250m"
        memory: "128Mi"
      limits:
        cpu: "500m"
        memory: "256Mi"

--------------------------------------------------

5) APPLY AND CHECK
kubectl apply -f demo-pod.yaml
kubectl get pods -o wide
kubectl describe pod demo-pod
kubectl top pod demo-pod

--------------------------------------------------

6) SIMPLE MEMORY TRICK
- nodeAffinity = choose nodes
- podAffinity = stay close to some pods
- podAntiAffinity = stay away from some pods
- requests = guaranteed reservation for scheduling
- limits = max allowed usage
```



