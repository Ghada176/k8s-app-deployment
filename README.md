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
- **Ingress routing** — Layer 7 HTTP routing by hostname and
