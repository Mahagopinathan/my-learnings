# Docker & Kubernetes Interview Questions & Answers

Covers containers vs VMs, Docker fundamentals, Dockerfile, networking, volumes, Compose, Kubernetes architecture, workloads, services, configuration, deployments, scaling, observability, and common troubleshooting.

---

## Table of Contents
1. [Containers Fundamentals](#1-containers-fundamentals)
2. [Docker Basics](#2-docker-basics)
3. [Dockerfile & Image Best Practices](#3-dockerfile--image-best-practices)
4. [Docker Networking](#4-docker-networking)
5. [Docker Volumes & Storage](#5-docker-volumes--storage)
6. [Docker Compose](#6-docker-compose)
7. [Kubernetes Architecture](#7-kubernetes-architecture)
8. [Pods, Workloads & Controllers](#8-pods-workloads--controllers)
9. [Services, Ingress & Networking](#9-services-ingress--networking)
10. [Configuration, Secrets & Storage](#10-configuration-secrets--storage)
11. [Scaling, Rollouts & Probes](#11-scaling-rollouts--probes)
12. [Observability & Troubleshooting](#12-observability--troubleshooting)
13. [Common kubectl Commands](#13-common-kubectl-commands)

---

## 1. Containers Fundamentals

### Q1. What is a container?
A lightweight, isolated runtime environment for an application that packages the app with its dependencies. Built on Linux kernel features:
- **Namespaces** – isolation (PID, network, mount, user, IPC, UTS).
- **cgroups** – resource limits (CPU, memory, IO).
- **Layered filesystem** – overlayfs/aufs.

### Q2. Container vs Virtual Machine?
| Aspect | Container | VM |
|--------|-----------|----|
| Isolation | Process-level (shared kernel) | Hardware-level (own kernel) |
| Startup time | Seconds (or less) | Minutes |
| Size | MBs | GBs |
| Density | High (100s per host) | Low |
| OS support | Same kernel as host | Any OS |
| Use case | Microservices, CI/CD, dev | Strong isolation, mixed OSes |

### Q3. Benefits of containerization?
- Consistent environments across dev/test/prod.
- Fast startup and high density.
- Lightweight, easily portable.
- Standardized packaging (OCI images).
- Foundation for microservices and Kubernetes.

---

## 2. Docker Basics

### Q4. What is Docker?
A platform for building, shipping, and running containers. Includes the Docker daemon (`dockerd`), CLI (`docker`), images, containers, registries, networks, and volumes.

### Q5. Docker components?
- **Docker daemon** – runs on host, manages containers/images.
- **Docker CLI** – user interface.
- **Docker images** – immutable, layered templates.
- **Docker containers** – runtime instances of images.
- **Docker registry** – stores images (Docker Hub, ECR, GCR, Artifact Registry, Harbor).

### Q6. Image vs Container?
- **Image** – read-only template (binary + dependencies + metadata).
- **Container** – running instance of an image with a writable layer on top.

### Q7. Common Docker commands
```bash
# Build image
docker build -t myapp:1.0 .

# List images / containers
docker images
docker ps        # running
docker ps -a     # all

# Run container
docker run -d --name api -p 8080:8080 -e ENV=dev myapp:1.0

# Logs / shell
docker logs -f api
docker exec -it api sh

# Stop / remove
docker stop api && docker rm api
docker rmi myapp:1.0

# Push / pull
docker tag myapp:1.0 registry.example.com/myapp:1.0
docker push registry.example.com/myapp:1.0
docker pull registry.example.com/myapp:1.0
```

### Q8. Difference between `CMD`, `ENTRYPOINT`, and `RUN`?
- **`RUN`** – executes a command at **build time**, creating a new image layer (e.g., installing packages).
- **`CMD`** – default command/arguments at **runtime**; overridden by `docker run ... <cmd>`.
- **`ENTRYPOINT`** – fixed executable at runtime; `CMD` provides default args. Use `ENTRYPOINT` for the binary, `CMD` for default flags.

### Q9. Difference between `COPY` and `ADD`?
- **COPY** – copies files/directories. Preferred — explicit and predictable.
- **ADD** – also extracts local tar archives and supports remote URLs. Use only when those features are needed.

### Q10. Difference between `EXPOSE` and `-p`?
- **`EXPOSE`** – documentation-only. Declares ports the container listens on.
- **`-p host:container`** – actually publishes the port to the host network.

---

## 3. Dockerfile & Image Best Practices

### Q11. Sample Dockerfile (Spring Boot, multi-stage)
```Dockerfile
# Build stage
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app
COPY mvnw pom.xml ./
COPY .mvn .mvn
RUN ./mvnw -B dependency:go-offline
COPY src src
RUN ./mvnw -B package -DskipTests

# Runtime stage
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
RUN addgroup -S app && adduser -S app -G app
USER app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

### Q12. What is a multi-stage build?
A build using multiple `FROM` instructions where intermediate stages produce artifacts used by a smaller final image. Keeps build tools out of the runtime image and reduces size dramatically.

### Q13. Image best practices
- Use **specific version tags**, not `latest`.
- Pick **minimal base images** (`alpine`, `distroless`, `slim`).
- Run as a **non-root user**.
- **Order layers** from least- to most-frequently-changing for cache efficiency.
- Use a **`.dockerignore`** to exclude unneeded files.
- Combine related `RUN` commands; clean up apt caches in the same layer.
- Add a **`HEALTHCHECK`** instruction.
- **Pin dependencies**.
- Scan images for vulnerabilities (Trivy, Snyk, Grype).
- Sign and verify images (Cosign, Notary v2).

### Q14. How does Docker layering & caching work?
Each instruction creates a layer. Layers are content-addressable and cached. If a step's inputs (Dockerfile line + context) haven't changed, Docker reuses the cached layer. Reorder steps so frequently-changing ones come last.

### Q15. How to reduce image size?
- Multi-stage builds.
- Smaller base (`alpine`, `distroless`, `scratch`).
- Avoid installing build tools in the final image.
- Combine commands to avoid intermediate layers.
- Remove caches (`apt-get clean`, `pip --no-cache-dir`).

---

## 4. Docker Networking

### Q16. Default Docker network drivers
- **bridge** – default for containers on a single host; isolated subnet.
- **host** – container shares host's network namespace; no isolation.
- **none** – no network.
- **overlay** – multi-host networking (Swarm/K8s style).
- **macvlan** – containers get their own MAC, appear on physical LAN.

### Q17. How do containers talk to each other?
- On the **default bridge** – via IP only.
- On a **user-defined bridge** – also via container name (built-in DNS). Recommended.
- Across hosts – overlay networks or external tooling (Kubernetes CNI).

### Q18. Port publishing vs exposing
- **Publishing** (`-p 8080:80`) – maps host port to container port.
- **Exposing** (`EXPOSE 80`) – metadata only; doesn't publish.

---

## 5. Docker Volumes & Storage

### Q19. Why use volumes?
Container filesystems are **ephemeral**. Volumes provide persistent storage that survives container restarts/removal and can be shared among containers.

### Q20. Types of mounts
- **Volume** – managed by Docker (`/var/lib/docker/volumes`). Best for persistent data.
- **Bind mount** – mounts a host path into the container. Good for dev workflows.
- **tmpfs** – in-memory; fast and ephemeral.

```bash
docker volume create dbdata
docker run -v dbdata:/var/lib/postgresql/data postgres
docker run -v $(pwd)/src:/app/src node:20
```

---

## 6. Docker Compose

### Q21. What is Docker Compose?
A tool to define and run **multi-container** applications via a single YAML file. Useful for local development and small deployments.

### Q22. Sample `docker-compose.yml`
```yaml
version: "3.9"
services:
  api:
    build: .
    image: myapp:1.0
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: dev
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/myapp
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 10s
      retries: 5

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app
    volumes:
      - dbdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app"]
      interval: 5s
      retries: 10

volumes:
  dbdata:
```
Run with `docker compose up -d`.

---

## 7. Kubernetes Architecture

### Q23. What is Kubernetes?
An open-source **container orchestration** platform that automates deployment, scaling, networking, and management of containerized applications across clusters of machines.

### Q24. Why use Kubernetes?
- Self-healing (restart, reschedule).
- Declarative configuration & GitOps.
- Horizontal scaling and load balancing.
- Rolling updates and rollbacks.
- Service discovery, secrets, and config management.
- Cloud-portable abstraction.

### Q25. Kubernetes architecture (high level)
```
                      Control Plane
   ┌──────────────────────────────────────────┐
   │  kube-apiserver  (front door, REST API)  │
   │  etcd            (KV store, source of    │
   │                   truth for cluster)     │
   │  kube-scheduler  (chooses node for pod)  │
   │  controller-mgr  (reconciliation loops)  │
   │  cloud-controller-mgr (cloud integration)│
   └──────────────────────────────────────────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
       Node 1        Node 2         Node 3
   ┌──────────┐  ┌──────────┐   ┌──────────┐
   │ kubelet  │  │ kubelet  │   │ kubelet  │
   │ kube-proxy│ │ kube-proxy│  │ kube-proxy│
   │ container │ │ container │  │ container │
   │ runtime   │ │ runtime   │  │ runtime   │
   │ (containerd│ │           │  │           │
   │  /CRI-O)  │ │           │  │           │
   │ Pods...   │ │ Pods...   │  │ Pods...   │
   └──────────┘  └──────────┘   └──────────┘
```

### Q26. Control plane components?
- **kube-apiserver** – exposes the Kubernetes API; all interactions go through it.
- **etcd** – consistent key/value store for all cluster data.
- **kube-scheduler** – assigns pods to nodes based on resources, affinity, taints, etc.
- **kube-controller-manager** – runs controllers (replicaset, node, deployment, etc.).
- **cloud-controller-manager** – integrates with cloud provider (LBs, volumes, nodes).

### Q27. Node components?
- **kubelet** – agent that runs pods, reports node status to the API server.
- **kube-proxy** – maintains network rules for Service routing (iptables/IPVS/eBPF).
- **Container runtime** – containerd, CRI-O (Docker engine is no longer used directly since K8s 1.24).

### Q28. What is etcd?
A distributed, strongly consistent key-value store using the Raft consensus algorithm. Stores **all** cluster state. Must be backed up.

### Q29. What is the kubelet?
The primary node agent. Watches the API server for pods scheduled to its node and ensures the containers described are running and healthy.

---

## 8. Pods, Workloads & Controllers

### Q30. What is a Pod?
The smallest deployable unit in Kubernetes. A pod hosts one or more **tightly coupled containers** sharing:
- A **network namespace** (same IP, same localhost).
- **Storage volumes**.
- A **lifecycle** (scheduled together).

### Q31. Multi-container pod patterns?
- **Sidecar** – auxiliary container (log shipper, proxy).
- **Ambassador** – proxy to remote services.
- **Adapter** – normalize output (e.g., metrics format).
- **Init container** – runs to completion before main containers start.

### Q32. ReplicaSet vs Deployment?
- **ReplicaSet** ensures N copies of a pod are running.
- **Deployment** manages ReplicaSets and provides **rolling updates, rollbacks, and revision history**. Use Deployments for stateless apps.

### Q33. StatefulSet?
For **stateful** apps that need stable identity and storage:
- Stable network names (`web-0`, `web-1`).
- Stable PersistentVolumes per pod.
- Ordered, graceful deployment, scaling, and termination.
Examples: Kafka, Elasticsearch, Cassandra, MySQL.

### Q34. DaemonSet?
Runs a pod copy on **every node** (or matching nodes) — log collectors, node exporters, network plugins.

### Q35. Job vs CronJob?
- **Job** – run-to-completion task (batch); retries on failure.
- **CronJob** – schedule-based Job (e.g., nightly backup).

### Q36. Sample Deployment manifest
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  labels: { app: web }
spec:
  replicas: 3
  selector:
    matchLabels: { app: web }
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels: { app: web }
    spec:
      containers:
        - name: web
          image: myapp:1.0
          ports:
            - containerPort: 8080
          resources:
            requests: { cpu: "100m", memory: "256Mi" }
            limits:   { cpu: "500m", memory: "512Mi" }
          readinessProbe:
            httpGet: { path: /actuator/health/readiness, port: 8080 }
            initialDelaySeconds: 10
          livenessProbe:
            httpGet: { path: /actuator/health/liveness, port: 8080 }
            initialDelaySeconds: 30
```

---

## 9. Services, Ingress & Networking

### Q37. What is a Service?
A stable network endpoint that fronts a dynamic set of pods (selected by labels). Provides DNS, load balancing, and service discovery within the cluster.

### Q38. Service types
- **ClusterIP** (default) – internal-only virtual IP.
- **NodePort** – exposes service on a static port on every node (`30000-32767`).
- **LoadBalancer** – provisions a cloud LB (ELB, ALB, GCLB) pointing to the service.
- **ExternalName** – CNAME to an external DNS name (no proxy).
- **Headless** (`clusterIP: None`) – returns pod IPs directly via DNS; used by StatefulSets.

### Q39. What is Ingress?
HTTP(S) traffic routing into the cluster, with host/path-based rules, TLS termination, etc. Requires an **Ingress Controller** (nginx, Traefik, HAProxy, AWS ALB, GKE Ingress).

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
    - hosts: [api.example.com]
      secretName: api-tls
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port: { number: 80 }
```

### Q40. Cluster networking model?
Kubernetes requires:
- All pods can reach all pods across nodes without NAT.
- All nodes can reach all pods.
- A pod sees itself with the same IP others see.

Implemented by **CNI plugins** (Calico, Cilium, Flannel, AWS VPC CNI, Azure CNI).

### Q41. NetworkPolicy?
Defines pod-level ingress/egress rules at L3/L4 (similar to a firewall). Requires a CNI that supports them (Calico, Cilium).

### Q42. Service discovery
- Each Service is registered in cluster DNS (`my-svc.my-ns.svc.cluster.local`).
- Pods auto-resolve services by name.

---

## 10. Configuration, Secrets & Storage

### Q43. ConfigMap vs Secret?
- **ConfigMap** – non-sensitive config (env vars, files).
- **Secret** – sensitive data (passwords, tokens, certs); base64-encoded by default. Encrypt etcd at rest and use external KMS / Sealed Secrets / External Secrets Operator for production.

### Q44. Mounting config
```yaml
envFrom:
  - configMapRef: { name: app-config }
  - secretRef:    { name: app-secret }
volumeMounts:
  - name: tls
    mountPath: /etc/tls
    readOnly: true
volumes:
  - name: tls
    secret: { secretName: tls-cert }
```

### Q45. Persistent storage
- **PersistentVolume (PV)** – cluster-level storage resource (provisioned statically or dynamically).
- **PersistentVolumeClaim (PVC)** – user request for storage; binds to a matching PV.
- **StorageClass** – defines provisioner & parameters for dynamic provisioning.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata: { name: data }
spec:
  accessModes: [ReadWriteOnce]
  resources: { requests: { storage: 10Gi } }
  storageClassName: gp3
```

### Q46. Access modes?
- **ReadWriteOnce (RWO)** – one node read/write.
- **ReadOnlyMany (ROX)** – many nodes read.
- **ReadWriteMany (RWX)** – many nodes read/write (NFS, CephFS).
- **ReadWriteOncePod** (1.22+) – exactly one pod.

---

## 11. Scaling, Rollouts & Probes

### Q47. Health probes
- **livenessProbe** – when failing, kubelet **restarts** the container.
- **readinessProbe** – when failing, the pod is **removed from service endpoints** (no traffic) but not restarted.
- **startupProbe** – allows slow-starting apps to take longer before liveness kicks in.

### Q48. Resource requests vs limits
- **Requests** – guaranteed minimum; used for scheduling.
- **Limits** – hard cap; container is throttled (CPU) or OOM-killed (memory) if exceeded.

### Q49. QoS classes
- **Guaranteed** – every container has equal requests & limits set.
- **Burstable** – at least one container has requests but not all limits == requests.
- **BestEffort** – no requests/limits set. First to be evicted.

### Q50. Horizontal Pod Autoscaler (HPA)
Scales pod replicas based on CPU/memory or custom/external metrics.
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata: { name: web }
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target: { type: Utilization, averageUtilization: 70 }
```

### Q51. Vertical Pod Autoscaler (VPA)
Adjusts pod CPU/memory **requests/limits** (not replicas) based on historical usage. Don't combine VPA with HPA on CPU/memory of the same workload.

### Q52. Cluster Autoscaler
Adjusts the **number of nodes** in a node pool based on pending unschedulable pods.

### Q53. Deployment strategies
- **RollingUpdate** (default) – gradually replace old pods with new.
- **Recreate** – stop all old pods then create new (downtime).
- **Blue-Green** – two parallel deployments; switch service selector. Easy with two Deployments + Service.
- **Canary** – send a small % of traffic to new version. Use Argo Rollouts, Flagger, or weighted Ingress.

### Q54. Rollback
```bash
kubectl rollout history deployment/web
kubectl rollout undo deployment/web
kubectl rollout undo deployment/web --to-revision=3
```

---

## 12. Observability & Troubleshooting

### Q55. Logs in Kubernetes
- Containers log to stdout/stderr.
- Cluster-level aggregation: Fluent Bit / Fluentd / Vector → Elasticsearch/Loki/Splunk.
- `kubectl logs <pod> [-c container] [--previous]`.

### Q56. Metrics
- **metrics-server** – live CPU/memory for HPA & `kubectl top`.
- **Prometheus** – pull-based scraping, time-series store.
- **Grafana** – dashboards.
- **kube-state-metrics** – metrics about cluster objects.
- **Node exporter** – node-level metrics.

### Q57. Tracing
- OpenTelemetry instrumentation in apps.
- Backends: Jaeger, Tempo, Zipkin, vendor APMs.
- Trace context propagation via W3C `traceparent` header.

### Q58. Common pod statuses
- **Pending** – waiting for scheduling/PVC/image pull.
- **ContainerCreating** – pulling image, attaching volumes.
- **Running** – at least one container is running.
- **CrashLoopBackOff** – container keeps crashing.
- **ImagePullBackOff / ErrImagePull** – wrong image, no permission.
- **OOMKilled** – container exceeded memory limit.
- **Evicted** – removed due to node pressure.

### Q59. How to debug a failing pod
```bash
kubectl get pods
kubectl describe pod <pod>          # events, conditions, image, probes
kubectl logs <pod> --previous       # last container's logs
kubectl exec -it <pod> -- sh        # shell in
kubectl get events --sort-by=.lastTimestamp
kubectl top pod                     # resource usage
```

### Q60. Common failure causes
- Missing `imagePullSecret` for private registries.
- Wrong env vars / missing ConfigMap or Secret.
- Probes too aggressive (frequent restarts).
- Memory limit too low → OOM kills.
- PVC binding failures (no matching PV / wrong storageClass).
- Service selector doesn't match pod labels (no endpoints).
- NetworkPolicy blocking traffic.
- DNS issues (CoreDNS overloaded).

### Q61. RBAC basics
- **Role / ClusterRole** – set of permissions.
- **RoleBinding / ClusterRoleBinding** – bind a Role to a user/group/ServiceAccount.
- **ServiceAccount** – identity for pods to access the API.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata: { name: pod-reader, namespace: dev }
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

### Q62. What is a Helm chart?
A package of Kubernetes manifests templated with values. Helm is the de facto package manager for Kubernetes.
- `Chart.yaml` (metadata), `values.yaml` (defaults), `templates/` (Go-templated manifests).
- Commands: `helm install`, `helm upgrade`, `helm rollback`, `helm template`.

### Q63. What is an Operator?
A pattern where a custom controller manages a complex stateful application (e.g., databases) via **Custom Resource Definitions (CRDs)**. Frameworks: Operator SDK, Kubebuilder.

### Q64. Namespaces – when to use?
- Multi-tenant clusters.
- Separate environments (dev/staging/prod) on a shared cluster (with caveats).
- Quotas and RBAC scoped per namespace.

### Q65. Taints & Tolerations vs Node Affinity
- **Taint** on node + matching **Toleration** on pod → controls which pods can be scheduled where (e.g., dedicate nodes to GPU workloads).
- **Node Affinity** (`requiredDuringScheduling…` / `preferredDuringScheduling…`) – pod opts in to certain nodes by labels.

---

## 13. Common kubectl Commands

```bash
# Cluster info
kubectl cluster-info
kubectl get nodes -o wide
kubectl version

# Pods / Deployments / Services
kubectl get pods -A
kubectl get deploy,svc,ingress -n web
kubectl describe deploy web

# Apply / delete manifests
kubectl apply -f deployment.yaml
kubectl delete -f deployment.yaml
kubectl apply -k ./overlays/prod   # kustomize

# Scale / rollout
kubectl scale deploy web --replicas=5
kubectl rollout status deploy/web
kubectl rollout undo deploy/web

# Exec / port-forward
kubectl exec -it pod/web-abc -- sh
kubectl port-forward svc/web 8080:80
kubectl logs -f deploy/web -c app

# Debug helpers
kubectl get events --sort-by=.lastTimestamp
kubectl top pod -A
kubectl describe pod <pod>
kubectl run tmp --rm -it --image=busybox -- sh

# Config / context
kubectl config current-context
kubectl config use-context my-cluster
kubectl config view --minify
```
