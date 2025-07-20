## **EKS Architecture (ASCII Diagram)**

```plaintext
                         +-------------------------------+
                         |      AWS Managed Control      |
                         |         Plane (EKS)           |
                         |  - Kubernetes API Server      |
                         |  - etcd                       |
                         |                               |
                         +-------------------------------+
                                       |
       +-------------------------------|--------------------------------+
       |                               |                                |
+--------------+            +-------------------------------+   +------------------+
|   Worker     |            |   Worker Node (EC2 Instance)  |   |  Fargate Node   |
|   Node (EC2) |            |   - Runs Kubernetes Pods      |   |  - Serverless   |
|   - Manages  |            |   - Uses AWS IAM Roles        |   |  - Lightweight  |
|   Pods       |            |                               |   |                 |
+--------------+            +-------------------------------+   +------------------+

                        Kubernetes Networking (AWS VPC)
      +-------------------------------------------------------------------+
      | - Pods communicate using VPC IPs                                 |
      | - AWS CNI Plugin manages pod-to-pod and pod-to-service traffic   |
      +-------------------------------------------------------------------+

                  Load Balancers (ALB, NLB) for External Traffic
```

---

## **EKS Interview Questions with Answers**

### 1. **EKS Basics**
- **What is Amazon EKS?**
  A managed Kubernetes service that simplifies running Kubernetes by handling the control plane.

- **Key Components of EKS Architecture?**
  - AWS Managed Control Plane: Includes the Kubernetes API server and etcd.
  - Worker Nodes: EC2 or Fargate instances for running workloads.
  - Networking: AWS VPC and CNI for pod communication.

---

### 2. **Cluster Management**
- **How to Create and Manage Clusters?**
  Use `eksctl`, AWS CLI, or AWS Console.

- **What is `eksctl`?**
  A CLI tool to simplify cluster creation and node management.

---

### 3. **Networking**
- **How Does Networking Work in EKS?**
  - Pods are assigned IPs from the VPC CIDR range.
  - AWS CNI manages pod-to-pod networking.

- **Role of AWS CNI Plugin?**
  Ensures Kubernetes pods can use the VPC network for communication.

---

### 4. **IAM and Security**
- **How is IAM Used in EKS?**
  - IAM roles provide AWS permissions to worker nodes.
  - IAM users/roles mapped to Kubernetes RBAC using the `aws-auth` ConfigMap.

---

### 5. **Storage**
- **Storage Options in EKS?**
  - EBS for block storage.
  - EFS for shared file systems.
  - FSx for high-performance workloads.

- **Persistent Volumes (PVs) and Persistent Volume Claims (PVCs):**
  - PVCs are used by pods to request storage dynamically provisioned by StorageClasses.

---

### 6. **Monitoring and Logging**
- **How to Monitor EKS?**
  - Use CloudWatch, Prometheus, or Grafana for metrics.
  - Fluentd for centralized logging.

---

### 7. **Deployments and Workloads**
- **Difference Between Deployments, StatefulSets, and DaemonSets?**
  - Deployments: Manage stateless applications.
  - StatefulSets: Manage stateful apps with unique pod identities.
  - DaemonSets: Ensure a pod runs on every worker node.

- **What are Helm Charts?**
  Pre-configured Kubernetes manifests for deploying applications.

---

### 8. **Scaling and Load Balancing**
- **How Does EKS Support Load Balancing?**
  - Kubernetes Services of type `LoadBalancer` integrate with AWS ALB or NLB.

- **Scaling Options:**
  - HPA for automatically scaling pods.
  - Cluster Autoscaler for adding/removing nodes.

---

### 9. **EKS Add-ons**
- **What are EKS Add-ons?**
  Managed Kubernetes tools like CoreDNS, kube-proxy, and metrics-server.

---

### 10. **Troubleshooting**
- **How to Debug Issues?**
  - Use `kubectl logs` and `kubectl describe` to inspect pods.
  - Check node health with `kubectl get nodes`.

---

### 11. **Cost Optimization**
- **How to Optimize EKS Costs?**
  - Use Spot Instances for nodes.
  - Use Fargate for sporadic workloads.

---

### 12. **Best Practices**
- **Security:**
  - Use IAM roles for service accounts.
  - Enable network policies for pod isolation.

- **High Availability:**
  - Deploy worker nodes across multiple AZs.
  - Use multiple replicas for critical workloads.

---
