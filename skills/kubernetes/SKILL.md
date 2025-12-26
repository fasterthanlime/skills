---
name: kubernetes
description: Use kubectl for Kubernetes observability, debugging, and read-only operations. This skill should be invoked for any Kubernetes-related tasks including viewing logs, describing resources, getting pod status, port-forwarding, and debugging. CRITICAL SAFETY RULE - NEVER delete any Kubernetes resources under any circumstance. All destructive operations are strictly forbidden.
---

# Kubernetes Operations

## Overview

Use `kubectl` for Kubernetes cluster observability, debugging, and read-only operations. This skill provides guidance for safe Kubernetes operations with a focus on viewing, monitoring, and debugging cluster resources.

## ⚠️ CRITICAL SAFETY RULE

**NEVER DELETE KUBERNETES RESOURCES UNDER ANY CIRCUMSTANCE**

The following commands are **ABSOLUTELY FORBIDDEN**:

```bash
# ❌ NEVER RUN THESE
kubectl delete <any-resource>
kubectl delete all --all
kubectl delete pod <name>
kubectl delete deployment <name>
kubectl delete namespace <name>
kubectl delete -f <file>

# ❌ NEVER USE DESTRUCTIVE OPERATIONS
kubectl apply --prune
kubectl replace --force
kubectl drain (without explicit user approval)
kubectl cordon (without explicit user approval)
```

**If a task requires deleting resources, STOP and inform the user that manual intervention is required.**

## Safe Read-Only Operations

All commands in this section are safe and encouraged for observability and debugging.

### Viewing Resources

#### Get Resources

List and view resources in the cluster:

```bash
# View pods in current namespace
kubectl get pods

# View pods in all namespaces
kubectl get pods --all-namespaces
kubectl get pods -A

# View pods with more details
kubectl get pods -o wide

# View specific pod
kubectl get pod <pod-name>

# View pods with custom output
kubectl get pods -o json
kubectl get pods -o yaml

# View deployments
kubectl get deployments
kubectl get deployment <deployment-name>

# View services
kubectl get services
kubectl get svc <service-name>

# View nodes
kubectl get nodes
kubectl get nodes -o wide

# View configmaps
kubectl get configmaps
kubectl get cm <configmap-name>

# View secrets (names only - values are hidden by default)
kubectl get secrets

# View namespaces
kubectl get namespaces
kubectl get ns

# View all resources in namespace
kubectl get all

# View events
kubectl get events
kubectl get events --sort-by='.lastTimestamp'
```

#### Describe Resources

Get detailed information about resources:

```bash
# Describe a pod
kubectl describe pod <pod-name>

# Describe a deployment
kubectl describe deployment <deployment-name>

# Describe a service
kubectl describe service <service-name>

# Describe a node
kubectl describe node <node-name>

# Describe a namespace
kubectl describe namespace <namespace-name>
```

`describe` shows:
- Resource configuration
- Status and conditions
- Related events
- Labels and annotations
- Resource requests and limits

### Viewing Logs

#### Basic Log Viewing

```bash
# View logs for a pod
kubectl logs <pod-name>

# View logs for a specific container in a pod
kubectl logs <pod-name> -c <container-name>

# Stream logs (follow)
kubectl logs -f <pod-name>

# View logs from previous container instance (after crash)
kubectl logs <pod-name> --previous

# View last N lines of logs
kubectl logs <pod-name> --tail=100

# View logs since a time
kubectl logs <pod-name> --since=1h
kubectl logs <pod-name> --since=30m

# View logs with timestamps
kubectl logs <pod-name> --timestamps

# View logs for all containers in a pod
kubectl logs <pod-name> --all-containers=true

# View logs for pods matching a label
kubectl logs -l app=myapp
```

### Monitoring and Metrics

#### Resource Usage

```bash
# Show metrics for all nodes
kubectl top nodes

# Show metrics for all pods
kubectl top pods

# Show metrics for pods in a namespace
kubectl top pods -n <namespace>

# Show metrics for a specific pod
kubectl top pod <pod-name>

# Show metrics for containers in a pod
kubectl top pod <pod-name> --containers
```

Requires metrics-server to be installed in the cluster.

### Debugging Operations

#### Execute Commands in Containers

Run commands inside containers for debugging:

```bash
# Execute a single command
kubectl exec <pod-name> -- <command>

# Open an interactive shell
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -- /bin/sh

# Execute in a specific container
kubectl exec -it <pod-name> -c <container-name> -- /bin/bash

# Run a command and get output
kubectl exec <pod-name> -- env
kubectl exec <pod-name> -- ps aux
kubectl exec <pod-name> -- curl localhost:8080/health
```

