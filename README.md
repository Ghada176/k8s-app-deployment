# k8s-app-deployment

Production-grade Kubernetes deployment manifests for a containerised
web application, demonstrating real-world configuration patterns used
in enterprise environments.

![Kubernetes](https://img.shields.io/badge/Kubernetes-1.25-blue?logo=kubernetes)
![nginx](https://img.shields.io/badge/nginx-1.25--alpine-green?logo=nginx)
![HPA](https://img.shields.io/badge/HPA-enabled-orange)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

---

## What this demonstrates

- **Namespace isolation** — dedicated namespace, never deploying 
  into `default`
- **ConfigMap-driven configuration** — no config baked into the 
  image, fully externalised
- **Liveness + readiness probes** — health checking with a 
  dedicated `/health` endpoint
- **Resource requests and limits** — CPU/memory reservation and 
  hard ceiling per pod
- **RollingUpdate strategy** — zero-downtime deployments with 
  configurable availability guarantees
- **Horizontal Pod Autoscaler** — automatic scaling between 2 and 
  5 replicas based on CPU utilisation
- **ClusterIP Service** — stable internal endpoint, pods never 
  directly exposed
- **Ingress routing** — Layer 7 HTTP routing by hostname and path

---

## Architecture
```
Internet
    │
    ▼
Ingress (webapp.local)          # Layer 7 router — reads host + path
    │
    ▼
Service: webapp-service         # Stable ClusterIP load balancer
(ClusterIP: internal only)      # routes to healthy pods only
    │
    ├──────────────────┐
    ▼                  ▼
Pod 1 (nginx)      Pod 2 (nginx)    # 2 replicas minimum
    │                  │            # HPA scales up to 5 under load
    └──────┬───────────┘
           │
    ConfigMap: webapp-config        # nginx.conf + env vars
           │
    /health endpoint                # liveness + readiness probe target
```

**Kubernetes object hierarchy:**
```
Deployment (desired state + update strategy)
└── ReplicaSet (maintains pod count)
    ├── Pod 1 (runs nginx:1.25-alpine)
    └── Pod 2 (runs nginx:1.25-alpine)
```

---

## Repository structure
```
k8s-app-deployment/
├── manifests/
│   ├── namespace.yaml      # isolates all resources in 'webapp' namespace
│   ├── configmap.yaml      # nginx config + env vars, decoupled from image
│   ├── deployment.yaml     # 2 replicas, probes, resource limits, rolling update
│   ├── service.yaml        # ClusterIP internal load balancer
│   ├── ingress.yaml        # hostname-based HTTP routing
│   └── hpa.yaml            # autoscaler: 2–5 replicas at 70% CPU threshold
├── DECISIONS.md            # engineering rationale for every config choice
└── README.md
```

---

## How to run

**Prerequisites:** minikube, kubectl
```bash
# Clone the repo
git clone git@github.com:Ghada176/k8s-app-deployment.git
cd k8s-app-deployment

# Start minikube and enable ingress
minikube start --driver=docker --cpus=2 --memory=2048
minikube addons enable ingress
minikube addons enable metrics-server

# Apply all manifests
kubectl apply -f manifests/

# Verify everything is running
kubectl get all -n webapp
```

**Expected output:**
```
NAME                          READY   STATUS    RESTARTS   AGE
pod/webapp-69fc54f7fd-854sb   1/1     Running   0          65m
pod/webapp-69fc54f7fd-xsmdt   1/1     Running   0          65m

NAME                      TYPE        CLUSTER-IP       PORT(S)   AGE
service/webapp-service    ClusterIP   10.103.200.141   80/TCP    65m

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp   2/2     2            2           65m

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-69fc54f7fd   2         2         2       65m
```

**Test the health endpoint:**
```bash
kubectl port-forward -n webapp svc/webapp-service 8080:80 &
curl http://localhost:8080/health
# Expected: healthy
```

**Check autoscaler status:**
```bash
kubectl get hpa -n webapp
# NAME         REFERENCE              TARGETS      MINPODS   MAXPODS   REPLICAS
# webapp-hpa   Deployment/webapp      cpu: 2%/70%  2         5         2
```

---

## Key configuration decisions

| Decision | Choice | Reason |
|---|---|---|
| Base image | `nginx:1.25-alpine` | 10 MB vs 140 MB full image — smaller attack surface, faster pull |
| Replicas | 2 minimum | Single replica = downtime on any restart |
| Update strategy | `RollingUpdate` | maxUnavailable=1 ensures zero-downtime deploys |
| Config management | `ConfigMap` | Decoupled from image — update config without rebuilding |
| Service type | `ClusterIP` | Pods never directly exposed — traffic only through Ingress |
| Probes | Liveness + Readiness | Different questions, different consequences — both required |
| Resource limits | Requests + Limits | Requests for scheduling, limits as circuit breaker |
| HPA threshold | 70% CPU | Standard production threshold — headroom before degradation |

See [DECISIONS.md](DECISIONS.md) for detailed engineering rationale.

---

## Concepts covered

### Liveness vs readiness probes
| Probe | Question | Failure consequence |
|---|---|---|
| Liveness | Is the pod alive? | Pod is restarted |
| Readiness | Can it serve traffic? | Removed from Service endpoints |

### Resource requests vs limits
| Field | Meaning | Analogy |
|---|---|---|
| `requests` | What Kubernetes reserves on the node | Booking a hotel room |
| `limits` | Hard ceiling — pod is throttled or OOMKilled if exceeded | Maximum occupancy |

### Traffic flow
```
curl request
  → Ingress reads hostname (webapp.local) and path (/)
  → forwards to Service (webapp-service)
  → Service load-balances across healthy pods
  → Pod handles the request
```

---

## Skills demonstrated

`Kubernetes` `Docker` `nginx` `YAML` `kubectl`
`Health Checks` `Autoscaling` `Ingress` `ConfigMap`
`Resource Management` `Infrastructure as Code`
