# ClusterIP with Labels Lab

This directory contains examples of pods and services using labels for pod selection.

## Files:
- `pod-1.yaml` - Pod with label `curso: 2TCNPZ` (nginx)
- `pod-2.yaml` - Pod without labels (busybox)
- `pod-3.yaml` - Pod without labels (alpine/curl)
- `pod-sem-label.yaml` - Pod with different label `ambiente: teste` (nginx)
- `svc-pod-1.yaml` - Service targeting pods with label `curso: 2TCNPZ`
- `svc-pod-2.yaml` - Service targeting pods with label `app: segundo-pod`

## Lab Objectives:
- Understand how services use label selectors
- Practice pod-to-pod communication
- Test service connectivity and Nginx responses

## Commands for Students to Try

### 1. Deploy all resources
```bash
kubectl apply -f pod-1.yaml
kubectl apply -f pod-2.yaml
kubectl apply -f pod-3.yaml
kubectl apply -f pod-sem-label.yaml
kubectl apply -f svc-pod-1.yaml
kubectl apply -f svc-pod-2.yaml
```

### 2. Verify pod deployment and labels
```bash
# Check all pods
kubectl get pods --show-labels

# Check specific pod details
kubectl describe pod pod-1
kubectl describe pod pod-sem-label
```

### 3. Verify services and endpoints
```bash
# Check services
kubectl get services

# Check service endpoints (which pods are selected)
kubectl get endpoints
kubectl describe service svc-pod-1
kubectl describe service svc-pod-2
```

### 4. Test connectivity between pods

#### From pod-2 (busybox) to nginx pods
```bash
# Get pod IPs first
kubectl get pods -o wide

# Test direct pod-to-pod connectivity
kubectl exec -it pod-2 -- wget -qO- <pod-1-ip>:80
kubectl exec -it pod-2 -- wget -qO- <pod-sem-label-ip>:80

# Test service connectivity
kubectl exec -it pod-2 -- wget -qO- svc-pod-1:80
kubectl exec -it pod-2 -- nslookup svc-pod-1
```

#### From pod-3 (alpine/curl) to nginx pods
```bash
# Test direct pod connectivity
kubectl exec -it pod-3 -- curl <pod-1-ip>:80
kubectl exec -it pod-3 -- curl <pod-sem-label-ip>:80

# Test service connectivity
kubectl exec -it pod-3 -- curl svc-pod-1:80
kubectl exec -it pod-3 -- nslookup svc-pod-1
```

### 5. Test Nginx responses

#### Test default Nginx page
```bash
# From busybox pod
kubectl exec -it pod-2 -- wget -qO- svc-pod-1:80

# From curl pod
kubectl exec -it pod-3 -- curl svc-pod-1:80

# Test with headers
kubectl exec -it pod-3 -- curl -I svc-pod-1:80
```

#### Test service that has no matching pods
```bash
# This should fail because no pods have label "app: segundo-pod"
kubectl exec -it pod-2 -- wget -qO- svc-pod-2:80
kubectl exec -it pod-3 -- curl svc-pod-2:80
```

### 6. Connectivity troubleshooting commands
```bash
# Check DNS resolution
kubectl exec -it pod-2 -- nslookup svc-pod-1
kubectl exec -it pod-3 -- nslookup svc-pod-1

# Check if port is reachable
kubectl exec -it pod-2 -- nc -zv svc-pod-1 80
kubectl exec -it pod-3 -- nc -zv svc-pod-1 80

# Test connectivity to pod IPs directly
kubectl exec -it pod-2 -- ping <pod-1-ip>
kubectl exec -it pod-3 -- ping <pod-1-ip>
```

### 7. Advanced testing
```bash
# Check service load balancing (if multiple pods matched)
for i in {1..5}; do kubectl exec -it pod-3 -- curl -s svc-pod-1 | grep -i server; done

# Monitor network traffic
kubectl exec -it pod-2 -- netstat -tuln
kubectl exec -it pod-1 -- netstat -tuln
```

## Expected Results

### Successful connections:
- pod-2 → pod-1 (direct IP): ✅ Nginx welcome page
- pod-3 → pod-1 (direct IP): ✅ Nginx welcome page
- pod-2 → svc-pod-1: ✅ Nginx welcome page (routes to pod-1)
- pod-3 → svc-pod-1: ✅ Nginx welcome page (routes to pod-1)

### Failed connections:
- Any pod → svc-pod-2: ❌ Connection refused (no matching pods)

## Key Learning Points
1. Services only route to pods with matching labels
2. Pod-to-pod communication works with direct IPs
3. Service DNS names resolve within the cluster
4. Services provide load balancing across matching pods
5. Wrong label selectors result in no endpoints

## Cleanup
```bash
kubectl delete -f .
```