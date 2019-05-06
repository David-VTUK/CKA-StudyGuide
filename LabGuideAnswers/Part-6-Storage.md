
**<span style="text-decoration:underline;">CKA Lab Part 6 - Storage</span>**

**<span style="text-decoration:underline;">Lab 1 - Create a PV</span>**

Create a PV in your environment 1GB in size. Pick a type suitable for your lab:



*   HostPath (for single nodes)
*   NFS
*   Azure Disk
*   Etc

Ensure it is capable of being both read and written to.


```
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
 storageClassName: slow
 hostPath:
   path: /tmp
```


**<span style="text-decoration:underline;">Lab 2 - Modify PV</span>**

Change the access mode of the PV in lab 1 to “ReadOnlyMany”

**Change:**


```
 accessModes:
   - ReadWriteOnce

To:

 accessModes:
   - ReadOnlyMany
```


**<span style="text-decoration:underline;">Lab 3 - Create a PVC</span>**

Create a claim to the persistent volume you created in Lab 1, for 512MB. Choose an applicable access mode based on the state of the persistent volume


```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
 name: myclaim
spec:
 accessModes:
   - ReadOnlyMany
 volumeMode: Filesystem
 resources:
   requests:
     storage: 512Mi
 storageClassName: slow
```


**<span style="text-decoration:underline;">Lab 4 - Consume storage</span>**

Configure a pod to leverage the PVC and mount it to /mnt/readonly


```
apiVersion: v1
kind: Pod
metadata:
 name: pod-with-ro-mount
spec:
 volumes:
   - name: readonlyvolume
     persistentVolumeClaim:
      claimName: readonlyclaim
 containers:
 - name: busybox
   image: busybox
   args:
   - sleep
   - "1000000"
   volumeMounts:
     - mountPath: "/mnt/readonly"
       name: readonlyvolume
