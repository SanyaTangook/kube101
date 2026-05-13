# kube101 — AGENTS.md

Kubernetes 101 learning repo (Thai language). Teaching material, not production code.

## Structure

- `c01/` — 10 K8s YAML manifests (pod, deployment, service, storageclass, pv, pvc, ingress, configmap, secret, hpa) + README.md
- `c02/` — 3 workload manifests for teaching: Deployment, DaemonSet, StatefulSet

## Applying manifests

```bash
kubectl apply -f c01/     # apply all
kubectl apply -f c01/1-pod.yaml  # apply single resource
kubectl delete -f c01/1-pod.yaml
```

## Labs

`labs.md` — 7 lab exercises covering Pod, Deployment+Service, ConfigMap+Secret, DaemonSet, StatefulSet, HPA, and workload comparison.

## No code/build/tooling

This repo has no package.json, no CI, no lint, no test framework, no Dockerfile. Every file is a standalone K8s YAML resource. No commands beyond `kubectl`.

## Key resource relationships (from c01/README.md)

```
StorageClass → PersistentVolume → PersistentVolumeClaim → Pod/Deployment (← ConfigMap, Secret)
                                                         → HPA → Service → Ingress
```
