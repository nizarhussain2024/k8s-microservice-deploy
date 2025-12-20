# Kubernetes Microservice Deployment

Kubernetes deployment configurations and manifests for microservices.

## Architecture

### Kubernetes Cluster Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                        │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              Control Plane                            │  │
│  │  - API Server                                         │  │
│  │  - etcd (State Store)                                │  │
│  │  - Scheduler                                          │  │
│  │  - Controller Manager                                │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              Worker Nodes                             │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐           │  │
│  │  │   Pod    │  │   Pod    │  │   Pod    │           │  │
│  │  │ (App 1)  │  │ (App 2)  │  │ (App 3)  │           │  │
│  │  └──────────┘  └──────────┘  └──────────┘           │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐           │  │
│  │  │   Pod    │  │   Pod    │  │   Pod    │           │  │
│  │  │ (App 1)  │  │ (App 2)  │  │ (App 3)  │           │  │
│  │  └──────────┘  └──────────┘  └──────────┘           │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              Networking                               │  │
│  │  - Services (ClusterIP, NodePort, LoadBalancer)       │  │
│  │  - Ingress Controller                                 │  │
│  │  - Network Policies                                   │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              Storage                                  │  │
│  │  - Persistent Volumes                                │  │
│  │  - Storage Classes                                    │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Deployment Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    External Traffic                          │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              Ingress Controller                        │  │
│  │  - SSL Termination                                    │  │
│  │  - Routing Rules                                      │  │
│  └──────────────────────────────────────────────────────┘  │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        │
┌───────────────────────▼─────────────────────────────────────┐
│                    Service Layer                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │         Kubernetes Service (LoadBalancer)             │  │
│  │  - Load balancing                                    │  │
│  │  - Service discovery                                 │  │
│  └──────────────────────────────────────────────────────┘  │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        │
┌───────────────────────▼─────────────────────────────────────┐
│                    Deployment Layer                         │
│  ┌──────────────────────────────────────────────────────┐  │
│  │         Deployment                                    │  │
│  │  - ReplicaSet management                             │  │
│  │  - Rolling updates                                   │  │
│  │  - Health checks                                     │  │
│  └──────────────────────────────────────────────────────┘  │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        │
┌───────────────────────▼─────────────────────────────────────┐
│                      Pod Layer                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │         Pods (Containers)                             │  │
│  │  - Application containers                            │  │
│  │  - Sidecar containers (optional)                     │  │
│  │  - Init containers (optional)                        │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## Design Decisions

### Deployment Strategy
- **Rolling Update**: Zero-downtime deployments
- **Health Checks**: Liveness and readiness probes
- **Resource Limits**: CPU and memory constraints
- **Replicas**: High availability with multiple pods

### Networking
- **Service Type**: ClusterIP for internal, LoadBalancer for external
- **Ingress**: Nginx or Traefik for external access
- **DNS**: Kubernetes DNS for service discovery

### Storage
- **Persistent Volumes**: For stateful applications
- **ConfigMaps**: For configuration
- **Secrets**: For sensitive data

## End-to-End Flow

### Flow 1: Deploy Application

```
1. Prepare Deployment Manifest
   └─> Create deployment.yaml:
       ├─> Deployment spec
       ├─> Container image
       ├─> Resource limits
       ├─> Environment variables
       └─> Health checks

2. Apply Deployment
   └─> kubectl apply -f deployment.yaml
       └─> Kubernetes API server receives manifest

3. Deployment Controller Processing
   └─> Deployment controller:
       ├─> Parse deployment spec
       ├─> Create ReplicaSet
       └─> Set desired replica count (e.g., 3)

4. ReplicaSet Creation
   └─> ReplicaSet controller:
       ├─> Create ReplicaSet object
       └─> Ensure desired number of pods

5. Pod Scheduling
   └─> Scheduler:
       ├─> Select nodes for pods
       ├─> Consider:
       │   ├─> Resource availability
       │   ├─> Node affinity
       │   ├─> Pod anti-affinity
       │   └─> Taints and tolerations
       └─> Assign pods to nodes

6. Container Runtime
   └─> Kubelet on each node:
       ├─> Pull container image
       ├─> Create container
       ├─> Start container
       └─> Report status to API server

7. Pod Initialization
   └─> Pod lifecycle:
       ├─> Init containers (if any)
       ├─> Main container starts
       ├─> Liveness probe begins
       └─> Readiness probe begins

8. Service Registration
   └─> Service controller:
       ├─> Create Service object
       ├─> Create endpoints
       ├─> Register with DNS
       └─> Configure load balancing

9. Ingress Configuration (if external)
   └─> Ingress controller:
       ├─> Read Ingress resource
       ├─> Configure routing rules
       └─> Update load balancer

10. Deployment Complete
    └─> All pods running
        └─> Service accessible
            └─> Application ready
```

