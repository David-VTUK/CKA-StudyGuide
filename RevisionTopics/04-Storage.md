# Understand storage classes, persistent volumes

A StorageClass provides a way for administrators to describe the "classes" of storage they offer. Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary policies determined by the cluster administrators. Kubernetes itself is unopinionated about what classes represent. This concept is sometimes called "profiles" in other storage systems.

An example of a storage class is below:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
allowVolumeExpansion: true
mountOptions:
  - debug
volumeBindingMode: Immediate
```

Key declarations are:

* `provisioner` : Determine which volume plugin to use. This usually matches to a cloud provider and a specific storage service that it offers. In this example AWS Elastic Block Store

* `parameters` : Describe characteristics of this storage class in context of the underlying provisioner. In this cample, the `type` is `gp2` which, in AWS lingo means General Purpose SSD. Other types include `IO1` (Provisioned IOPS), `ST1` (Throughput Optimised) and `STC` (Cold Storage).

A `storageclass` object by itself defines what storage is provisioned *when it is invoked*. By itself, it does nothing.

A `persistentvolume` object can be used to request storage from a `storageclass` and is typically part of a pod manifest

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: standard
```

# Understand volume mode, access modes and reclaim policies for volumes

## Volume modes:

Only two exist:

* `block` - Mounted to a pod as a raw block device *without* a filesystem. The Pod / application needs to understand how to deal with raw block devices. Presenting storage can yield better performance, at the expense of complexity.
  

* `filesystem` - Mounted inside a pods filesystem inside a directory. If the volume is backed by a block device with no filesystem, Kubernetes will create one. Compared to `block` devices, this method offers the highest compatibility, at the expense of performance.

## Access Modes

Three options exist:

*  `ReadWriteOnce` – The volume can be mounted as read-write by a single node
*  `ReadOnlyMany` – The volume can be mounted read-only by many nodes
*  `ReadWriteMany` – The volume can be mounted as read-write by many nodes

# Understand persistent volume claims primitive

A `PersistentVolume` can be thought of as storage provisioned by an administrator. Think of this as pre-allocation

A `PersistentVolumeClaim` can be thought of as storage requested by a user/workload. 

When the user creates PVC to request storage, Kubernetes will try to match that PVC with a pre-allocated PV. If a match can be found, the PVC will be bound to the PV, and the user will start to use that pre-allocated piece of storage.


# Know how to configure applications with persistent storage

When not using storageclasses there are a number of steps:

Create the PV: 

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

Create the workload to levage this:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```

If leveraging a storageclass, a `persistentvolume` object is not required, we just need a `persistentvolumeclaim`


```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: myStorageClass
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```


