# Kubernetes Interview Preparation — Experienced Level

Focus: workloads, scheduling, networking, scaling, storage, deployments, security and production troubleshooting.

## Core Objects

### Q1. What is Kubernetes?
<details><summary>Answer</summary>

A declarative orchestration platform that schedules workloads and continuously reconciles actual state toward desired state.

</details>

### Q2. Pod vs Deployment vs Service?
<details><summary>Answer</summary>

Pod is the basic workload unit; Deployment manages replicated stateless Pods and rollouts; Service provides stable network access to selected Pods.

</details>

### Q3. ReplicaSet?
<details><summary>Answer</summary>

Maintains a desired number of matching Pods. Deployments normally manage ReplicaSets rather than users managing them directly.

</details>

### Q4. Namespace?
<details><summary>Answer</summary>

A logical scope for many Kubernetes resources, useful for organization, access control and policy boundaries.

</details>

### Q5. Labels vs annotations?
<details><summary>Answer</summary>

Labels identify/select objects. Annotations attach metadata that is not generally used for selection.

</details>

## Scheduling & Resources

### Q6. Requests vs limits?
<details><summary>Answer</summary>

Requests influence scheduling and represent expected resource needs. Limits constrain usage. Incorrect values can cause poor placement, throttling or OOM kills.

</details>

### Q7. What is a taint/toleration?
<details><summary>Answer</summary>

A taint repels Pods from a node unless the Pod has a matching toleration. It helps reserve nodes for specific workloads.

</details>

### Q8. Node affinity?
<details><summary>Answer</summary>

Rules influence where Pods should or should not be scheduled based on node labels.

</details>

### Q9. Pod disruption budget?
<details><summary>Answer</summary>

Defines limits on voluntary disruptions so maintenance/eviction does not reduce service availability excessively.

</details>

## Networking

### Q10. How does Service discovery work?
<details><summary>Answer</summary>

Services provide stable virtual endpoints for selected Pods; cluster DNS lets workloads resolve service names.

</details>

### Q11. ClusterIP vs NodePort vs LoadBalancer?
<details><summary>Answer</summary>

ClusterIP is internal virtual service access; NodePort exposes a port on nodes; LoadBalancer integrates with an external load-balancing mechanism where supported.

</details>

### Q12. What is Ingress?
<details><summary>Answer</summary>

A Kubernetes API for HTTP/HTTPS routing into services, implemented by an Ingress Controller.

</details>

### Q13. What is NetworkPolicy?
<details><summary>Answer</summary>

Policies can restrict allowed Pod/network traffic according to the cluster's network implementation and policy support.

</details>

## Health, Scaling & Deployments

### Q14. Liveness vs readiness vs startup probe?
<details><summary>Answer</summary>

Liveness controls restart decisions; readiness controls traffic eligibility; startup protects slow-starting apps from premature liveness failures.

</details>

### Q15. How does HPA work?
<details><summary>Answer</summary>

It changes replica counts based on resource or custom metrics. Effective scaling requires realistic requests and enough startup/downstream capacity.

</details>

### Q16. Rolling update?
<details><summary>Answer</summary>

Gradually replace old Pods with new ones while maintaining desired availability. Readiness and deployment thresholds determine safe progression.

</details>

### Q17. Canary deployment?
<details><summary>Answer</summary>

Gradually expose a new version to a small traffic slice, observe health, then increase exposure.

</details>

### Q18. Blue/green?
<details><summary>Answer</summary>

Maintain separate old/new environments and switch traffic between them, simplifying rollback at the cost of additional capacity.

</details>

## Storage & State

### Q19. StatefulSet?
<details><summary>Answer</summary>

Manages stateful workloads requiring stable identity, ordering or persistent storage associations.

</details>

### Q20. PersistentVolume vs PersistentVolumeClaim?
<details><summary>Answer</summary>

A PV represents storage capacity; a PVC is a workload's request for storage. Storage classes can dynamically provision volumes.

</details>

### Q21. Why avoid databases in Kubernetes sometimes?
<details><summary>Answer</summary>

Managed databases often provide easier backups, patching, failover and operational guarantees. Running databases in Kubernetes is valid but requires strong expertise and operational discipline.

</details>

## Troubleshooting

### Q22. How do you debug CrashLoopBackOff?
<details><summary>Answer</summary>

Inspect describe/events, current and previous logs, exit codes, probes, resources, environment/configuration and dependencies. The status itself is not the root cause.

</details>

### Q23. Pod is Pending—what do you check?
<details><summary>Answer</summary>

Scheduling events, resource requests, node capacity, taints, affinity, PVC binding and admission/policy constraints.

</details>

### Q24. Service returns no traffic—what do you check?
<details><summary>Answer</summary>

Verify selectors match Pod labels, endpoints exist, Pods are Ready, ports/targetPorts match and NetworkPolicy/Ingress rules allow traffic.

</details>

### Q25. Pods are OOMKilled—what next?
<details><summary>Answer</summary>

Check memory requests/limits, actual usage, leaks, workload behavior and node pressure. Increase limits only after understanding the memory profile.

</details>

### Q26. How do you perform zero-downtime deployment?
<details><summary>Answer</summary>

Multiple replicas, readiness probes, rolling/canary strategy, graceful shutdown, connection draining and backward-compatible API/database changes.

</details>

## Question Count

**26 experienced-level questions** in this file.

## Quick Revision Checklist

