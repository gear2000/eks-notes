# Debugging Commands

## Brief (Original Commands)

### Cluster and Node Information
- `kubectl cluster-info`
- `kubectl get nodes`
- `kubectl get node docker-desktop -o yaml`
- `kubectl get pods -n kube-system -o wide`

### Kubelet Service
- `systemctl status kubelet`
- `journalctl -u kubelet.service`
- `systemctl restart kubelet`

### Pods and Events
- `kubectl get pods --all-namespaces`
- `kubectl get events -n app --sort-by=.metadata.creationTimestamp`
- `kubectl get pod old-busybox -n app`
- `kubectl logs old-busybox -n app`
- `kubectl get pod old-busybox -n app -o yaml > my-old-pod.yaml`
- `kubectl edit deploy <deployment-name>`

### Scaling and Exposing Resources
- `kubectl scale rs nginx --replicas=4`
- `kubectl expose deployment nginx --port 80`
- `kubectl expose pod nginx --port 80`
- `kubectl get all -A`
- `kubectl explain replicaset`

### Notes on Services
- Service types: NodePort, ClusterIP, LoadBalancer
- Fully qualified domain name (FQDN) for services:
  `<service_name>.<namespace>.svc.cluster.local`
  Example: `nginx.dev.svc.cluster.local`

### Running and Managing Pods
- `kubectl run nginx --image nginx`
- `kubectl run nginx --image nginx --dry-run=client -o yaml`  # generates base YAML file for modification
- `kubectl describe pod nginx`
- `kubectl edit pod redis`
- `kubectl delete pod nginx`

### Creating Deployments
- `kubectl create deployment --help`
- `kubectl create deployment test --image nginx --replicas=3`

### Example Commands
- `kubectl run --image=nginx nginx`
- `kubectl create deployment --image=nginx nginx`
- `kubectl scale deployment nginx --replicas=5`
- `kubectl set image deployment nginx nginx=nginx:1.18`
- `kubectl set image pod nginx nginx=nginx:1.18`
- `kubectl create -f nginx.yaml`
- `kubectl replace --force -f nginx.yaml`
- `kubectl delete -f nginx.yaml`

### Tips for Creating Services
- Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379:
  ```bash
  kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
  ```

## Extensive (Additional EKS Debugging Commands)

### Cluster and Node Information
- `kubectl cluster-info dump` - Dump detailed cluster state information
- `kubectl get nodes -o wide` - List all nodes with IPs and versions
- `kubectl describe node <node-name>` - Show detailed node information including resource usage
- `kubectl top nodes` - Display node resource usage (CPU/memory)

### AWS EKS Specific
- `aws eks describe-cluster --name <cluster-name>` - Get cluster configuration
- `aws eks list-nodegroups --cluster-name <cluster-name>` - List node groups
- `aws eks describe-nodegroup --cluster-name <cluster-name> --nodegroup-name <nodegroup-name>` - Get node group details
- `aws ec2 describe-instances --filters "Name=tag:eks:cluster-name,Values=<cluster-name>"` - List EC2 instances in cluster

### Node Troubleshooting
- `ssh ec2-user@<node-ip>` - SSH into node (if using Amazon Linux 2)
- `sudo systemctl status kubelet` - Check kubelet service status
- `sudo journalctl -u kubelet.service --no-pager` - View kubelet logs
- `sudo systemctl restart kubelet` - Restart kubelet service
- `kubectl drain <node-name> --ignore-daemonsets` - Safely evict pods before maintenance
- `kubectl cordon <node-name>` - Mark node as unschedulable

### Pod Troubleshooting
- `kubectl describe pod <pod-name> -n <namespace>` - Show detailed pod information
- `kubectl logs <pod-name> -n <namespace> -c <container-name>` - View specific container logs
- `kubectl logs <pod-name> -n <namespace> --previous` - View previous container logs (if crashed)
- `kubectl exec -it <pod-name> -n <namespace> -- /bin/bash` - Get shell access to pod
- `kubectl delete pod <pod-name> -n <namespace> --grace-period=0 --force` - Force delete hanging pod

### Network Debugging
- `kubectl get svc -n <namespace>` - List services
- `kubectl get ingress -n <namespace>` - List ingresses
- `kubectl get endpoints <service-name> -n <namespace>` - Check service endpoints
- `kubectl run curl --image=curlimages/curl -it --rm -- curl <service-name>.<namespace>` - Test service connectivity
- `kubectl port-forward <pod-name> -n <namespace> 8080:80` - Forward local port to pod
- `kubectl describe svc <service-name> -n <namespace>` - View service details including endpoints

### Configuration and Resource Management
- `kubectl get configmaps -n <namespace>` - List ConfigMaps
- `kubectl get secrets -n <namespace>` - List Secrets
- `kubectl rollout history deploy/<deployment-name> -n <namespace>` - View deployment history
- `kubectl rollout undo deploy/<deployment-name> -n <namespace>` - Rollback deployment
- `kubectl scale deployment <deployment-name> --replicas=<number> -n <namespace>` - Scale deployment

### EKS IAM and Authentication
- `aws eks update-kubeconfig --name <cluster-name>` - Update kubeconfig for cluster access
- `kubectl auth can-i <verb> <resource> -n <namespace>` - Check permissions
- `aws eks list-fargate-profiles --cluster-name <cluster-name>` - List Fargate profiles (if using)
- `kubectl get configmap aws-auth -n kube-system -o yaml` - Check AWS IAM mapping to Kubernetes RBAC

### Logging and Metrics
- `kubectl logs -f deployment/<deployment-name> -n <namespace>` - Stream logs from deployment
- `kubectl top pods -n <namespace>` - Show pod resource usage
- `aws logs describe-log-groups --log-group-name-prefix /aws/eks/<cluster-name>` - Check CloudWatch log groups
- `aws logs get-log-events --log-group-name /aws/eks/<cluster-name>/cluster --log-stream-name <log-stream>` - Get CloudWatch logs

### Resource Health and Constraints
- `kubectl get hpa -n <namespace>` - List Horizontal Pod Autoscalers
- `kubectl get pdb -n <namespace>` - List Pod Disruption Budgets
- `kubectl get limitranges -n <namespace>` - List LimitRanges
- `kubectl get resourcequotas -n <namespace>` - List ResourceQuotas