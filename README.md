# Security-Focused Kubernetes Deployment  

Minimal, security-hardened Kubernetes deployment of **httpbin** with even pod distribution and Trivy security reports.

> Verified repo contents: `trivy/` folder with scan of image as well, because it looks not very reliable


## Even Distribution & Scheduling Logic

This Deployment uses a *hard* topology spread rule to keep replicas evenly spread across nodes,
and a *soft* (preferred) pod anti-affinity. Together they produce resilient,
well-balanced scheduling without over-constraining the cluster.

```yaml
# --- Hard balancing rule: keep replica counts even across nodes ---
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: kubernetes.io/hostname
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app: httpbin

# --- Soft preference: avoid co-locating httpbin pods on the same node when alternatives exist ---
affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          topologyKey: kubernetes.io/hostname
          labelSelector:
            matchLabels:
              app: httpbin
```
## Security Hardening

This deployment follows Kubernetes and container security best practices to minimize attack surface and enforce the principle of least privilege.

Pod-level Security Context:
```yaml
securityContext:
  seccompProfile:
    type: RuntimeDefault
  runAsNonRoot: true
  runAsUser: 1000
  runAsGroup: 1000
  fsGroup: 2000     
```

Container-level Security Context:
```yaml
securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop: ["ALL"]    
```