### Flow 2: Rolling Update

```
1. Update Deployment
   └─> kubectl set image deployment/app app=app:v2
       └─> Update deployment spec with new image

2. Deployment Controller
   └─> Detect change:
       ├─> Create new ReplicaSet
       ├─> Set desired replicas: 3
       └─> Start rolling update

3. Pod Creation (New Version)
   └─> New ReplicaSet:
       ├─> Create pod with new image
       ├─> Wait for readiness probe
       └─> Mark pod as ready

4. Pod Termination (Old Version)
   └─> Old ReplicaSet:
       ├─> Receive termination signal
       ├─> Graceful shutdown (30s default)
       ├─> Stop accepting new traffic
       └─> Terminate pod

5. Rolling Update Process
   └─> Repeat until all pods updated:
       ├─> Create 1 new pod
       ├─> Wait for readiness
       ├─> Terminate 1 old pod
       └─> Continue until all updated

6. Update Complete
   └─> All pods running new version
       └─> Old ReplicaSet scaled to 0
           └─> Deployment complete
```

### Flow 3: Service Discovery

```
1. Application Startup
   └─> Pod starts application
       └─> Application needs to connect to database service

2. DNS Lookup
   └─> Application queries DNS:
       ├─> Service name: database-service
       ├─> Namespace: default
       └─> Full DNS: database-service.default.svc.cluster.local

3. DNS Resolution
   └─> Kubernetes DNS (CoreDNS):
       ├─> Resolve service name to ClusterIP
       └─> Return IP address: 10.96.0.1

4. Service Routing
   └─> Application connects to ClusterIP
       └─> kube-proxy:
           ├─> Intercepts connection
           ├─> Load balances to pod endpoints
           └─> Routes to healthy pod

5. Connection Established
   └─> Application connected to database pod
       └─> Communication proceeds
```

## Deployment Manifest Structure

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nizar-sample-app
spec:
  replicas: 3                    # Number of pod replicas
  selector:
    matchLabels:
      app: sample
  template:
    metadata:
      labels:
        app: sample
    spec:
      containers:
      - name: sample
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

## Kubernetes Resources

### Core Resources
- **Deployment**: Manages ReplicaSets and pods
- **Service**: Exposes pods internally/externally
- **ConfigMap**: Configuration data
- **Secret**: Sensitive data
- **Ingress**: External HTTP access

### Advanced Resources
- **HorizontalPodAutoscaler**: Auto-scaling
- **PodDisruptionBudget**: Availability guarantees
- **NetworkPolicy**: Network security
- **PersistentVolumeClaim**: Storage

## Deployment Strategies

### Rolling Update (Default)
- Gradual replacement of pods
- Zero downtime
- Rollback capability

### Blue-Green
- Two identical environments
- Switch traffic instantly
- Quick rollback

### Canary
- Gradual traffic shift
- Test new version with subset
- Monitor metrics

## Build & Deploy

### Prerequisites
- Kubernetes cluster (local or cloud)
- kubectl configured
- Container registry access

### Deploy
```bash
# Apply deployment
kubectl apply -f k8s/deployment.yaml

# Check status
kubectl get deployments
kubectl get pods

# View logs
kubectl logs -f deployment/nizar-sample-app

# Scale deployment
kubectl scale deployment nizar-sample-app --replicas=5

# Update deployment
kubectl set image deployment/nizar-sample-app sample=nginx:1.21

# Rollback
kubectl rollout undo deployment/nizar-sample-app
```

### Delete
```bash
kubectl delete -f k8s/deployment.yaml
```

## Future Enhancements

- [ ] Service mesh integration (Istio)
- [ ] Auto-scaling (HPA)
- [ ] Multi-environment configs (dev, staging, prod)
- [ ] Helm charts
- [ ] GitOps (ArgoCD/Flux)
- [ ] Monitoring integration (Prometheus)
- [ ] Logging aggregation (ELK stack)
- [ ] Security policies (Pod Security Standards)
- [ ] Resource quotas
- [ ] Network policies

## AI/NLP Capabilities

This project includes AI and NLP utilities for:
- Text processing and tokenization
- Similarity calculation
- Natural language understanding

*Last updated: 2025-12-20*
