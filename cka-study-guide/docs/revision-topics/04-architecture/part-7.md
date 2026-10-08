# Understand extension interfaces (CNI, CSI, CRI, etc.)

Kubernetes relies on a modular, plugin-driven architecture powered by standardised extension interfaces. These are known as the CRI (Container Runtime Interface), CNI (Container Network Interface), CSI (Container Storage Interface), and CPI (Cloud Provider Interface) to decouple core orchestration logic from underlying infrastructure.

These interfaces allow Kubernetes components like `kubelet` and the control plane to seamlessly interact with third-party container runtimes, networking fabrics, storage systems, and cloud environments.

This plug-and-play design ensures Kubernetes remains vendor-neutral, highly customizable, and capable of operating consistently across any environment, from bare-metal datacenters to managed cloud platforms.

We can visualise this as so:

```mermaid
graph TD
    %% Core Kubernetes Components
    subgraph K8S ["Kubernetes Core"]
        direction LR
        APISERVER["kube-apiserver<br/>(Control Plane)"]
        CCM["cloud-controller-manager<br/>(Control Plane)"]
        KUBELET["kubelet<br/>(Node Agent)"]
    end

    %% Container Runtime Interface
    subgraph CRI ["CRI (Container Runtime Interface)"]
        CRI_API["gRPC Interface<br/>(RuntimeService & ImageService)"]
        RUNTIMES["Container Runtimes<br/>(containerd / CRI-O)"]
        
        CRI_API --> RUNTIMES
    end

    %% Container Network Interface
    subgraph CNI ["CNI (Container Network Interface)"]
        CNI_SPEC["CNI Exec Plugin Spec<br/>(ADD / DEL / CHECK)"]
        NET_PLUGINS["Network Plugins<br/>(Calico / Cilium / Flannel)"]
        
        CNI_SPEC --> NET_PLUGINS
    end

    %% Container Storage Interface (Split into Controller & Node responsibilities)
    subgraph CSI ["CSI (Container Storage Interface)"]
        CSI_CTRL["CSI Controller Plugin<br/>(Provision & Attach)"]
        CSI_NODE["CSI Node Plugin<br/>(Format & Mount)"]
        STORAGE_DRIVERS["Storage Plugins<br/>(AWS EBS / Longhorn / Rook)"]

        CSI_CTRL --> STORAGE_DRIVERS
        CSI_NODE --> STORAGE_DRIVERS
    end

    %% Cloud Provider Interface
    subgraph CPI ["CPI (Cloud Provider Interface)"]
        CPI_API["CloudProvider Go Interface"]
        CLOUD_PROVIDERS["Cloud Infrastructure<br/>(AWS / GCP / Azure)"]
        
        CPI_API --> CLOUD_PROVIDERS
    end

    %% Explicit Interactions
    KUBELET -->|"1. Container Lifecycle"| CRI_API
    KUBELET -->|"2. Pod Network Setup"| CNI_SPEC

    APISERVER -->|"3a. Provisions & Attaches Volumes<br/>(via CSI Sidecars)"| CSI_CTRL
    KUBELET -->|"3b. Mounts Volumes to Pod"| CSI_NODE

    CCM -->|"4. Syncs Node / LB / Route Metadata"| CPI_API
```

Each plugin type has a specific purpose:

| Interface | Full Name | Primary Responsibility | Protocol | Key Examples |
| :--- | :--- | :--- | :--- | :--- |
| **CRI** | Container Runtime Interface | Manages container lifecycles, image pulling, and pod sandboxes on the node | gRPC over Unix Domain Socket | `containerd`, `CRI-O` |
| **CNI** | Container Network Interface | Configures network interfaces, IP address allocation (IPAM), and routes for Pods | Executable binary calls (`ADD`, `DEL`, `CHECK`) | Calico, Cilium, Flannel |
| **CSI** | Container Storage Interface | Handles volume lifecycle (provisioning, attaching, mounting, snapshotting) | gRPC (`Identity`, `Controller`, `Node` services) | AWS EBS CSI, Longhorn, Rook/Ceph |
| **CPI** | Cloud Provider Interface | Integrates Kubernetes control plane with cloud provider infrastructure (Node metadata, LBs, Routes) | Go interface inside `cloud-controller-manager` | AWS, GCP, Azure, OpenStack plugins |
