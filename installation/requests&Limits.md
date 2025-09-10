# Requests and Limits in Kubernetes

In **Kubernetes**, **requests** and **limits** are resource management settings defined in a container's specification. They help ensure your applications run reliably and efficiently on the cluster by controlling CPU and memory (RAM) usage.

---

## 🔹 What Are **Requests** and **Limits**?

| Term        | Description                                                              |
| ----------- | ------------------------------------------------------------------------ |
| **Request** | The **minimum** amount of CPU or memory the container is guaranteed.     |
| **Limit**   | The **maximum** amount of CPU or memory the container is allowed to use. |

---

## 💡 Why Use Them?

* **Requests** are used by the **scheduler** to decide which node can run a pod.
* **Limits** prevent a container from using too many resources and affecting other containers.

---

## 🧠 Memory and CPU Explained

| Resource | Unit                    | Notes                                      |
| -------- | ----------------------- | ------------------------------------------ |
| CPU      | millicores (m)          | `500m` = 0.5 CPU core. `1000m` = 1 core.   |
| Memory   | bytes (e.g. `Mi`, `Gi`) | `128Mi` = 128 mebibytes. `1Gi` = 1024 MiB. |

---

## 🧾 Example Pod Spec

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: demo-container
    image: nginx
    resources:
      requests:
        memory: "128Mi"
        cpu: "250m"
      limits:
        memory: "256Mi"
        cpu: "500m"
```

---

## 🔍 What This Means

* Kubernetes will schedule this pod on a node with at least:

  * **128Mi** memory
  * **250m** CPU
* The container **cannot use more than**:

  * **256Mi** memory (will be OOMKilled if it exceeds this)
  * **500m** CPU (throttled if it tries to go beyond this)

---

## 🚨 What Happens If...

| Scenario                                                 | Result                                          |
| -------------------------------------------------------- | ----------------------------------------------- |
| Container exceeds **memory limit**                       | It is **killed** (OOMKilled).                   |
| Container exceeds **CPU limit**                          | It is **throttled** (slowed down).              |
| Container uses more than **request** but under **limit** | Allowed to use, but not guaranteed.             |
| Node runs out of resources                               | Containers with **lower QoS** might be evicted. |

---

## 📊 QoS (Quality of Service) Classes

Depending on how you define `requests` and `limits`, your pod falls into one of these QoS classes:

| QoS Class      | Definition                                              |
| -------------- | ------------------------------------------------------- |
| **Guaranteed** | Requests == Limits for all containers (CPU and Memory). |
| **Burstable**  | Requests < Limits or not all containers specify both.   |
| **BestEffort** | No requests or limits specified.                        |

