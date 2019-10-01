
**<span style="text-decoration:underline;">CKA Lab Part 3 - Application Lifecycle Management</span>**

**<span style="text-decoration:underline;">Lab 1 - Perform rolling updates on a deployment</span>**

Apply the following yaml file  [https://raw.githubusercontent.com/David-VTUK/CKAExampleYaml/master/nginx-svc-and-deployment.yaml](https://raw.githubusercontent.com/David-VTUK/CKAExampleYaml/master/nginx-svc-and-deployment.yaml)

Update this deployment to leverage the nginx container version 1.7.11. Ensure that --record=true has been used.

**<span style="text-decoration:underline;">Lab 2 - Change the update strategy for a deployment</span>**

Using the YAML file from Lab 1, amend it so that:



*   Strategy is “Rolling Update”
*   Max Surge is “1”
*   Max Unavailable is “1”

**<span style="text-decoration:underline;">Lab 3 - Perform a rollback on a deployment</span>**

Rollback the changes that were implemented from Lab 1.

**<span style="text-decoration:underline;">Lab 4 - Scale a deployment</span>**

Scale the deployment from the first lab exercise to leverage 6 pods.

**<span style="text-decoration:underline;">Lab 5 - Create and run a Job</span>**

Spec and execute a job that:



*   Leverages the “perl” image
*   Calculates pi to 2000 places

Note, use the command command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"] in the pod manifest

The command above will output to stdout on the container, therefore inspect the output

**<span style="text-decoration:underline;">Lab 6 - Create and use a Config Map</span>**

Create two texts files in /tmp/

db_h.txt with the contents “database_host”

db_p.txt with the contents “database_port”

Create a configmap called “db-connection” from the above two files.

Create a nginx pod which leverages these values as environment variables “db_h” and “db_p”

**<span style="text-decoration:underline;">Lab 7 - Create and use Secrets</span>**

Create a secret called “db-credentials” directly from the CLI with the following key:value pair.

db-username : dbuser

db-password : dbpassword

Create a pod to leverage these as environment variables.

**<span style="text-decoration:underline;">Lab 8 - Configure a pod with specific environment variables</span>**

Create a pod that has two environment variables configured:

Variable1 = somevalue

Variable2 = someothervalue