**Note:** `kubectl exec` modifies container state but is generally safe for debugging. Use responsibly.

#### Debug Command

Create debugging sessions:

```bash
# Add a debug container to a running pod
kubectl debug <pod-name> -it --image=busybox:1.28

# Debug with a different image
kubectl debug <pod-name> -it --image=ubuntu

# Create a copy of a pod with debugging enabled
kubectl debug <pod-name> -it --copy-to=<new-pod-name> --container=<container-name>

# Debug a node by creating a pod in the node's host namespaces
kubectl debug node/<node-name> -it --image=busybox:1.28
```

**Security Note:** `kubectl debug` can be dangerous if misused. Use only for legitimate debugging purposes.

#### Port Forwarding

Forward local ports to pods for debugging:

```bash
# Forward local port 8080 to pod port 8080
kubectl port-forward <pod-name> 8080:8080

# Forward local port 9000 to pod port 8080
kubectl port-forward <pod-name> 9000:8080

# Forward to a service
kubectl port-forward service/<service-name> 8080:80

# Forward to a deployment
kubectl port-forward deployment/<deployment-name> 8080:8080

# Listen on all interfaces (not just localhost)
kubectl port-forward --address 0.0.0.0 <pod-name> 8080:8080
```

**Security Note:** Only use port-forward for development and debugging. Not suitable for production traffic.

#### Attach to Running Containers

```bash
# Attach to a running container
kubectl attach <pod-name> -it

# Attach to a specific container
kubectl attach <pod-name> -c <container-name> -it
```

### Context and Configuration

#### Managing Contexts

```bash
# View current context
kubectl config current-context

# List all contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <context-name>

# View current namespace
kubectl config view --minify --output 'jsonpath={..namespace}'

# Set default namespace for current context
kubectl config set-context --current --namespace=<namespace>
```

#### Cluster Information

```bash
# View cluster info
kubectl cluster-info

# View cluster info with debugging details
kubectl cluster-info dump

# View API versions
kubectl api-versions

# View API resources
kubectl api-resources

# View specific API resource details
kubectl explain pod
kubectl explain pod.spec
kubectl explain deployment.spec.replicas
```

### Filtering and Selection

#### Label Selectors

```bash
# Get pods with specific label
kubectl get pods -l app=myapp

# Get pods with multiple labels
kubectl get pods -l app=myapp,env=prod

# Get pods without a label
kubectl get pods -l '!app'

# Get pods with label value in set
kubectl get pods -l 'env in (prod,staging)'
```

#### Field Selectors

```bash
# Get running pods
kubectl get pods --field-selector status.phase=Running

# Get pods on a specific node
kubectl get pods --field-selector spec.nodeName=<node-name>

# Combine multiple field selectors
kubectl get pods --field-selector status.phase=Running,metadata.namespace=default
```

#### Namespace Operations

```bash
# Get resources in a specific namespace
kubectl get pods -n <namespace>

# Get resources in all namespaces
kubectl get pods --all-namespaces
kubectl get pods -A

# Set default namespace for current context
kubectl config set-context --current --namespace=<namespace>
```

### Output Formats

#### JSON and YAML

```bash
# Output as JSON
kubectl get pod <pod-name> -o json

# Output as YAML
kubectl get pod <pod-name> -o yaml

# Pretty-print JSON
kubectl get pod <pod-name> -o json | jq .

# Extract specific field with JSONPath
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[0].image}'

# Extract and format multiple fields
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.phase}{"\n"}{end}'
```

#### Custom Columns

```bash
# Display custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName

# Display pod name and image
kubectl get pods -o custom-columns=NAME:.metadata.name,IMAGE:.spec.containers[0].image
```

#### Wide Output

```bash
# Get additional columns (IP, Node, etc.)
kubectl get pods -o wide
kubectl get nodes -o wide
```

### Watching Resources

```bash
# Watch pods (updates in real-time)
kubectl get pods --watch
kubectl get pods -w

# Watch events
kubectl get events --watch
```

### Copying Files (Use with Caution)

```bash
# Copy file from pod to local
kubectl cp <pod-name>:/path/in/pod /local/path

# Copy file from local to pod
kubectl cp /local/path <pod-name>:/path/in/pod

# Copy from specific container
kubectl cp <pod-name>:/path/in/pod /local/path -c <container-name>
```

**Note:** `kubectl cp` uses `tar` under the hood and can modify container state. Use for debugging only.

### Checking Resource Status

#### Rollout Status

