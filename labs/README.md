# Kubernetes Labs

### Lab 1: Basic Pod Creation

**Scenario:** Your team needs to deploy a simple nginx web server for testing purposes.

**Requirements:**
1. Create a Pod named `nginx-test`
2. Use the `nginx:1.25` image
3. Expose port 80 inside the container
4. Add labels: `app=nginx`, `environment=test`

**Solution:** [kubernetes/lab01-basic-pod.yaml](kubernetes/lab01-basic-pod.yaml)

---

### Lab 2: Multi-Container Pod

**Scenario:** Deploy an application that requires a main container and a sidecar logging container.

**Requirements:**
1. Create a Pod named `multi-container-app`
2. Main container: `nginx:1.25` on port 80
3. Sidecar container: `busybox` that tails logs from a shared volume
4. Use an `emptyDir` volume mounted at `/var/log/nginx` in both containers

**Solution:** [kubernetes/lab02-multi-container.yaml](kubernetes/lab02-multi-container.yaml)

---

### Lab 3: Deployment with Rolling Updates

**Scenario:** Deploy a production application with zero-downtime updates.

**Requirements:**
1. Create a Deployment named `web-app`
2. Use 3 replicas
3. Image: `nginx:1.24`
4. Configure rolling update strategy: maxSurge=1, maxUnavailable=0
5. Add resource limits: 128Mi memory, 100m CPU
6. Add readiness probe on port 80

**Solution:** [kubernetes/lab03-deployment.yaml](kubernetes/lab03-deployment.yaml)

---

### Lab 4: Service and Ingress

**Scenario:** Expose your deployment externally with proper load balancing.

**Requirements:**
1. Create a ClusterIP Service named `web-service` targeting the `web-app` deployment
2. Create an Ingress resource with host `myapp.example.com`
3. Route traffic from path `/` to the service

**Solution:** [kubernetes/lab04-service-ingress.yaml](kubernetes/lab04-service-ingress.yaml)

---

### Lab 5: ConfigMaps and Secrets

**Scenario:** Configure an application with environment-specific settings and sensitive data.

**Requirements:**
1. Create a ConfigMap named `app-config` with:
   - `LOG_LEVEL=info`
   - `APP_ENV=production`
2. Create a Secret named `app-secrets` with:
   - `DB_PASSWORD=supersecret123`
3. Create a Pod that uses both as environment variables

**Solution:** [kubernetes/lab05-configmap-secret.yaml](kubernetes/lab05-configmap-secret.yaml)

---

### Lab 6: Persistent Storage

**Scenario:** Deploy a stateful database that persists data across pod restarts.

**Requirements:**
1. Create a PersistentVolumeClaim named `db-pvc` requesting 1Gi storage
2. Create a Deployment for `postgres:15` using the PVC
3. Mount the volume at `/var/lib/postgresql/data`
4. Set required environment variables for PostgreSQL

**Solution:** [kubernetes/lab06-persistent-storage.yaml](kubernetes/lab06-persistent-storage.yaml)

---

### Lab 7: Horizontal Pod Autoscaler

**Scenario:** Automatically scale your application based on CPU usage.

**Requirements:**
1. Create a Deployment with resource requests defined
2. Create an HPA targeting 50% CPU utilization
3. Scale between 2 and 10 replicas

**Solution:** [kubernetes/lab07-hpa.yaml](kubernetes/lab07-hpa.yaml)

---

### Lab 8: StatefulSet with Headless Service

**Scenario:** Deploy a distributed database cluster with stable network identities.

**Requirements:**
1. Create a Headless Service for stable DNS entries
2. Create a StatefulSet with 3 replicas
3. Each pod should have its own PVC (volumeClaimTemplates)
4. Use `podManagementPolicy: OrderedReady`

**Solution:** [kubernetes/lab8-statefulset.yaml](kubernetes/lab10-statefulset.yaml)

---
