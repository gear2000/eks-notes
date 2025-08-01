# EKS Architecture Diagram

```
                            AWS Cloud
    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                 │
    │  ┌─────────────────────────────────────────────────────────┐    │
    │  │                EKS Control Plane                       │    │
    │  │              (Managed by AWS)                          │    │
    │  │                                                        │    │
    │  │  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐  │    │
    │  │  │  API Server  │  │     etcd     │  │ Scheduler   │  │    │
    │  │  └──────────────┘  └──────────────┘  └─────────────┘  │    │
    │  │                                                        │    │
    │  │  ┌──────────────┐  ┌──────────────┐                   │    │
    │  │  │ Controller   │  │   CoreDNS    │                   │    │
    │  │  │   Manager    │  │              │                   │    │
    │  │  └──────────────┘  └──────────────┘                   │    │
    │  └─────────────────────────────────────────────────────────┘    │
    │                               │                                 │
    │                               │ Secure API Calls               │
    │                               │                                 │
    │  ┌─────────────────────────────────────────────────────────┐    │
    │  │                    VPC                                  │    │
    │  │                                                         │    │
    │  │  ┌─────────────────┐    ┌─────────────────┐            │    │
    │  │  │   Subnet AZ-1   │    │   Subnet AZ-2   │            │    │
    │  │  │                 │    │                 │            │    │
    │  │  │ ┌─────────────┐ │    │ ┌─────────────┐ │            │    │
    │  │  │ │Worker Node  │ │    │ │Worker Node  │ │            │    │
    │  │  │ │   (EC2)     │ │    │ │   (EC2)     │ │            │    │
    │  │  │ │             │ │    │ │             │ │            │    │
    │  │  │ │┌───────────┐│ │    │ │┌───────────┐│ │            │    │
    │  │  │ ││   Pod 1   ││ │    │ ││   Pod 3   ││ │            │    │
    │  │  │ │└───────────┘│ │    │ │└───────────┘│ │            │    │
    │  │  │ │┌───────────┐│ │    │ │┌───────────┐│ │            │    │
    │  │  │ ││   Pod 2   ││ │    │ ││   Pod 4   ││ │            │    │
    │  │  │ │└───────────┘│ │    │ │└───────────┘│ │            │    │
    │  │  │ └─────────────┘ │    │ └─────────────┘ │            │    │
    │  │  └─────────────────┘    └─────────────────┘            │    │
    │  │                                                         │    │
    │  │  ┌─────────────────┐                                   │    │
    │  │  │  Fargate Tasks  │                                   │    │
    │  │  │                 │                                   │    │
    │  │  │ ┌─────────────┐ │                                   │    │
    │  │  │ │    Pod A    │ │                                   │    │
    │  │  │ └─────────────┘ │                                   │    │
    │  │  │ ┌─────────────┐ │                                   │    │
    │  │  │ │    Pod B    │ │                                   │    │
    │  │  │ │             │ │                                   │    │
    │  │  │ └─────────────┘ │                                   │    │
    │  │  └─────────────────┘                                   │    │
    │  └─────────────────────────────────────────────────────────┘    │
    │                                                                 │
    │  ┌─────────────────────────────────────────────────────────┐    │
    │  │                AWS Services                             │    │
    │  │                                                         │    │
    │  │ ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐ │    │
    │  │ │     IAM     │ │     EBS     │ │    Load Balancer    │ │    │
    │  │ │   Roles     │ │   Storage   │ │    (ALB/NLB/CLB)    │ │    │
    │  │ └─────────────┘ └─────────────┘ └─────────────────────┘ │    │
    │  │                                                         │    │
    │  │ ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐ │    │
    │  │ │ CloudWatch  │ │     EFS     │ │      Route 53       │ │    │
    │  │ │ Monitoring  │ │  File Sys   │ │        DNS          │ │    │
    │  │ └─────────────┘ └─────────────┘ └─────────────────────┘ │    │
    │  └─────────────────────────────────────────────────────────┘    │
    └─────────────────────────────────────────────────────────────────┘
```

# EKS Networking Flow

```
Internet/Users
      │
      ▼
┌─────────────┐
│Load Balancer│ (ALB/NLB/CLB)
│             │
└─────────────┘
      │
      ▼
┌─────────────────────────────────────────┐
│              VPC Network                │
│                                         │
│  ┌─────────────┐    ┌─────────────┐    │
│  │   Public    │    │   Private   │    │
│  │   Subnet    │    │   Subnet    │    │
│  │             │    │             │    │
│  │    NAT      │    │   Worker    │    │
│  │  Gateway    │────│   Nodes     │    │
│  │             │    │             │    │
│  └─────────────┘    └─────────────┘    │
│                            │            │
│     AWS CNI Plugin         │            │
│    ┌─────────────────┐     │            │
│    │ Pod-to-Pod      │◄────┘            │
│    │ Communication   │                  │
│    │ (VPC IPs)       │                  │
│    └─────────────────┘                  │
└─────────────────────────────────────────┘
            │
            ▼
    ┌─────────────┐
    │EKS Control  │
    │   Plane     │
    │  (AWS API)  │
    └─────────────┘
```

