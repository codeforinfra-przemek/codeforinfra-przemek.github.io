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


