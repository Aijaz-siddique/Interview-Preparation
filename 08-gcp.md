# GCP Interview Preparation

### Q1. Explain the GCP resource hierarchy.
<details><summary>Answer</summary>

A simplified hierarchy is Organization → Folders → Projects → Resources.

IAM policies can be inherited through the hierarchy. Projects provide an important boundary for APIs, billing association, quotas and resource organization.
</details>

### Q2. GCE vs GKE vs Cloud Run?
<details><summary>Answer</summary>

GCE provides VMs.

GKE provides managed Kubernetes.

Cloud Run provides managed container execution with serverless operational characteristics.

Choose based on workload control, orchestration needs, scaling and operational burden.
</details>

### Q3. Explain GCP service accounts.
<details><summary>Answer</summary>

Service accounts represent workload identities.

Prefer attaching service identities/roles to workloads rather than storing service-account key files.

Use least privilege and short-lived credentials where possible.
</details>

### Q4. Cloud Storage vs database?
<details><summary>Answer</summary>

Cloud Storage is object storage suited for files, blobs, backups and data lakes.

A database should be used when you need structured querying, transactional semantics or low-latency record operations.

Often both are used together.
</details>

### Q5. Explain GKE at a high level.
<details><summary>Answer</summary>

GKE is managed Kubernetes on Google Cloud.

Important concepts include:
- cluster
- node pools
- pods
- deployments
- services
- ingress/load balancing
- autoscaling
- workload identity
- observability

Know which responsibilities Google manages and which remain with the application/team.
</details>

### Q6. Pub/Sub vs Kafka?
<details><summary>Answer</summary>

Both support event-driven architectures, but their operational models differ.

Pub/Sub is a managed GCP messaging service with deep GCP integration.

Kafka provides a log-based streaming model with strong control over partitions, offsets and replay semantics.

Choose based on event model, ordering, replay, ecosystem and operational requirements.
</details>

### Q7. Explain BigQuery.
<details><summary>Answer</summary>

BigQuery is a serverless analytical data warehouse optimized for large-scale SQL analytics.

It is not normally a replacement for a low-latency transactional OLTP database.
</details>

### Q8. How do you secure GCP workloads?
<details><summary>Answer</summary>

Use:
- IAM least privilege
- service accounts/workload identity
- private networking
- Secret Manager
- encryption
- audit logging
- organization policies
- segmentation
- vulnerability management

Avoid long-lived credentials where possible.
</details>

### Q9. How would you troubleshoot a GKE pod that keeps restarting?
<details><summary>Answer</summary>

Check:
1. pod events
2. container logs
3. previous container logs
4. exit code
5. liveness/readiness probes
6. resource limits/OOMKilled
7. configuration/secrets
8. image/version
9. dependency connectivity

Determine whether the application is crashing or Kubernetes is killing it.
</details>

### Q10. What is autoscaling in GKE?
<details><summary>Answer</summary>

Horizontal Pod Autoscaler adjusts pod replicas based on metrics.

Cluster Autoscaler adjusts nodes based on scheduling demand.

Both must be configured with realistic resource requests and limits; otherwise scaling decisions can be misleading.
</details>

### Q11. What is Cloud Monitoring/Logging?
<details><summary>Answer</summary>

Google Cloud Observability services provide metrics, logs, traces and alerting.

Build dashboards around service-level health, latency, errors and saturation rather than only CPU/memory.
</details>

### Q12. How would you design a highly available GCP service?
<details><summary>Answer</summary>

Use managed load balancing, multiple zones where appropriate, autoscaling, resilient data services, health checks, graceful degradation and tested backups/DR.

The architecture should be driven by RTO/RPO and availability requirements.
</details>

## Quick Revision Checklist

Resource hierarchy → IAM → service accounts → GCE → GKE → Cloud Run → Cloud Storage → Pub/Sub → BigQuery → Secret Manager → networking → observability → DR.
