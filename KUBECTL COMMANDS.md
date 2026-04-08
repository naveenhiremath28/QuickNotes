## 1. Cluster & Context

```bash
# View cluster info
kubectl cluster-info

# Check kubectl version
kubectl version

# View current context
kubectl config current-context

# List all contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <context-name>

# View kubeconfig
kubectl config view

# Set default namespace for current context
kubectl config set-context --current --namespace=production
```

---

## 2. Namespace

```bash
# List all namespaces
kubectl get namespaces
kubectl get ns

# Create a namespace
kubectl create namespace <name>
kubectl create ns production

# Delete a namespace
kubectl delete namespace <name>

# Describe a namespace
kubectl describe namespace <name>
```

---

## 3. Pods

```bash
# List pods in current namespace
kubectl get pods
kubectl get po

# List pods in a specific namespace
kubectl get pods -n production

# List pods in all namespaces
kubectl get pods -A

# List pods with more details (node, IP, etc.)
kubectl get pods -o wide

# Watch pods in real time
kubectl get pods -w

# Describe a pod (events, config, status)
kubectl describe pod <pod-name>
kubectl describe pod <pod-name> -n production

# Get pod logs
kubectl logs <pod-name>

# Get logs from a specific container in a pod
kubectl logs <pod-name> -c <container-name>

# Stream live logs
kubectl logs -f <pod-name>

# Get last N lines of logs
kubectl logs <pod-name> --tail=100

# Get logs from previous (crashed) container
kubectl logs <pod-name> --previous

# Execute a command inside a pod
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -- /bin/sh

# Execute command in a specific container
kubectl exec -it <pod-name> -c <container-name> -- /bin/bash

# Delete a pod
kubectl delete pod <pod-name>

# Delete a pod forcefully
kubectl delete pod <pod-name> --force --grace-period=0

# Run a temporary pod for debugging
kubectl run debug --image=busybox --rm -it --restart=Never -- sh
```

---

## 4. Deployments

```bash
# List deployments
kubectl get deployments
kubectl get deploy

# Describe a deployment
kubectl describe deployment <name>

# Create a deployment
kubectl create deployment <name> --image=nginx

# Scale a deployment
kubectl scale deployment <name> --replicas=3

# Update image of a deployment
kubectl set image deployment/<name> <container>=<new-image>:<tag>
kubectl set image deployment/myapp app=nginx:1.25

# Check rollout status
kubectl rollout status deployment/<name>

# View rollout history
kubectl rollout history deployment/<name>

# Undo last rollout (roll back)
kubectl rollout undo deployment/<name>

# Roll back to a specific revision
kubectl rollout undo deployment/<name> --to-revision=2

# Pause a rollout
kubectl rollout pause deployment/<name>

# Resume a rollout
kubectl rollout resume deployment/<name>

# Restart all pods in a deployment (rolling restart)
kubectl rollout restart deployment/<name>

# Delete a deployment
kubectl delete deployment <name>
```

---

## 5. Services

```bash
# List services
kubectl get services
kubectl get svc

# Describe a service
kubectl describe service <name>

# Expose a deployment as a service
kubectl expose deployment <name> --port=80 --type=ClusterIP
kubectl expose deployment <name> --port=80 --type=NodePort
kubectl expose deployment <name> --port=80 --type=LoadBalancer

# Delete a service
kubectl delete service <name>

# Port-forward a service to localhost
kubectl port-forward service/<name> 8080:80

# Port-forward a pod to localhost
kubectl port-forward pod/<pod-name> 8080:80
```

---

## 6. ConfigMaps & Secrets

```bash
# List ConfigMaps
kubectl get configmaps
kubectl get cm

# Create a ConfigMap from literal values
kubectl create configmap <name> --from-literal=KEY=VALUE

# Create a ConfigMap from a file
kubectl create configmap <name> --from-file=config.properties

# Describe a ConfigMap
kubectl describe configmap <name>

# Delete a ConfigMap
kubectl delete configmap <name>

# List Secrets
kubectl get secrets

# Create a generic Secret
kubectl create secret generic <name> --from-literal=password=mysecret

# Create a TLS Secret
kubectl create secret tls <name> --cert=tls.crt --key=tls.key

# Decode a Secret value
kubectl get secret <name> -o jsonpath='{.data.password}' | base64 --decode

# Describe a Secret (values are hidden)
kubectl describe secret <name>

# Delete a Secret
kubectl delete secret <name>
```

---

## 7. Nodes

```bash
# List all nodes
kubectl get nodes

# List nodes with more detail
kubectl get nodes -o wide

# Describe a node
kubectl describe node <node-name>

# Check node resource usage
kubectl top node

# Cordon a node (prevent new pods from scheduling)
kubectl cordon <node-name>

# Uncordon a node
kubectl uncordon <node-name>

# Drain a node (evict all pods for maintenance)
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# Taint a node
kubectl taint nodes <node-name> key=value:NoSchedule

# Remove a taint
kubectl taint nodes <node-name> key=value:NoSchedule-
```

---

## 8. Apply / Delete Resources

```bash
# Apply a manifest file
kubectl apply -f deployment.yaml

# Apply all manifests in a directory
kubectl apply -f ./manifests/

# Apply from a URL
kubectl apply -f https://example.com/manifest.yaml

# Delete resources from a file
kubectl delete -f deployment.yaml

# Delete all resources of a type in a namespace
kubectl delete pods --all -n production
kubectl delete deployments --all -n production

# Force delete
kubectl delete -f deployment.yaml --force --grace-period=0
```

---

## 9. Resource Inspection

```bash
# Get resource in YAML format
kubectl get pod <name> -o yaml
kubectl get deployment <name> -o yaml

# Get resource in JSON format
kubectl get pod <name> -o json

# Use JSONPath to extract a specific field
kubectl get pod <name> -o jsonpath='{.status.podIP}'
kubectl get nodes -o jsonpath='{.items[*].metadata.name}'

# Get all resources in a namespace
kubectl get all -n production

# Get all resources across all namespaces
kubectl get all -A

# List all API resources
kubectl api-resources

# Explain a resource field
kubectl explain pod
kubectl explain pod.spec.containers
```

---

## 10. Resource Usage (Metrics)

```bash
# Check pod CPU/memory usage (requires metrics-server)
kubectl top pods
kubectl top pods -n production
kubectl top pods -A

# Check node CPU/memory usage
kubectl top nodes
```

---

## 11. Labels & Annotations

```bash
# List pods with labels shown
kubectl get pods --show-labels

# Filter pods by label
kubectl get pods -l app=nginx
kubectl get pods -l env=production,app=nginx

# Add a label to a pod
kubectl label pod <pod-name> env=production

# Remove a label from a pod
kubectl label pod <pod-name> env-

# Add an annotation
kubectl annotate pod <pod-name> description="my pod"
```

---

## 12. Debugging & Troubleshooting

```bash
# Get events in a namespace (sorted by time)
kubectl get events -n production --sort-by='.lastTimestamp'

# Describe any resource for detailed events
kubectl describe pod <pod-name>
kubectl describe node <node-name>
kubectl describe service <name>

# Check pod resource requests and limits
kubectl describe pod <pod-name> | grep -A 5 "Limits"

# Copy files to/from a pod
kubectl cp <pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <pod-name>:/path/to/file

# Create a temporary debug container (K8s 1.23+)
kubectl debug -it <pod-name> --image=busybox --target=<container>

# Check if a pod can reach a service
kubectl exec -it <pod-name> -- curl http://<service-name>:<port>
```

---