# GCP Interview Preparation — Experienced Level

Focus: GCP architecture, GKE, Cloud Run, data, messaging, IAM, networking, observability and DR.

## GCP Core

### Q1. Explain GCP resource hierarchy.
<details><summary>Answer</summary>

Organization → folders → projects → resources. IAM can inherit through the hierarchy. Projects also organize APIs, quotas and billing association.

</details>

### Q2. GCE vs GKE vs Cloud Run?
<details><summary>Answer</summary>

GCE provides VMs, GKE provides managed Kubernetes, and Cloud Run provides managed container execution with serverless operational characteristics.

</details>

### Q3. What is a project?
<details><summary>Answer</summary>

A project is a core administrative boundary for resources, APIs, IAM and billing association.

</details>

### Q4. Region vs zone?
<details><summary>Answer</summary>

A region is a geographic area containing zones. Multi-zone deployment improves resilience against zone-level failures.

</details>

## Compute & Kubernetes

### Q5. What is GKE?
<details><summary>Answer</summary>

Managed Kubernetes on Google Cloud. Google manages the control plane in supported modes while workloads, configuration and application behavior remain customer responsibilities.

</details>

### Q6. What is GKE Autopilot?
<details><summary>Answer</summary>

A managed GKE operating mode where Google takes more responsibility for node management and resource optimization, reducing operational overhead.

</details>

### Q7. What is Cloud Run?
<details><summary>Answer</summary>

A managed platform for running containers with automatic scaling and a request/event-driven operational model.

</details>

### Q8. When choose GCE over GKE?
<details><summary>Answer</summary>

Choose GCE when VM-level control or legacy software requires it. Choose GKE when Kubernetes orchestration, portability and workload scheduling justify its complexity.

</details>

## Data & Messaging

### Q9. Cloud Storage vs database?
<details><summary>Answer</summary>

Cloud Storage is object storage; databases provide structured query/transaction capabilities. They often complement each other.

</details>

### Q10. Cloud SQL vs Spanner?
<details><summary>Answer</summary>

Cloud SQL provides managed relational databases with familiar engines. Spanner targets globally scalable relational workloads with stronger distributed characteristics and higher architectural complexity/cost.

</details>

### Q11. BigQuery use case?
<details><summary>Answer</summary>

Large-scale analytical SQL and data warehousing, not a general-purpose low-latency OLTP replacement.

</details>

### Q12. Pub/Sub vs Kafka?
<details><summary>Answer</summary>

Pub/Sub is deeply managed/integrated with GCP. Kafka provides a log-oriented streaming model with explicit partition/offset control. Choose based on replay, ordering, ecosystem and operational needs.

</details>

## IAM & Security

### Q13. What is a service account?
<details><summary>Answer</summary>

A workload identity used by applications/services to access Google Cloud resources.

</details>

### Q14. How should service accounts be secured?
<details><summary>Answer</summary>

Use least privilege, workload identity/attached identities and short-lived credentials where possible. Avoid long-lived key files.

</details>

### Q15. What is Secret Manager?
<details><summary>Answer</summary>

A managed service for storing and controlling access to secrets, with integration into workloads and IAM.

</details>

### Q16. How do you secure GCP networking?
<details><summary>Answer</summary>

Use private networking, firewall rules, identity-aware controls, segmentation, private service access/endpoints where appropriate and centralized policy.

</details>

## Observability & Reliability

### Q17. Cloud Monitoring vs Cloud Logging?
<details><summary>Answer</summary>

Monitoring provides metrics, dashboards and alerts; Logging centralizes log data and analysis. Together they support operational diagnosis.

</details>

### Q18. How do you debug a restarting GKE pod?
<details><summary>Answer</summary>

Inspect pod events, current/previous logs, exit codes, probes, resource limits, configuration and dependency connectivity.

</details>

### Q19. How do you design GCP DR?
<details><summary>Answer</summary>

Define RTO/RPO, choose regional/multi-regional capabilities where appropriate, replicate data, automate recovery and test restoration.

</details>

### Q20. How do you optimize GCP cost?
<details><summary>Answer</summary>

Right-size resources, autoscale, select appropriate storage/compute, remove idle resources and monitor project/service cost allocation.

</details>

## Question Count

**20 experienced-level questions** in this file.

## Quick Revision Checklist

