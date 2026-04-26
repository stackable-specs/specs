---
id: kubernetes
layer: platform
extends: []
---

# Kubernetes

## Purpose

Kubernetes is a container-orchestration substrate whose defaults optimize for "this YAML is valid" rather than "this workload will survive a node drain at 3 a.m." A naked Pod that the scheduler placed last week will not come back when its node dies. A container without resource requests is best-effort and gets evicted first; a container without a memory limit can OOM-kill its neighbors; a CPU limit set without thought introduces CFS-throttling latency cliffs that look like a bug in the application. A probe that returns 200 from a static handler tells the kubelet the pod is ready before it actually is. The `default` namespace, the `default` ServiceAccount, automatic token mounting, allow-all networking, plaintext etcd secrets, and `:latest` image tags are each one outage waiting to happen, and together they are the standard "we just need to ship" cluster. This spec pins the cross-workload baseline — declarative state, image pinning, controllers over naked Pods, namespaces and recommended labels, requests/limits/probes, graceful termination and PodDisruptionBudgets, topology spread, the `restricted` Pod Security profile with a hardened `securityContext`, dedicated least-privilege ServiceAccounts, external-secret-manager-backed Secrets, etcd encryption-at-rest, and default-deny NetworkPolicy — so individual application specs (each Deployment, StatefulSet, etc.) can refine on top of a cluster floor instead of redefining it per workload.

## References

- **external** `https://kubernetes.io/` — Kubernetes home
- **external** `https://kubernetes.io/docs/concepts/configuration/overview/` — Configuration best practices
- **external** `https://kubernetes.io/docs/concepts/security/pod-security-standards/` — Pod Security Standards
- **external** `https://kubernetes.io/docs/concepts/security/pod-security-admission/` — Pod Security Admission
- **external** `https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/` — Resource requests and limits
- **external** `https://kubernetes.io/docs/concepts/workloads/pods/disruptions/` — Disruptions and PodDisruptionBudget
- **external** `https://kubernetes.io/docs/concepts/scheduling-eviction/topology-spread-constraints/` — Topology spread constraints
- **external** `https://kubernetes.io/docs/concepts/services-networking/network-policies/` — NetworkPolicy
- **external** `https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/` — Encrypting confidential data at rest (etcd)
- **external** `https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/` — Recommended labels
- **external** `https://github.com/external-secrets/external-secrets` — External Secrets Operator
- **external** `https://kubernetes.io/releases/version-skew-policy/` — Kubernetes version skew and support window

## Rules

1. Manage cluster state declaratively from version-controlled manifests via GitOps (Argo CD, Flux) or `kubectl apply` / `kustomize` / `helm`; do not `kubectl edit` or `kubectl patch` production resources interactively.
2. Pin the Kubernetes server version per cluster and stay within the upstream support window (the current minor plus the previous two); do not run a control-plane version that no longer receives security patches.
3. Pin every container image to a specific immutable tag (e.g. `myapp:1.4.2`) or a digest (`myapp@sha256:…`); do not deploy `:latest` or untagged images to a production namespace.
4. Set `imagePullPolicy: IfNotPresent` for digest- or version-pinned images; do not combine `imagePullPolicy: Always` with a moving tag in production.
5. Run application workloads as a `Deployment`, `StatefulSet`, `Job`, `CronJob`, or `DaemonSet`; do not create naked `Pod`s in production namespaces.
6. Scope every workload to a non-`default` namespace dedicated to its team or environment; do not deploy production workloads to the `default` namespace.
7. Apply the recommended Kubernetes labels (`app.kubernetes.io/name`, `app.kubernetes.io/instance`, `app.kubernetes.io/version`, `app.kubernetes.io/component`, `app.kubernetes.io/part-of`, `app.kubernetes.io/managed-by`) to every workload's pod template.
8. Set `resources.requests` for both CPU and memory on every container; do not deploy a container with no resource requests on a multi-tenant cluster.
9. Set `resources.limits.memory` on every container; do not run unbounded-memory containers that can OOM-kill their neighbors.
10. Set `resources.requests.cpu` and omit `resources.limits.cpu` unless the workload demonstrably needs CPU throttling, since hard CPU limits introduce CFS-throttling latency cliffs.
11. Define a `livenessProbe`, a `readinessProbe`, and (for slow-starting workloads) a `startupProbe` on every long-running container, pointing at an in-process endpoint that exercises real readiness; do not point a probe at a static handler that always returns 200.
12. Set `terminationGracePeriodSeconds` to fit the workload's drain time and add a `preStop` hook or in-process SIGTERM handling that stops accepting new work before exit; do not rely on the default grace period with no drain logic.
13. Configure a `PodDisruptionBudget` for every multi-replica workload; do not let a node drain take all replicas down at once.
14. Configure `topologySpreadConstraints` (or pod anti-affinity) across availability zones / failure domains for production workloads with more than one replica; do not let all replicas land on one node or one zone.
15. Apply the Pod Security `restricted` profile to every application namespace via Pod Security Admission labels (`pod-security.kubernetes.io/enforce: restricted`); reserve `baseline` and `privileged` for namespaces that document why they need them.
16. Set `securityContext.runAsNonRoot: true`, a non-zero `runAsUser` / `runAsGroup`, `allowPrivilegeEscalation: false`, `readOnlyRootFilesystem: true`, `capabilities.drop: ["ALL"]`, and `seccompProfile.type: RuntimeDefault` on every application container.
17. Provide writable scratch via `emptyDir` (or PVC) volumes; do not relax `readOnlyRootFilesystem` to make the container's root writable.
18. Do not use `hostNetwork`, `hostPID`, `hostIPC`, `hostPath` volumes, or `hostPort` on application workloads — reserve them for explicitly-justified infrastructure DaemonSets.
19. Bind every workload to a dedicated `ServiceAccount` (not `default`) and grant least-privilege `Role` / `RoleBinding` (or `ClusterRoleBinding`) permissions; do not bind workloads to `cluster-admin` or to the `default` ServiceAccount.
20. Set `automountServiceAccountToken: false` on workloads that do not call the Kubernetes API; for those that do, mount projected, audience-bound tokens.
21. Source secrets from an external secret manager (External Secrets Operator pulling from AWS Secrets Manager, GCP Secret Manager, Azure Key Vault, or HashiCorp Vault) into projected `Secret` volumes; do not embed secret values in `ConfigMap`, inline `env` vars, or container images.
22. Enable etcd encryption-at-rest (`EncryptionConfiguration` covering `secrets` and other sensitive resources); do not run a production cluster with secrets stored in etcd in plaintext.
23. Enforce a default-deny `NetworkPolicy` per application namespace and explicitly allow required ingress and egress; do not run production namespaces with cluster-wide allow-all networking.
