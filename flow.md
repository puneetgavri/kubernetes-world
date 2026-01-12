# Kubernetes Pod Creation Flow: `kubectl create/run pod`

When a user runs `kubectl run nginx --image=nginx` or `kubectl create -f pod.yaml`, here's the **complete step-by-step flow** through the Kubernetes architecture:

## 1. kubectl Command Execution
```
kubectl run nginx --image=nginx
```
- **kubectl** converts the imperative command into a Pod JSON/YAML specification
- **Authenticates** with kube-apiserver using kubeconfig (~/.kube/config)
- Sends **HTTP POST** request to kube-apiserver API endpoint: `/api/v1/namespaces/default/pods`

## 2. kube-apiserver Processing
```
User → kubectl → kube-apiserver (REST API)
```
- Validates **authentication** (tokens, certs, webhooks)
- Validates **authorization** (RBAC - do you have "create pods" permission?)
- Runs **admission controllers** (ValidatingAdmissionWebhook, MutatingAdmissionWebhook)
- Stores Pod spec in **etcd** (desired state: 1 nginx pod)
- Pod appears in etcd with **Pending** status
- Generates **Event** ("Pod created") [stored in etcd]

## 3. kube-controller-manager Watches & Reacts
```
etcd (new Pod object) → kube-controller-manager
```
- **Pod Controller** detects new Pod in Pending state (no node assigned)
- No action needed (unlike Deployments which create ReplicaSets)

## 4. kube-scheduler Assigns Node
```
Pending Pod → kube-scheduler → Filter → Score → Bind
```
- Watches API server for **unscheduled pods** (nodeName: nil)
- **Filter phase**: Checks node requirements (CPU/memory requests, taints/tolerations, nodeSelector)
- **Score phase**: Ranks eligible nodes (0-10) based on:
  - Resource utilization
  - Node affinity/anti-affinity
  - Pod affinity/anti-affinity
- **Binds** Pod to node by updating etcd: `spec.nodeName=worker-2`

## 5. kubelet on Worker Node Receives Instructions
```
etcd (Pod assigned to this node) → kubelet (on worker-2)
```
- kubelet **watches** API server for Pods assigned to its node
- Downloads Pod spec from kube-apiserver
- **Mounts** volumes (if any: ConfigMaps, Secrets, PVCs)
- Calls **Container Runtime Interface (CRI)**

## 6. Container Runtime + CNI Setup
```
kubelet → CRI → containerd/docker + CNI plugin
```
- **Container Runtime** (containerd/docker):
  - Pulls image `nginx` from registry (if not cached)
  - Creates container with Pod spec (env vars, commands, ports)
- **CNI Plugin** (Calico/Flannel/Cilium):
  - Assigns Pod IP from cluster CIDR
  - Creates veth pair + network namespace
  - Sets up routing rules for pod-to-pod communication
  - Updates `/etc/cni/net.d/` config

## 7. Pod Becomes Running
```
Container starts → kubelet → CRI → reports status → kube-apiserver → etcd
```
- kubelet executes container health checks (readiness/liveness probes)
- Container runtime reports **container ID, IP, status** back to kubelet
- kubelet updates Pod **status** in etcd via API server:
  ```
  status.phase: Running
  status.podIP: 10.244.1.5
  status.containerStatuses.ready: true
  ```

## 8. kube-proxy + CoreDNS Integration
```
Pod Running → kube-proxy → Service routing + CoreDNS → DNS resolution
```
- **kube-proxy**: Watches Services/Endpoints, updates iptables for Service ClusterIP → PodIP routing
- **CoreDNS**: Resolves `nginx.default.svc.cluster.local` → Service ClusterIP

## 9. User Verification
```
kubectl get pods → kube-apiserver → etcd → Pod status: Running
```

## Visual Flow Diagram
```
kubectl → apiserver → etcd
                    ↓
           controller-manager (watches)
                    ↓
              scheduler (assigns node)
                    ↓
            kubelet (on worker node)
                    ↓
      CRI + CNI → Container Running
                    ↓
            status → apiserver → etcd
```

## Key etcd State Changes
```
1. Desired: {spec: nginx, status: Pending}
2. Assigned: {spec: {nodeName: worker-2}, status: Pending}
3. Running: {status: {phase: Running, podIP: 10.244.1.5}}
```

**Total time**: 2-10 seconds depending on image pull, CNI setup, and cluster load
