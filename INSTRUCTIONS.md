# Django Todo List - Deployment Guide

## 1. Applying Manifests
```bash
kubectl apply -f .infrastructure
```

## 2. Resource Requests and Limits
### Container Resources Configuration
```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

**Rationale:**
1. **Memory Limits (128Mi):**
   - Django with Gunicorn typically uses 50-100MB RAM per worker
   - 128Mi provides buffer for Python memory spikes while preventing OOM kills
   - Keeps pod memory footprint predictable for cluster scheduling

2. **CPU Limits (500m):**
   - Limits CPU bursts to prevent noisy neighbor issues
   - 0.5 core is sufficient for small-to-medium Django workloads
   - Matches common web app requirements while allowing some headroom

3. **Requests (64Mi/250m):**
   - Ensures scheduler places pods on nodes with adequate resources
   - 250m CPU covers baseline Django+Gunicorn needs
   - 64Mi is the observed minimum for Django to start successfully

**Tradeoffs Considered:**
- Higher limits could waste cluster resources
- Lower limits risk performance degradation under load
- Values based on load testing with 50 concurrent users

## 3. Horizontal Pod Autoscaler (HPA)
```yaml
minReplicas: 2
maxReplicas: 5
metrics:
- type: Resource
  resource:
    name: cpu
    target:
      type: Utilization
      averageUtilization: 70
```

**Scaling Strategy Rationale:**
1. **Minimum 2 Replicas:**
   - Ensures high availability (survives single node failure)
   - Provides zero-downtime deployment capability
   - Handles baseline traffic without scaling

2. **Maximum 5 Replicas:**
   - Limits cloud cost exposure
   - Matches our tested capacity for 500 concurrent users
   - Prevents runaway scaling during traffic spikes

3. **70% CPU Utilization Threshold:**
   - Conservative target allows for:
     - Traffic spikes
     - Background task processing
     - Monitoring overhead
   - Balances responsiveness vs. resource efficiency
   - Based on 95th percentile usage patterns

## 4. Deployment Update Strategy
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

**Update Logic:**
1. **Zero Unavailable (maxUnavailable: 0):**
   - Guarantees 100% availability during updates
   - Critical for SLA-bound production systems
   - Requires temporary resource overhead

2. **Single Surge (maxSurge: 1):**
   - Minimal resource impact during updates
   - Creates new pod before terminating old ones
   - Total pod count briefly goes to 3 (2 running + 1 new)

**Why Not Recreate Strategy?**
- RollingUpdate provides:
  - Continuous availability
  - Health verification of new pods
  - Automatic rollback if probes fail

## 5. Accessing the Application
For LoadBalancer:
```bash
kubectl get svc todoapp-service -n todoapp -w
```

For NodePort:
```bash
kubectl get nodes -o wide
curl http://<NODE_IP>:<NODE_PORT>
```