```bash
# Check deployment rollout status (read-only)
kubectl rollout status deployment/<deployment-name>

# View rollout history (read-only)
kubectl rollout history deployment/<deployment-name>
```

**Note:** Do NOT use `kubectl rollout undo` or `kubectl rollout restart` without explicit user approval as these modify cluster state.

## Common Debugging Workflows

### Troubleshooting Pod Issues

```bash
# 1. Check pod status
kubectl get pod <pod-name>

# 2. Get detailed pod information
kubectl describe pod <pod-name>

# 3. View pod logs
kubectl logs <pod-name>

# 4. If pod crashed, view previous logs
kubectl logs <pod-name> --previous

# 5. Check events
kubectl get events --sort-by='.lastTimestamp' | grep <pod-name>

# 6. If pod is running, exec into it
kubectl exec -it <pod-name> -- /bin/sh
```

### Investigating Service Issues

```bash
# 1. Check service
kubectl get service <service-name>
kubectl describe service <service-name>

# 2. Check endpoints
kubectl get endpoints <service-name>

# 3. Verify pods behind service
kubectl get pods -l <service-selector-label>

# 4. Test service connectivity
kubectl port-forward service/<service-name> 8080:80
# Then curl localhost:8080
```

### Examining Node Issues

```bash
# 1. List nodes and status
kubectl get nodes
kubectl get nodes -o wide

# 2. Describe node
kubectl describe node <node-name>

# 3. Check node metrics
kubectl top node <node-name>

# 4. View pods on node
kubectl get pods --all-namespaces --field-selector spec.nodeName=<node-name>

# 5. Check node conditions
kubectl get node <node-name> -o jsonpath='{.status.conditions}'
```

### Monitoring Cluster Health

```bash
# Check cluster components
kubectl get componentstatuses
kubectl get cs

# Check all pods in kube-system namespace
kubectl get pods -n kube-system

# View cluster events
kubectl get events --all-namespaces --sort-by='.lastTimestamp'

# Check node status
kubectl get nodes

# View resource usage
kubectl top nodes
kubectl top pods --all-namespaces
```

## JSONPath Examples

Extract specific data from resources:

```bash
# Get pod IPs
kubectl get pods -o jsonpath='{.items[*].status.podIP}'

# Get container images
kubectl get pods -o jsonpath='{.items[*].spec.containers[*].image}'

# Get pod name and status
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.phase}{"\n"}{end}'

# Get node names
kubectl get nodes -o jsonpath='{.items[*].metadata.name}'

# Get resource requests
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].resources.requests}'
```

## Diff Operations (Read-Only)

View differences before applying changes:

```bash
# See what would change if you applied a file (does NOT apply)
kubectl diff -f <file.yaml>
```

**Note:** This is read-only and safe to use.

## Best Practices for Safe Operations

1. **Always verify namespace** - Use `-n <namespace>` or check current context
2. **Use describe for context** - Get full resource information before taking action
3. **Check logs before exec** - Logs often reveal issues without needing shell access
4. **Use --dry-run for testing** - When testing configurations (does not apply)
5. **Never delete resources** - Absolutely forbidden
6. **Watch for events** - Events often show what went wrong
7. **Use labels effectively** - Filter resources precisely to avoid mistakes
8. **Port-forward for debugging only** - Not for production traffic
9. **Be cautious with exec** - Modifies container state, use responsibly
10. **Always check context** - Verify you're in the right cluster and namespace

## When Destructive Operations Are Requested

If asked to perform any destructive operation (delete, prune, replace --force, etc.):

1. **STOP immediately**
2. **Inform the user** that the operation requires manual intervention
3. **Explain the command** that would be needed (but do NOT execute it)
4. **Recommend alternatives** if applicable (e.g., scaling down instead of deleting)

Example response:
> "I cannot delete Kubernetes resources. To delete this deployment, you will need to manually run: `kubectl delete deployment <name>`. This is a destructive operation that requires manual intervention."

## Summary

**Safe kubectl operations for observability and debugging:**

- `kubectl get` - List and view resources
- `kubectl describe` - Detailed resource information
- `kubectl logs` - View container logs
- `kubectl top` - Resource usage metrics
- `kubectl exec` - Debug with shell access (use responsibly)
- `kubectl debug` - Advanced debugging (use with caution)
- `kubectl port-forward` - Local port forwarding for debugging
- `kubectl config` - Context and configuration management
- `kubectl explain` - API resource documentation
- `kubectl diff` - Preview changes (read-only)

**Absolutely forbidden operations:**

- `kubectl delete` - Never delete resources
- Any destructive commands - Require manual user intervention