# EKS Security and IAM Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                         IAM Integration                          │
│                                                                  │
│ ┌─────────────┐          ┌─────────────────┐                    │
│ │   IAM User  │          │   IAM Role      │                    │
│ │             │          │  (for Nodes)    │                    │
│ └─────┬───────┘          └─────┬───────────┘                    │
│       │                        │                                │
│       │                        │                                │
│       ▼                        ▼                                │
│ ┌─────────────────────────────────────────────────────────────┐ │
│ │             aws-auth ConfigMap                              │ │
│ │                                                             │ │
│ │  mapRoles:                    mapUsers:                     │ │
│ │  - rolearn: arn:aws:iam::...  - userarn: arn:aws:iam::...   │ │
│ │    username: system:node:     username: admin              │ │
│ │    groups:                    groups:                       │ │
│ │    - system:bootstrappers     - system:masters             │ │
│ │    - system:nodes                                           │ │
│ └─────────────────────────────────────────────────────────────┘ │
│                                │                                │
│                                ▼                                │
│ ┌─────────────────────────────────────────────────────────────┐ │
│ │                Kubernetes RBAC                             │ │
│ │                                                             │ │
│ │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │ │
│ │  │    Role     │  │ ClusterRole │  │   RoleBinding       │ │ │
│ │  │             │  │             │  │  ClusterRoleBinding │ │ │
│ │  └─────────────┘  └─────────────┘  └─────────────────────┘ │ │
│ └─────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────┘
```

# EKS Storage Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     EKS Storage Options                        │
│                                                                 │
│ ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│ │      EBS        │  │      EFS        │  │      FSx        │ │
│ │  Block Storage  │  │  File Storage   │  │ High Performance│ │
│ │                 │  │                 │  │  File System    │ │
│ │ ┌─────────────┐ │  │ ┌─────────────┐ │  │ ┌─────────────┐ │ │
│ │ │   Volume    │ │  │ │ Shared File │ │  │ │   Lustre    │ │ │
│ │ │   (GP2/GP3) │ │  │ │   System    │ │  │ │   or ONTAP  │ │ │
│ │ └─────────────┘ │  │ └─────────────┘ │  │ └─────────────┘ │ │
│ └─────────────────┘  └─────────────────┘  └─────────────────┘ │
│         │                      │                      │       │
│         ▼                      ▼                      ▼       │
│ ┌─────────────────────────────────────────────────────────────┐ │
│ │               Storage Classes                               │ │
│ │                                                             │ │
│ │  gp2-storage     efs-storage     fsx-lustre-storage         │ │
│ │  provisioner:    provisioner:    provisioner:              │ │
│ │  ebs.csi...      efs.csi...      fsx.csi...                │ │
│ └─────────────────────────────────────────────────────────────┘ │
│                                │                               │
│                                ▼                               │
│ ┌─────────────────────────────────────────────────────────────┐ │
│ │                 Pod Storage Usage                           │ │
│ │                                                             │ │
│ │  ┌───────────────┐    ┌─────────────────┐                  │ │
│ │  │      PVC      │────│       PV        │                  │ │
│ │  │   (Request)   │    │    (Volume)     │                  │ │
│ │  └───────────────┘    └─────────────────┘                  │ │
│ │          │                      │                          │ │
│ │          └──────────────────────┼──────────────────────────┘ │ │
│ │                                 │                            │ │
│ │  ┌─────────────────────────────────────────────────────────┐ │ │
│ │  │                    Pod                                  │ │ │
│ │  │                                                         │ │ │
│ │  │  volumeMounts:                                          │ │ │
│ │  │  - name: storage                                        │ │ │
│ │  │    mountPath: /data                                     │ │ │
│ │  └─────────────────────────────────────────────────────────┘ │ │
│ └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

# EKS Scaling Architecture

```
                              Scaling in EKS
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              Horizontal Pod Autoscaler (HPA)               │    │
│  │                                                             │    │
│  │  Metrics Server ──► CPU/Memory Usage ──► Scale Pods        │    │
│  │       │                    │                   │           │    │
│  │       ▼                    ▼                   ▼           │    │
│  │  ┌─────────┐          ┌─────────┐          ┌─────────┐     │    │
│  │  │  Pod 1  │          │  Pod 2  │    ┌────►│  Pod 3  │     │    │
│  │  └─────────┘          └─────────┘    │     └─────────┘     │    │
│  │                                      │                     │    │
│  │                                      │ (Auto-created)      │    │
│  └──────────────────────────────────────┼─────────────────────┘    │
│                                         │                          │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              Cluster Autoscaler                             │    │
│  │                                                             │    │
│  │  ┌─────────────┐          ┌─────────────┐                  │    │
│  │  │   Node 1    │          │   Node 2    │                  │    │
│  │  │   (EC2)     │          │   (EC2)     │                  │    │
│  │  │             │          │             │                  │    │
│  │  │ ┌─────────┐ │          │ ┌─────────┐ │                  │    │
│  │  │ │ Pod 1   │ │          │ │ Pod 2   │ │                  │    │
│  │  │ │ Pod 3   │ │          │ │         │ │                  │    │
│  │  │ └─────────┘ │          │ └─────────┘ │                  │    │
│  │  └─────────────┘          └─────────────┘                  │    │
│  │                                │                           │    │
│  │     When pods can't be         │                           │    │
│  │     scheduled due to           ▼                           │    │
│  │     resource constraints   ┌─────────────┐                 │    │
│  │                            │   Node 3    │ ◄── New Node   │    │
│  │                            │   (EC2)     │     Added      │    │
│  │                            │             │                 │    │
│  │                            │ ┌─────────┐ │                 │    │
│  │                            │ │ Pod 3   │ │                 │    │
│  │                            │ └─────────┘ │                 │    │
│  │                            └─────────────┘                 │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                   Fargate Scaling                          │    │
│  │                                                             │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │    │
│  │  │ Fargate     │  │ Fargate     │  │ Fargate     │         │    │
│  │  │ Task 1      │  │ Task 2      │  │ Task 3      │         │    │
│  │  │             │  │             │  │             │         │    │
│  │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │         │    │
│  │  │ │  Pod A  │ │  │ │  Pod B  │ │  │ │  Pod C  │ │         │    │
│  │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │         │    │
│  │  └─────────────┘  └─────────────┘  └─────────────┘         │    │
│  │                                                             │    │
│  │  (Serverless - No node management required)                │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

