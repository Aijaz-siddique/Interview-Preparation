# Kubernetes Interview Preparation

### Q1. Pod vs Deployment vs Service?
<details><summary>Answer</summary>

A Pod is the smallest deployable unit and contains one or more containers.

A Deployment manages replicated Pods and rollout/rollback behavior.

A Service provides stable network access to a set of Pods selected by labels.
</details>

### Q2. Explain Kubernetes desired state.
<details><summary>Answer</summary>

You declare the desired state, such as replicas and image version. Controllers continuously compare actual state with desired state and take corrective actions.

This reconciliation model is fundamental to Kubernetes.
</details>

### Q3. Requests vs limits?
<details><summary>Answer</summary>

Requests influence scheduling and represent expected resource requirements.

Limits constrain resource usage.

Incorrect values can cause poor scheduling, throttling or OOM kills.
</details>

### Q4. Liveness vs readiness vs startup probes?
<details><summary>Answer</summary>

Liveness determines whether the container should be restarted.

Readiness determines whether it should receive traffic.

Startup probe allows slow-starting applications time to initialize before liveness checks become active.
</details>

### Q5. ConfigMap vs Secret?
<details><summary>Answer</summary>

ConfigMaps hold non-secret configuration.

Secrets are intended for sensitive values, although their security still depends on cluster configuration, access control and encryption.

Never assume a Kubernetes Secret automatically solves secret-management requirements.
</details>

### Q6. Deployment rolling update?
<details><summary>Answer</summary>

A rolling update gradually replaces old Pods with new ones.

Control availability and rollout speed with deployment strategy settings and health probes.

Always plan rollback and database compatibility for schema-changing releases.
</details>

### Q7. What is a StatefulSet?
<details><summary>Answer</summary>

StatefulSet is designed for stateful workloads requiring stable identity, ordering or persistent storage associations.

Many databases are better operated using managed database services unless there is a strong reason to run them in Kubernetes.
</details>

### Q8. What is an Ingress?
<details><summary>Answer</summary>

Ingress defines HTTP/HTTPS routing rules into services. An Ingress Controller implements those rules.

In modern Kubernetes ecosystems, Gateway API is also increasingly relevant.
</details>

### Q9. What happens when a node fails?
<details><summary>Answer</summary>

The control plane detects node health problems. Pods managed by controllers can be recreated elsewhere if capacity and scheduling constraints allow.

Persistent workloads require storage behavior and topology to be considered.
</details>

### Q10. Explain HPA.
<details><summary>Answer</summary>

Horizontal Pod Autoscaler adjusts replica count based on metrics such as CPU, memory or custom/external metrics.

Autoscaling should account for startup time, downstream capacity and stabilization to avoid oscillation.
</details>

### Q11. How would you debug CrashLoopBackOff?
<details><summary>Answer</summary>

Use:
- `kubectl describe pod`
- `kubectl logs`
- `kubectl logs --previous`
- pod events
- exit codes
- environment/configuration
- probes
- resource limits
- dependency connectivity

CrashLoopBackOff is a symptom, not the root cause.
</details>

### Q12. How do you perform zero-downtime deployment?
<details><summary>Answer</summary>

Use:
- multiple replicas
- readiness probes
- rolling updates
- graceful shutdown
- connection draining
- backward-compatible API/schema changes
- appropriate rollout thresholds

For risky changes, use canary or blue/green strategies.
</details>

### Q13. What is a NetworkPolicy?
<details><summary>Answer</summary>

NetworkPolicy restricts which Pods/namespaces can communicate according to supported cluster networking behavior.

Use it to reduce unnecessary east-west communication and limit blast radius.
</details>

### Q14. Kubernetes vs Docker?
<details><summary>Answer</summary>

Docker is primarily used to build/package/run containers.

Kubernetes orchestrates containers across a cluster, handling scheduling, service discovery, rollout, scaling and reconciliation.

They solve different layers of the problem.
</details>

## Quick Revision Checklist

Pod → Deployment → Service → Ingress → probes → requests/limits → ConfigMap/Secret → StatefulSet → HPA → rollout → node failure → NetworkPolicy → debugging.
