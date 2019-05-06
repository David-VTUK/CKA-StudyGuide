

**<span style="text-decoration:underline;">CKA Lab Part 9 - Networking</span>**

**<span style="text-decoration:underline;">Lab 1 - Create a ClusterIP</span>**



*   Create a deployment consisting of three nginx containers.
*   Create a service of type “Cluster IP” which is exposed on port 8080 that facilitates connections to the aforementioned deployment, which listens on port 80
*   Test the connectivity by spinning up a pod

**<span style="text-decoration:underline;">Lab 2 - Create a LoadBalancer </span>**



*   Note, for this to work your environment must be able to provision load balancers (GKE, AKE, etc)
*   Create a deployment consisting of three nginx containers.
*   Create a service of type “Load Balancer” which is exposed on port 8080 that facilitates connections to the aforementioned deployment, which listens on port 80.
*   Test the connectivity by accessing the website

**<span style="text-decoration:underline;">Lab 3 - Create a NodePort</span>**



*   Create a deployment consisting of three nginx containers.
*   Create a service of type “NodePort” which is exposed on port 30010 that facilitates connections to the aforementioned deployment, which listens on port 80.
*   Test the connectivity by spinning up a pod

**<span style="text-decoration:underline;">Lab 4 - Create a Ingress Resource</span>**

Create a ingress resource “website-ingress” that directs traffic to the following conditions:



*   Base domain website.com
    *   Default backend service is “default-service” on port 80
    *   /backend directs traffic to “backend-service” on port 443
    *   /test directs traffic to “test-service” on port 8000
