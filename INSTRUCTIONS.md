# Django Todo llist testing instructions
## 1. Applying all the manifests:
```bash
kubectl apply -f .infrastructure
```



## 2. Resource requests and limits
### Container Resources


The `resources` section defines the resource requests and limits for the container.

**`limits`**: Maximum resources the container can consume.
```yaml
limits:
  memory: "128Mi"  # Hard limit on RAM usage. Exceeding this can lead to OOMKilled.
  cpu: "500m"     # Limit on CPU usage (0.5 core). Exceeding this can lead to throttling.
```

**`requests`**: Minimum resources the container needs to run properly (used for scheduling).
```yaml
requests:
  memory: "64Mi"   # Minimum RAM required.
  cpu: "250m"    # Minimum CPU required (0.25 core).
```



## 3. HorizontalPodAutoscaler (HPA)
The HPA automatically adjusts the number of pods in your application based on workload metrics such as CPU utilization.
**Explanation:**
- **`minReplicas: 2`**: A minimum of 2 replicas ensures high availability of the application.
- **`maxReplicas: 5`**: A maximum of 5 replicas to prevent unbounded resource consumption.
- **`averageUtilization: 70%`**: Pods will scale up if CPU utilization exceeds 70% of the allocated requests.



## 4. Deployment Update Strategy
The `Deployment` resource in Kubernetes supports update strategies like `RollingUpdate`, which replaces pods gradually, minimizing downtime during updates.

Example strategy configuration:
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

**Strategy Explanation:**
- **`maxUnavailable: 0`**: During an update 2 pods will be available. It secures 0 downtime for 2 pods at all time.
- **`maxSurge: 1`**: During an update, up to 1 extra pods can be created temporarily.
  This configuration minimizes risks during updates and ensures there’s no interruption to the service's availability.



## 5. Access the Application
Once deployed successfully:
- If a `LoadBalancer` service is used, obtain the external IP with the following command:
  ```bash
  kubectl get svc my-app-service
  ```
- Access the application in a browser or via a tool like `curl` using:
  ```bash
  curl http://<EXTERNAL-IP>
  ```

For a `NodePort` service, use the node's IP address and port.