# EKS Add-ons and Monitoring

```
┌─────────────────────────────────────────────────────────────────────┐
│                        EKS Add-ons                                 │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                  Core Add-ons                              │    │
│  │                                                             │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │    │
│  │  │   CoreDNS   │  │ Kube-proxy  │  │    AWS CNI Plugin   │ │    │
│  │  │             │  │             │  │                     │ │    │
│  │  │ DNS Resol-  │  │ Network     │  │ Pod Networking      │ │    │
│  │  │ ution for   │  │ Rules &     │  │ & IP Assignment     │ │    │
│  │  │ Services    │  │ Load Bal.   │  │                     │ │    │
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘ │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                Monitoring Stack                             │    │
│  │                                                             │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │    │
│  │  │ CloudWatch  │  │ Prometheus  │  │     Grafana         │ │    │
│  │  │             │  │             │  │                     │ │    │
│  │  │ AWS Native  │  │ Metrics     │  │ Visualization       │ │    │
│  │  │ Monitoring  │  │ Collection  │  │ & Dashboards        │ │    │
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘ │    │
│  │         │                │                    │             │    │
│  │         ▼                ▼                    ▼             │    │
│  │  ┌─────────────────────────────────────────────────────────┐ │    │
│  │  │               Metrics Flow                             │ │    │
│  │  │                                                         │ │    │
│  │  │  Pod Metrics ──► Node Metrics ──► Cluster Metrics      │ │    │
│  │  │       │               │                │               │ │    │
│  │  │       ▼               ▼                ▼               │ │    │
│  │  │   CPU/Memory    Disk/Network    Control Plane Health   │ │    │
│  │  └─────────────────────────────────────────────────────────┘ │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Logging                                  │    │
│  │                                                             │    │
│  │  ┌─────────────┐                ┌─────────────────────────┐ │    │
│  │  │  Fluentd/   │     Logs       │    CloudWatch Logs      │ │    │
│  │  │  Fluent Bit │ ──────────────►│                         │ │    │
│  │  │             │                │   Log Groups:           │ │    │
│  │  │ (DaemonSet) │                │   - /aws/eks/cluster    │ │    │
│  │  └─────────────┘                │   - Application logs    │ │    │
│  │         ▲                       └─────────────────────────┘ │    │
│  │         │                                                   │    │
│  │  ┌─────────────────────────────────────────────────────────┐ │    │
│  │  │              Pod Log Sources                            │ │    │
│  │  │                                                         │ │    │
│  │  │  stdout/stderr ──► Container Logs ──► Node Log Files    │ │    │
│  │  └─────────────────────────────────────────────────────────┘ │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```
