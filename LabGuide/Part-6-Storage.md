

**<span style="text-decoration:underline;">CKA Lab Part 6 - Storage</span>**

**<span style="text-decoration:underline;">Lab 1 - Create a PV</span>**

Create a PV in your environment 1GB in size. Pick a type suitable for your lab:



*   HostPath (for single nodes)
*   NFS
*   Azure Disk
*   Etc

Ensure it is capable of bein both read and written to.

**<span style="text-decoration:underline;">Lab 2 - Modify PV</span>**

Change the access mode of the PV in lab 1 to “ReadOnlyMany”

**<span style="text-decoration:underline;">Lab 3 - Create a PVC</span>**

Create a claim to the persistent volume you created in Lab 1, for 512MB. Choose an applicable access mode based on the state of the persistent volume

**<span style="text-decoration:underline;">Lab 4 - Consume storage</span>**

Configure a pod to leverage the PVC and mount it to /mnt/readonly
