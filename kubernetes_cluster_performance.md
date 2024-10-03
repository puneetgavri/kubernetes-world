## ### Kubernetes Optimization Tips for Peak Performance 🚀 (With code example)
**Optimizing your Kubernetes cluster can significantly boost performance and efficiency. Here are some practical strategies to help you streamline operations and reduce costs.**

 ## 1. Optimize Resource Requests and Limits
**Set appropriate resource requests and limits for CPU and memory prevent over-provisioning and underutilization plus monitor workloads and use tools like kubectl top, Prometheus, or metrics server to set resource requests and limits accurately based on actual usage.**
````yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: optimized-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: optimized-app
  template:
    metadata:
      labels:
        app: optimized-app
    spec:
      containers:
      - name: app-container
        image: your-app-image
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1"
````

### 2. Use Horizontal Pod Autoscaling (HPA)
**HPA can help scale pods automatically based on CPU, memory, or custom metrics to ensure applications can handle varying workloads by defining a target metric for autoscaling and configuring the HPA in the deployment YAML.**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: optimized-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
```

### 3. Leverage Cluster Autoscaler
**Automatically we can adjust the size of the cluster by adding or removing nodes when required, which can reduce costs and ensure there’s always enough capacity by configuring the cluster autoscaler to work with your cloud provider (e.g., AWS, GCP) to add or remove nodes based on resource demand.**
```yaml
# Configure Cluster Autoscaler to scale node groups in an AWS EKS cluster
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  labels:
    app: cluster-autoscaler
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
    spec:
      serviceAccountName: cluster-autoscaler
      containers:
      - image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.xxx #enter latest version
        name: cluster-autoscaler
        command:
        - ./cluster-autoscaler
        - --cloud-provider=aws
        - --skip-nodes-with-local-storage=false
        - --nodes=1:10:<node-group-name>
```
Or use a better alternative like Karpenter
Karpenter is an open-source Kubernetes cluster auto scaler that dynamically provisions just-in-time compute resources based on the needs of your workloads. It simplifies node provisioning and reduces cluster costs by launching nodes tailored to your workload requirements.
```yaml
# Install Karpenter Helm chart
helm repo add karpenter https://charts.karpenter.sh/
helm repo update

helm install karpenter karpenter/karpenter \
    --namespace karpenter --create-namespace \
    --set controller.clusterName=<your-cluster-name> \
    --set controller.clusterEndpoint=$(aws eks describe-cluster --name <your-cluster-name> --query "cluster.endpoint" --output text) \
    --set serviceAccount.create=true \
    --set serviceAccount.annotations."eks\.amazonaws\.com/role-arn"=<your-karpenter-role-arn> 
```
```yaml
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: amd64
spec:
  requirements:
    - key: karpenter.sh/capacity-type
      operator: In
      values: ["on-demand"]
    - key: "topology.kubernetes.io/zone" 
      operator: In
      values: ["eu-west-1a", "eu-west-1b", "eu-west-1c"]
    - key: "kubernetes.io/arch" 
      operator: In
      values: ["amd64"]
  limits:
    resources:
      cpu: 100
  provider:
    instanceProfile: KarpenterNodeInstanceProfile-karpenter-cluster
    securityGroupSelector:
      kubernetes.io/cluster/karpenter-cluster: '*'
  ttlSecondsAfterEmpty: 30
```

### 4. Implement Pod Disruption Budgets (PDB)
**Ensure critical applications maintain availability during voluntary disruptions like node upgrades or autoscaling events by defining Pod Disruption Budgets to limit the number of pods that can be taken down simultaneously during planned maintenance.**

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: pdb-app
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: optimized-app

```
### 5. Enable Vertical Pod Autoscaler (VPA)
PA automatically adjusts resource requests and limits based on actual usage over time, improving resource utilization by enabling VPA for workloads that require frequent changes in resource consumption to automatically tune their resource allocations. 
(This is especially useful for DB and cache-related workloads)

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: vpa-app
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: optimized-app
  updatePolicy:
    updateMode: "Auto"
```
###  6. Optimize Networking with CNI Plugins
**Efficient network communication reduces latency and improves performance, especially for large clusters by choosing a suitable CNI plugin (e.g., Calico, Cilium, Flannel) based on network requirements and configuring network policies to secure and streamline communication between pods. (Below is an example of Calico )**

#### install Calico CNI plugin:
```yaml
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-specific-app
spec:
  podSelector:
    matchLabels:
      app: optimized-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: allowed-app
```

### 7. Use Efficient Storage Classes and Persistent Volumes
Properly configured storage classes improve I/O performance for applications that rely on databases or need fast read/write speeds by choosing appropriate storage classes based on the workload (SSD, HDD, etc.), and ensuring that persistent volumes are efficiently utilized by deleting unused volumes.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iopsPerGB: "50"
```

---

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-storage
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: fast-ssd
  resources:
    requests:
      storage: 10Gi
```

### 8.  Leverage Node Local DNS Cache
**It improves DNS resolution times and reduces the load on the cluster DNS service by caching DNS lookups locally using the NodeLocal DNS Cache feature in Kubernetes to improve performance, especially for applications that make many DNS queries.**

##### Enable NodeLocal DNS cache by running this command
kubectl apply -f https://k8s.io/examples/admin/dns/nodelocaldns.yaml


### 9. Optimize Pod Affinity and Anti-affinity Rules
It ensures efficient workload distribution across nodes, preventing resource contention and improving resilience by using pod affinity and anti-affinity rules to co-locate or distribute pods based on performance needs and failover scenarios.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: affinity-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: affinity-app
  template:
    metadata:
      labels:
        app: affinity-app
    spec:
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - another-app
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: affinity-container
        image: your-app-image
```

### 10. Use Readiness and Liveness Probes
**It ensures that only healthy pods serve traffic, preventing bad user experiences and reducing downtime by configuring readiness and liveness probes in your deployments to monitor the health of your application and automatically restart unhealthy pods.**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: probe-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: probe-app
  template:
    metadata:
      labels:
        app: probe-app
    spec:
      containers:
      - name: probe-container
        image: your-app-image
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /readiness
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
```

By applying these optimization techniques, you can enhance Kubernetes cluster performance, improve resource utilization, and ensure smoother, more scalable operations.
