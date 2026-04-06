# Architecture & Design Decisions

This document explains the reasoning behind key configuration
choices in this deployment. Written for engineering reviewers
and as a personal reference.

## Why a dedicated namespace?
Namespaces provide resource isolation at the cluster level.
Deploying into `default` makes it impossible to cleanly
scope RBAC policies, resource quotas, or network policies
to a single application. All production workloads should
live in named namespaces.

## Why ConfigMap for nginx configuration?
Configuration should be decoupled from the container image.
Baking config into the image means rebuilding and redeploying
for every config change. With ConfigMap, config updates
are applied by reloading the pod — no new image build needed.
This also makes config changes auditable via Git history.

## Why nginx:alpine over nginx:latest?
The alpine variant is ~10 MB vs ~140 MB for the full image.
Benefits: faster pull times in CI/CD pipelines, smaller
attack surface (fewer packages = fewer CVEs), and lower
storage cost at scale. `latest` tags are also avoided in
production because they are not reproducible — the same
tag can resolve to different images over time.

## Why 2 replicas and RollingUpdate strategy?
A single replica means any pod restart (update, node
failure, OOM kill) causes downtime. Two replicas with
`maxUnavailable: 1` ensures at least one pod is always
serving traffic during rolling updates. In production,
the minimum for any user-facing service is 2 replicas.

## Why separate liveness and readiness probes?
These answer different questions with different consequences:
- **Liveness**: "Is the process alive?" — failure triggers
  a pod restart. Used to recover from deadlocks.
- **Readiness**: "Can this pod serve traffic?" — failure
  removes the pod from the Service endpoints without
  restarting it. Used during startup and graceful shutdown.
Using only liveness would restart a pod that is alive but
still initialising — causing a restart loop.

## Why resource requests AND limits?
- **Requests**: what Kubernetes reserves on the node for
  scheduling. Guarantees the pod gets this much.
- **Limits**: the maximum the pod can consume. Acts as a
  circuit breaker — prevents one pod from starving others
  on the same node (the "noisy neighbour" problem).
Setting only limits without requests can cause scheduling
issues. Setting only requests without limits allows
unbounded memory growth.
