# Understand the primitives used to create robust, self-healing, application deployments

Deployments facilitate this by employing a reconciliation loop to check the number of deployed pods matches what’s defined in the manifest. Under the hood, deployments leverage ReplicaSets, which are primarily responsible for this feature.

## Deployments and Replicasets

Deployments were covered in part 1 but lets dig into the self-healing aspects a bit more:

```mermaid
graph TD

    %% Elements %%
    User([User]) -->|1. Applies Manifest| API[kube-apiserver]
    API <-->|Stores desired spec| ETCD[(etcd)]

    subgraph DC [Deployment Controller Loop]
        direction TB
        Watch[2. Watches for Changes] -.-> Compare{3. Desired == Actual?}
        Compare -- Yes --> Idle[Do Nothing / Watch]
        Compare -- No --> Reconcile[4. Determine Action]
    end

    API -->|Watch Event| Watch

    subgraph RS [ReplicaSet & Pod Controllers]
        direction TB
        ScaleUp[Create Pods]
        ScaleDown[Terminate Pods]
        Rolling[Progress Rollout]
    end

    Reconcile -->|Create/Update ReplicaSet| RS_Obj[ReplicaSet]:::resource
    RS_Obj -->|Reconciliation Loop| RS_Ctrl{Desired Replicas == Active Pods?}
    
    RS_Ctrl -- Lacking --> ScaleUp:::action
    RS_Ctrl -- Excess --> ScaleDown:::action
    
    Reconcile -.->|Update Template Hash| Rolling:::action

    ScaleUp --> Pods[Actual Pods]:::state
    ScaleDown --> Pods
    Rolling --> Pods

    Pods -.->|Status Updates| API

    %% Styling Application %%
    class API,ETCD,Watch,Compare,RS_Ctrl control;
    class RS_Obj,Pods resource;
    class ScaleUp,ScaleDown,Rolling,Reconcile action;
```

In a `Deployment` manifest we specify the number of `replicas`:

```yaml hl_lines="10"
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: default
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx-demo
  replicas: 3
....
```

This is what the Kubernetes API will continuously monitor. If this changes, or if the running state does not match the desired state, it takes action.

This differs from running standalone `Pods` - When a `Pod` is terminated, it's gone forever. When a replica in a `Deployment` is terminated, it is replaced.

Not all failures are obvious, a application can be running but deadlocked, or silently failing. This is where probes come in.

A probe is a way of declaring in our workloads what constitutes a Pod needing to be restarted and when it's ready to accept traffic.

## Probes

A `probe` in Kubernetes is a mechanism for determining the state of an application inside a Pod. Namely to establish:

* Is the container alive and functioning properly? (`liveness` probe)
* Is the container ready to receive traffic? (`readiness` probe)
* Is the container finished with its initial boot sequence and ready for standard health monitoring to being? (`startup` probe)

Regardless of which probe to use, they all use the same `type`:

| Probe Type              | YAML Field     | Success Criteria                                                 | Best Use Case                                              |
| ----------------------- | -------------- | -----------------------------------------------------------------|------------------------------------------------------------|
| **HTTP GET**            | `httpGet`      | Returns an HTTP 200 status code                                  | Standard web services and REST APIs.                       |
| **TCP Socket**          | `tcpSocket`    | Successfully establishes a TCP connection on the specified port. | Non-HTTP services like databases or message brokers.       |
| **Command Execution**   | `exec`         | Command completes with a return code of `0`.                     | Custom health scripts or legacy applications.              |
| **gRPC**                | `grpc`         | Application responds with a `SERVING` status.                    | gRPC-native applications                                   |

### Liveness Probe

In this example, the `kubelet` will, every 5 seconds issue a `HTTP GET` command to the container on port 80. If it passes? do nothing. If it fails, terminate it:

```yaml hl_lines="21-27"
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: default
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx-demo
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
      - name: nginx
        image: nginx:1.31.6
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 5
          failureThreshold: 3
```

We can visualise this process in the graph below:

```mermaid
graph TD
    Timer([Probe Interval Timer]) --> Kubelet

    subgraph Node [Kubernetes Node]
        direction TB
        
        Kubelet[Kubelet Executes Liveness Probe]:::kubelet
        
        subgraph Pod [Target Pod]
            Container[App Container]:::container
        end
        
        Kubelet -- "1. HTTP GET, TCP Socket, Exec, or gRPC" --> Container
        Container -. "2. Returns Status" .-> Evaluate
        
        Evaluate{3. Is Check Successful?}
        
        Evaluate -- "Yes (e.g., HTTP 200, Exit 0)" --> Healthy[Container Healthy]:::container
        
        Evaluate -- "No (e.g., Timeout, HTTP 500)" --> Unhealthy[Container Unhealthy]:::alert
        Unhealthy --> Kill[4. Kubelet Terminates Container]:::kubelet
        Kill --> Restart[5. Kubelet Restarts Container]:::kubelet
    end
    
    Healthy -. "Wait for next interval" .-> Timer
    Restart -. "Wait for initialDelaySeconds" .-> Timer
```

