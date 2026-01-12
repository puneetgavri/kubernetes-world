
# Kubernetes Cluster Architecture

In a Kubernetes (K8s) cluster, the architecture is organized into two main components: **the control plane** and **the worker nodes**. Together, these components form a powerful system for managing containerized applications at scale.

## Control Plane

**It is the brain of the Kubernetes cluster.**

The control plane acts as the central management hub for the entire Kubernetes cluster. It oversees the overall state and behavior of the cluster, ensuring everything is functioning as intended.

The control plane maintains the **desired state** of the cluster i.e., it continuously compares the current state of the cluster (e.g., the number of pods running) with the desired state (as defined by the user). If there are any discrepancies, it takes corrective actions, such as starting new pods or shutting down excess ones.

The control plane manages configurations for applications and services within the cluster, storing this information in a consistent manner to ensure reliability and reproducibility.

### Components of the Control Plane

#### 1. etcd
etcd is a reliable distributed key-value store that is simple, secure, and fast.  
**etcd is a reliable distributed key-value store that is simple, secure, and fast.**  
It maintains the current status, desired state, configuration, and metadata for all Kubernetes objects.

**What info it stores?**
- **Cluster Configuration and Metadata**: Nodes, Namespaces, Resource Quotas, Cluster Roles and Role Bindings (RBAC)
- **Workloads**: Pods, Deployments, ReplicaSets, DaemonSets, StatefulSets, Jobs, and CronJobs
- **Services and Networking**: Services, Endpoints, Ingress, Network Policies, and ConfigMaps
- **Secrets and Credentials**: Secrets and Service Accounts Tokens
- **Persistent Storage**: Persistent Volumes (PVs), Persistent Volume Claims (PVCs), and Storage Classes
- **Schedulers and Controllers**: Controllers, Events, Scheduler Info
- **Custom Resource Definitions (CRDs)**
- **Admission Controllers**: Admission Webhooks
- **Autoscaling Data**: Horizontal Pod Autoscalers (HPA)
- **API Server Configuration**: API Server Discovery Info
- **Federated Resources**

#### 2. kube-apiserver
It is a central component of the Kubernetes control plane, responsible for managing communication within the cluster.  
**A primary management component.**

It exposes a RESTful API that allows users and components to perform operations on Kubernetes resources like pods, services, and deployments. Allows users and other components to create, read, update, and delete Kubernetes resources.

**Key functionalities:**
- RESTful Interface
- Resource Management
- Authentication and Authorization
- Admission Control
- Communication with etcd
- Event Notification
- Interaction with Controllers
- Error Handling

#### 3. kube-controller-manager
It is a core component of Kubernetes responsible for managing controllers that regulate the state of the cluster.  
**Each controller is a separate process, but they are all run within a single binary called kube-controller-manager.**

Continuously monitors the cluster state via the API server and makes decisions to bring the cluster closer to the desired state as defined in the Kubernetes specifications.

In a high-availability setup, multiple instances of kube-controller-manager can run, and they use leader election to ensure that only one instance actively manages the controllers at any given time.

**Few Controllers:**
- Replication Controller
- Deployment Controller
- Node Controller
- Namespace Controller
- Ingress Controller
- and many more…

#### 4. kube-scheduler
It is a key component of Kubernetes responsible for deciding which node an unscheduled pod will run on.  
**i.e., the scheduler watches for newly created pods that have no assigned nodes and evaluates them against available nodes.**

The kube-scheduler plays a crucial role in ensuring efficient and optimal placement of pods in a Kubernetes cluster, helping to balance workloads and maintain resource availability.

**Kubernetes supports custom schedulers, allowing users to implement their scheduling logic.**

**Scheduling Process:**
- When a pod is created, it enters the pending state if no nodes are available for scheduling
- The scheduler filters out nodes that do not meet the pod's requirements (e.g., resource limits)
- The remaining nodes are ranked based on various factors (uses a priority function to assign a score to the nodes on a scale from 0 to 10)

## Worker Nodes

**Also known as minions**, are where the actual applications run.  
Each worker node is managed by the control plane and runs the necessary services to execute and manage containerized applications.  
They provide the necessary resources (CPU, memory, storage) for executing these applications.

Worker nodes provide isolation between different applications and workloads, ensuring that one workload does not interfere with another. They also implement security measures to protect the applications running on them.

### Components of the Worker Node

#### 1. kubelet
It is a crucial component of Kubernetes, responsible for managing the lifecycle of containers on individual nodes in a Kubernetes cluster.  
It runs on each node, ensuring that the containers described in PodSpecs are running as expected.

The kubelet constantly checks the health and status of pods and containers running on the node. If a container fails, kubelet can restart it based on its configuration.

It gathers resource utilization data (CPU, memory, etc.) and reports to the control plane, helping in scheduling and scaling decisions.  
**Kubelet communicates with the container runtime (e.g., Docker, containerd) to start, stop, and manage containers.**

#### 2. Container Runtime Engine
It is a crucial component responsible for running and managing the containers.  
**Docker, containerd, CRI-O, gVisor**

**Container Runtime Interface is an API in Kubernetes that allows Kubernetes to interact with different container runtimes.**  
This separation enables Kubernetes to support multiple container runtimes while maintaining a consistent interface.

#### 3. kube-proxy
It is a critical component of Kubernetes that manages network communication within a cluster.  
**A pod network is an internal virtual network that extends across all nodes in the cluster, allowing all pods to connect to it.**  
It is a process that runs on each node in the cluster.

When a Service is created, kube-proxy sets up the necessary routing rules.  
**For example, if a Service with a cluster IP is created, kube-proxy ensures that any traffic sent to that IP is appropriately routed to one of the Pods backing the Service.**



## ADD-Ons
#### 1. Metrics Server
The Metrics Server is a cluster-wide aggregator of resource usage data, running as a pod in the control plane (typically in the kube-system namespace).
​
Kubelets expose resource metrics (CPU, memory) via a metrics endpoint, which Metrics Server scrapes and aggregates for use by Horizontal Pod Autoscaler (HPA), kubectl top, and other components.
​
It enables cluster-wide monitoring and autoscaling decisions without requiring external monitoring stacks like Prometheus.

### 2.  CNI (Container Network Interface) Plugin
CNI plugins implement Kubernetes networking model, required for pod-to-pod communication across nodes.
​
Kubelet invokes CNI plugins to set up pod network interfaces, assign IP addresses, and configure routes when pods are created or deleted.
​
Common plugins: Calico, Flannel, Cilium, Weave Net—enabling the pod network mentioned above.

### 3. CoreDNS
CoreDNS provides cluster DNS resolution as pods deployed via Deployment in kube-system namespace (typically 2 replicas for HA).
​
It resolves Kubernetes DNS names like my-service.default.svc.cluster.local for service discovery within the cluster.
​
Reachable via ClusterIP Service; integrates with kube-proxy for pod access.


***
