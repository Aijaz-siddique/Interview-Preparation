# Docker Interview Preparation — Experienced Level

Focus: images, builds, runtime, networking, security, Compose, CI/CD and troubleshooting.

## Docker Fundamentals

### Q1. Image vs container?
<details><summary>Answer</summary>

An image is an immutable package/template. A container is a running instance with its own process/filesystem/network view.

</details>

### Q2. How do image layers work?
<details><summary>Answer</summary>

Each Dockerfile instruction can create a layer that may be cached and reused. Ordering stable steps before frequently changing steps improves build cache reuse.

</details>

### Q3. CMD vs ENTRYPOINT?
<details><summary>Answer</summary>

ENTRYPOINT defines the primary executable behavior; CMD supplies defaults that can be overridden. They can be combined.

</details>

### Q4. COPY vs ADD?
<details><summary>Answer</summary>

COPY is explicit file copying. ADD has extra behaviors such as archive extraction. Prefer COPY unless ADD's extra behavior is required.

</details>

### Q5. What is Docker build context?
<details><summary>Answer</summary>

The files sent to the build engine. A good .dockerignore keeps it small and prevents accidental inclusion of secrets/artifacts.

</details>

## Builds & Images

### Q6. Why use multi-stage builds?
<details><summary>Answer</summary>

Build dependencies remain in a builder stage while the runtime image contains only what is needed, reducing size and attack surface.

</details>

### Q7. How do you reduce image size?
<details><summary>Answer</summary>

Use multi-stage builds, minimal runtime images, .dockerignore, dependency cleanup and sensible layer ordering.

</details>

### Q8. How do you secure images?
<details><summary>Answer</summary>

Use trusted/minimal bases, scan vulnerabilities, avoid secrets, run non-root where practical and control dependency provenance.

</details>

### Q9. Why pin base images?
<details><summary>Answer</summary>

Predictability and reproducibility improve when image versions/digests are controlled, though a maintenance process must still update them for security.

</details>

### Q10. Why not put secrets in Dockerfile ARG/ENV?
<details><summary>Answer</summary>

They can leak through image metadata/layers or runtime inspection. Use secret injection mechanisms appropriate to the build/runtime platform.

</details>

## Runtime & Networking

### Q11. Why does a container exit?
<details><summary>Answer</summary>

The container lifecycle follows its main process. If that process exits, the container stops unless a restart policy/orchestrator acts.

</details>

### Q12. Docker volume?
<details><summary>Answer</summary>

Persistent storage independent of a container's writable layer, suitable for data that must survive container replacement.

</details>

### Q13. Docker bridge network?
<details><summary>Answer</summary>

Provides isolated container networking and DNS/service connectivity for containers attached to the network.

</details>

### Q14. Why log to stdout/stderr?
<details><summary>Answer</summary>

Container platforms can collect standard streams centrally. This avoids relying on ephemeral container-local log files.

</details>

### Q15. What is port mapping?
<details><summary>Answer</summary>

It maps a host port to a container port, making a container service reachable through the host interface.

</details>

## Compose & Production

### Q16. What is Docker Compose?
<details><summary>Answer</summary>

A declarative way to define and run multi-container applications, especially useful for local/integration environments.

</details>

### Q17. How should health checks be designed?
<details><summary>Answer</summary>

Check meaningful application readiness without expensive dependency cascades. Health checks should have realistic intervals/timeouts and distinguish readiness from liveness.

</details>

### Q18. Docker vs Kubernetes?
<details><summary>Answer</summary>

Docker provides container build/runtime tooling; Kubernetes orchestrates workloads across a cluster. They address different layers.

</details>

### Q19. What is rootless Docker?
<details><summary>Answer</summary>

A mode where Docker components/containers run without requiring root privileges, reducing certain privilege risks.

</details>

### Q20. How do you troubleshoot a container?
<details><summary>Answer</summary>

Inspect logs, exit code, process/entrypoint, environment, mounts, permissions, network connectivity and resource limits.

</details>

## CI/CD & Supply Chain

### Q21. What should a container CI pipeline do?
<details><summary>Answer</summary>

Build, test, scan dependencies/image, produce a versioned artifact, optionally sign/provenance it, then deploy through controlled environments.

</details>

### Q22. Why avoid latest tag in production?
<details><summary>Answer</summary>

It is mutable and makes rollback/reproducibility ambiguous. Use immutable version tags/digests.

</details>

### Q23. What is image promotion?
<details><summary>Answer</summary>

Build once and promote the same immutable artifact across environments rather than rebuilding different binaries per environment.

</details>

### Q24. How do you handle vulnerability findings?
<details><summary>Answer</summary>

Classify severity/exploitability, prioritize actionable fixes, update base/dependencies and document accepted risk when immediate remediation is impossible.

</details>

## Question Count

**24 experienced-level questions** in this file.

## Quick Revision Checklist