### Readiness Probe

In this example, we've added a liveness probe that runs `cat /var/run/nginx.pid`. If this returns `0`, we mark this container as ready to receive traffic.

```yaml hl_lines="28-34"
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: default
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx-demo
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
      - name: nginx
        image: nginx:1.31.6
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 5
          failureThreshold: 3
        readinessProbe:
          exec:
            command:
            - cat
            - /var/run/nginx.pid
          initialDelaySeconds: 5
          periodSeconds: 5
```

Which we can visualise in the graph below. Two noteworthy remarks:

1. We can mix `liveness`, `readiness` and `startup` probes together.
2. For `readiness` probes, the Pod is not terminated, it is simply omitted from having traffic routed to it

```mermaid
graph TD
    Timer([Probe Interval Timer]) --> Kubelet

    subgraph Node [Kubernetes Node]
        direction TB
        
        Kubelet[Kubelet Executes Readiness Probe]:::kubelet
        
        subgraph Pod [Target Pod]
            Container[App Container]:::container
        end
        
        Kubelet -- "1. HTTP GET, TCP Socket, Exec, or gRPC" --> Container
        Container -. "2. Returns Status" .-> Evaluate
        
        Evaluate{3. Is Check Successful?}
        
        Evaluate -- "Yes" --> Ready[Pod Marked Ready]:::container
        Evaluate -- "No" --> NotReady[Pod Marked Not Ready]:::alert
    end

    subgraph Network [Service Routing]
        Ready --> Route[Add/Keep Pod IP in Endpoints<br/>Traffic Routes to Pod]:::network
        NotReady --> Drop[Remove Pod IP from Endpoints<br/>Traffic Stops Routing to Pod]:::network
    end
    
    Route -. "Wait for next interval" .-> Timer
    Drop -. "Container stays running.<br/>Wait for next interval" .-> Timer
```

### Startup Probes

Startup probes are a way of defining a "grace period" to determine a condition that needs to be satisfied before its the target of `liveness` and `readiness` probes. Examples include database migrations or populating caches.

```yaml hl_lines="21-25"
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: default
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx-demo
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80        
        startupProbe:
          grpc:
            port: 80
          failureThreshold: 30
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 5
          failureThreshold: 3
        readinessProbe:
          exec:
            command:
            - cat
            - /var/run/nginx.pid
          initialDelaySeconds: 5
          periodSeconds: 5
```

which we can visualise in the graph below:

```mermaid
graph TD
    Start([Container Starts]) --> KubeletStartup

    subgraph Phase 1: Grace Period
        direction TB
        KubeletStartup[Kubelet Executes Startup Probe]:::kubelet
        
        EvalStartup{Is Check Successful?}
        KubeletStartup --> EvalStartup
        
        EvalStartup -- No --> Threshold{Failure Threshold Exceeded?}
        Threshold -- No --> Wait[Wait periodSeconds] -.-> KubeletStartup
        
        PausedProbes[Liveness & Readiness Probes: PAUSED]:::paused
    end

    Threshold -- Yes --> Kill[Kubelet Terminates & Restarts Container]:::alert
    Kill -.-> Start

    subgraph Phase 2: Standard Monitoring
        direction TB
        ActiveProbes[Liveness & Readiness Probes: ACTIVE]:::success
        Monitor[Continuous monitoring for the<br/>lifecycle of the container]:::kubelet
        ActiveProbes --> Monitor
    end

    EvalStartup -- "Yes (Succeeds Once)" --> ActiveProbes
```

## Restart Policies

A `restartPolicy` is a Pod specification that governs how the nodes `kubelet` should respond when a container inside that Pod terminates, crashes or fails a liveness probe.

Three options exist:

1. `always` - This is the default. The Kubelet will automatically restart the container regardless of why it stopped.
2. `onFailure` - The Kubelet will automatically restart the container only when it exists with a non-zero exit code, or terminated by a `probe`.
3. `never` - The Kubelet will not restart the container **under any circumstances**.

We define it under `.spec.restartPolicy` inside the Pod spec:

```yaml hl_lines="17"
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: default
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx-demo
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      # Added restartPolicy at the Pod spec level
      restartPolicy: Always
      containers:
      - name: nginx
        image: nginx:1.31.6
        ports:
        - containerPort: 80
```

!!! success "Exam Tip"

    Liveness and readiness `probes` provide a continuous check to establish if our application is healthy.

!!! success "Exam Tip"

    If a liveness probe fails, the Pod is terminated#

!!! success "Exam Tip"

    If a readiness probe fails, the Pod is not terminated, but its endpoint is removed from the service list.

!!! success "Exam Tip"

    Use startup probes for applications that need a grace period upon starting up (ie for initialisation, migrations, pulling down data, etc).