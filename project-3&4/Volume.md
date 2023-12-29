# Kubernetes Persistent Volume and Persistent Volume Claim

This guide will help you create a Persistent Volume (PV), Persistent Volume Claim (PVC), and two pods (`busybox-one` and `busybox-two`) that use the PVC.

## Step 1: Create a Persistent Volume

```bash
cat<<'EOF' > pv.yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: myvolume
spec:
  storageClassName: local-path
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  hostPath:
    path: /etc/foo
EOF
```
```bash
kubectl apply -f pv.yaml
```

## Step 2: Create a Persistent Volume Claim

```bash
cat<<'EOF' > pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mypvc
spec:
  storageClassName: local-path
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
EOF
```
```
kubectl apply -f pvc.yaml
```

## Step 3: Create Pods

```bash
cat<<'EOF'> busybox-one.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: busybox-one
  name: busybox-one
spec:

  volumes:
  - name:  my-vol # has to match volumeMounts.name
    persistentVolumeClaim:
      claimName: mypvc

  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    name: busybox-one

    volumeMounts:
    - name: my-vol # has to match volumes.name
      mountPath: /etc/foo
EOF
```

```bash
kubectl apply -f busybox-one.yaml 
kubectl exec busybox-one -- ls /etc/foo
```

<br>

```bash
cat<<'EOF'> busybox-two.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: busybox-two
  name: busybox-two
spec:

  volumes:
  - name:  my-vol
    persistentVolumeClaim:
      claimName: mypvc

  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    name: busybox-two

    volumeMounts:
    - name: my-vol
      mountPath: /etc/foo
EOF
```

```bash
kubectl apply -f busybox-two.yaml
kubectl exec busybox-two -- ls /etc/foo
```
