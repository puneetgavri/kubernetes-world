# ** Persistent Volumes (PV) and Persistent Volume Claims (PVC) in Kubernetes**

## **1. Why Do We Need PVs and PVCs?**
By default, containers in Kubernetes are **ephemeral**, meaning:
- If a Pod is deleted, all data stored in it is lost.
- Even if a Pod restarts, its data does not persist.
- Containers within a Pod can share storage, but data does not survive when the Pod is removed.

For applications that require **persistent storage**, like databases or logs, Kubernetes provides **Persistent Volumes (PVs) and Persistent Volume Claims (PVCs).**

---

## **2. What is a Persistent Volume (PV)?**
A **Persistent Volume (PV)** is a **pre-provisioned** storage resource in Kubernetes.
- It **exists independently** of Pods.
- It can be backed by cloud storage (AWS EBS, GCP Persistent Disk, Azure Disk), network storage (NFS, CephFS), or on-prem storage.
- It has **specific properties**, such as storage capacity, access modes, and reclaim policies.

### **Example of a Persistent Volume (PV)**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: "/mnt/data"
```

### **Explanation:**
- **`capacity: storage: 5Gi`** → The PV provides **5GiB** of storage.
- **`accessModes: ReadWriteOnce`** → Only one node can mount this volume **at a time**.
- **`persistentVolumeReclaimPolicy: Retain`** → If the PVC using this PV is deleted, the PV **retains its data**.
- **`storageClassName: manual`** → The PV is manually created (not dynamically provisioned).
- **`hostPath: /mnt/data`** → The PV is stored on the host node's filesystem.

---

## **3. What is a Persistent Volume Claim (PVC)?**
A **Persistent Volume Claim (PVC)** is a **request** for storage by a Pod.
- A Pod does not claim storage directly; instead, it **requests storage using a PVC**.
- The PVC looks for a **matching PV** and binds to it.
- If a StorageClass is used, Kubernetes **dynamically creates a PV** for the PVC.

### **Example of a Persistent Volume Claim (PVC)**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  storageClassName: manual
```

---

## **4. Using a PVC in a Pod**
Once a PVC is created, a **Pod can use it as a volume**.

### **Example: Pod Using a PVC**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - mountPath: "/data"
          name: storage
  volumes:
    - name: storage
      persistentVolumeClaim:
        claimName: my-pvc
```

---

## **5. Storage Classes and Dynamic Provisioning**
A **StorageClass** in Kubernetes allows dynamic provisioning of Persistent Volumes, eliminating the need for manual PV creation.

### **How StorageClass Works:**
1. The **cluster administrator** defines a StorageClass.
2. When a **PVC requests storage**, Kubernetes **automatically provisions** a PV.
3. The provisioned PV is bound to the requesting PVC.

### **How StorageClass Differs from PVs:**
| **Feature** | **Persistent Volume (PV)** | **Storage Class** |
|------------|----------------------|----------------------|
| **Definition** | A pre-provisioned storage unit | A template for dynamically provisioning PVs |
| **Management** | Manually created and managed | Dynamically provisions PVs on demand |
| **Binding** | Bound to a PVC upon request | Used by PVCs to create PVs dynamically |
| **Flexibility** | Fixed configuration | More flexible, can create PVs with different settings |
| **Use Case** | Used when static storage allocation is needed | Preferred for cloud environments where dynamic storage is required |

### **Example: Defining a StorageClass**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
```

### **Example: PVC Using Dynamic Provisioning**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: fast-storage
```
- **Kubernetes will automatically create a PV on AWS EBS for this PVC.**

---

## **6. Persistent Volume Reclaim Policy**
Defines **what happens to a PV when its PVC is deleted**.

| **Policy** | **Description** |
|-----------|---------------|
| `Retain` | PV keeps its data, but needs **manual deletion** before reuse. |
| `Delete` | PV and its data are **automatically deleted** (useful for cloud storage). |
| `Recycle` | PV is **wiped and reset** (deprecated). |

### **Example: PV with `Delete` Policy**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: dynamic-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: fast-storage
  gcePersistentDisk:
    pdName: my-disk
    fsType: ext4
```
