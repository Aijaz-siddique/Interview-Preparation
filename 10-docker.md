# Docker Interview Preparation

### Q1. Image vs container?
<details><summary>Answer</summary>

An image is an immutable package/template containing application files and metadata.

A container is a running instance of an image with its own process/network/filesystem view.
</details>

### Q2. Explain Docker layers.
<details><summary>Answer</summary>

Docker images are built from layers. Layers can be cached and reused.

Good Dockerfiles order stable dependencies before frequently changing source code to maximize cache reuse and speed builds.
</details>

### Q3. CMD vs ENTRYPOINT?
<details><summary>Answer</summary>

`ENTRYPOINT` defines the main executable behavior.

`CMD` provides default command/arguments and can be overridden.

They can be combined so the image has a stable executable with configurable defaults.
</details>

### Q4. COPY vs ADD?
<details><summary>Answer</summary>

`COPY` is simpler and explicit for copying files.

`ADD` has additional behavior such as archive extraction and URL-related semantics.

Prefer `COPY` unless you specifically need ADD's extra behavior.
</details>

### Q5. Why use multi-stage builds?
<details><summary>Answer</summary>

Use one stage to compile/build and another minimal stage to run the application.

Benefits:
- smaller images
- reduced attack surface
- fewer build tools in production
- faster deployment/pull times
</details>

### Q6. How do containers differ from VMs?
<details><summary>Answer</summary>

VMs virtualize hardware and typically contain a full guest OS.

Containers share the host kernel while isolating processes/filesystem/network namespaces.

Containers generally have lower overhead, but isolation and operational characteristics differ from VMs.
</details>

### Q7. How do you reduce Docker image size?
<details><summary>Answer</summary>

Use:
- multi-stage builds
- minimal runtime images
- `.dockerignore`
- dependency cleanup
- no build tools in runtime image
- layer/cache optimization

Don't optimize size at the expense of security/debuggability without reason.
</details>

### Q8. What is Docker Compose?
<details><summary>Answer</summary>

Compose defines multi-container applications declaratively, including services, networks, volumes and configuration.

It is particularly useful for local development and integration environments.
</details>

### Q9. How do Docker volumes work?
<details><summary>Answer</summary>

Volumes persist data independently of a container's writable layer.

Use them for state that must survive container replacement. In orchestrated production environments, use the platform's appropriate persistent-storage mechanism.
</details>

### Q10. How do you troubleshoot a container that exits immediately?
<details><summary>Answer</summary>

Check:
- container logs
- exit code
- command/entrypoint
- environment variables
- mounted files
- permissions
- application startup error

A container exits when its main process exits; "container stopped" is usually a symptom of application/process behavior.
</details>

### Q11. How do you make Docker builds secure?
<details><summary>Answer</summary>

Use:
- trusted/minimal base images
- pinned/controlled dependencies where appropriate
- vulnerability scanning
- non-root runtime user
- no secrets in Dockerfile/image
- minimal packages
- signed/provenance-aware supply chain where required
</details>

### Q12. Why should applications log to stdout/stderr in containers?
<details><summary>Answer</summary>

Container platforms can collect stdout/stderr centrally.

Writing logs only to container-local files makes collection and lifecycle management harder unless an explicit logging architecture exists.
</details>

### Q13. What is a Docker network?
<details><summary>Answer</summary>

Docker networks provide isolated communication between containers.

Compose commonly creates a network where services can reach each other using service names as DNS names.
</details>

### Q14. What is the purpose of `.dockerignore`?
<details><summary>Answer</summary>

It prevents unnecessary files from being sent as build context.

This reduces build time, image leakage risk and context size. Exclude secrets, Git metadata, build outputs and local dependencies where appropriate.
</details>

## Quick Revision Checklist

Image/container → layers/cache → Dockerfile → CMD/ENTRYPOINT → multi-stage → image size → Compose → volumes → networking → security → troubleshooting → logging.
