# Lab Exercises for Storage

# Exercise 0 - Setup

* Prepare a cluster (Single node, kubedm, k3s, etc)
* Open browser tabs to https://kubernetes.io/docs/, https://github.com/kubernetes/ and  https://kubernetes.io/blog/ (these are permitted as per [the current guidelines](https://docs.linuxfoundation.org/tc-docs/certification/certification-resources-allowed#certified-kubernetes-administrator-cka-and-cerified-kubernetes-application-developer-ckad))

# Exercise 1 - Understand storage classes, persistent volumes

Note: Leverage the docs to help you. It's unlikey you will need to recall the nuances from memory - https://kubernetes.io/docs/concepts/storage/storage-classes/

1. Create a `storageclass` object named `aws-ebs` with the following specification:
    1. The `provisioner` should be `kubernetes.io/aws-ebs`
    2. The `type` is `io1`
    3. The `reclaimPolicy` is `Delete`
    4. `allowVolumeExpansion` set to `false`
2. Create a `persistentvolume` based on this storage class
    1. Note, it will not initialize unless AWS integration is configured, but the objective is to get the YAML correct.
    
<details><summary>Answer</summary>

Apply the following YAML to create the Storage Class

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: aws-ebs
provisioner: kubernetes.io/aws-ebs
parameters:
  type: io1
reclaimPolicy: Delete
allowVolumeExpansion: false
mountOptions:
  - debug
volumeBindingMode: Immediate
```

Apply the following YAML to create a Persistent Volume based on the Storage Class

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-test
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: aws-ebs
  awsElasticBlockStore:
    fsType: "ext4"
    volumeID: "vol-id" 
```

validate with: 

```shell
kubectl get pv                                                                         
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv-test   5Gi        RWO            Recycle          Available           aws-ebs                 5
```

</details>

# Exercise 2 - Know how to configure applications with persistent storage

In this exercise, we will **not** be using `storageClass` objects

1. Create a `persistentVolume` object of type `hostPath` with the following parameters:
    1. 1GB Capacity
    2. Path on the host is /tmp
    3. `storageClassName` is Manual
    4. `accessModes` is `ReadWriteOnce`
2. Create a `persistentVolumeClaim` to the aforementioned `persistentVolume`
3. Create a `pod` workload to leverage this `persistentVolumeClaim

<details><summary>Answer</summary>

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
 name: pv-hostpath-1gb
spec:
 capacity:
   storage: 1Gi
 volumeMode: Filesystem
 accessModes:
   - ReadWriteOnce
 persistentVolumeReclaimPolicy: Recycle
 storageClassName: manual
 hostPath:
   path: /tmp
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
 name: pvc-hostpath-claim
spec:
 accessModes:
   - ReadWriteOnce
 volumeMode: Filesystem
 resources:
   requests:
     storage: 512Mi
 storageClassName: manual
---
apiVersion: v1
kind: Pod
metadata:
 name: pod-with-pvc
spec:
 volumes:
   - name: myvol
     persistentVolumeClaim:
      claimName: pvc-hostpath-claim
 containers:
 - name: busybox
   image: busybox
   args:
   - sleep
   - "1000000"
   volumeMounts:
     - mountPath: "/mnt/readonly"
       name: myvol
```

Validate with:

```shell
kubectl get po,pv,pvc
NAME               READY   STATUS    RESTARTS   AGE
pod/pod-with-pvc   1/1     Running   0          2m7s

NAME                               CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                        STORAGECLASS   REASON   AGE
persistentvolume/pv-hostpath-1gb   1Gi        RWO            Recycle          Bound    default/pvc-hostpath-claim   manual                  2m7s

NAME                                       STATUS   VOLUME            CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/pvc-hostpath-claim   Bound    pv-hostpath-1gb   1Gi        RWO            manual         2m7s

```

</details>