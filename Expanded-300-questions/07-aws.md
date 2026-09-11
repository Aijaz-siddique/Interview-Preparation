# AWS Interview Preparation — Experienced Level

Focus: architecture, compute, networking, security, databases, messaging, observability, DR and cost.

## AWS Core Architecture

### Q1. Region vs Availability Zone?
<details><summary>Answer</summary>

A Region is a geographic AWS location containing multiple Availability Zones. AZ separation supports resilience against localized infrastructure failures.

</details>

### Q2. Design a highly available REST API on AWS.
<details><summary>Answer</summary>

A common pattern is CDN/WAF → load balancer/API layer → multi-AZ compute → cache/database, with asynchronous queues for non-critical work and centralized observability.

</details>

### Q3. EC2 vs ECS vs EKS vs Lambda?
<details><summary>Answer</summary>

EC2 provides VM control; ECS orchestrates containers with AWS-native simplicity; EKS provides Kubernetes; Lambda provides serverless event-driven execution. Choose based on operational control and workload shape.

</details>

### Q4. What is Auto Scaling?
<details><summary>Answer</summary>

It adjusts compute capacity based on demand or schedules, improving availability and cost efficiency.

</details>

### Q5. What is VPC?
<details><summary>Answer</summary>

A logically isolated virtual network where you define subnets, routing, gateways and network controls.

</details>

## Storage & Databases

### Q6. S3 use cases?
<details><summary>Answer</summary>

Object storage for files, backups, static assets, data lakes and archival workflows. Design lifecycle, encryption and access controls.

</details>

### Q7. S3 vs EBS vs EFS?
<details><summary>Answer</summary>

S3 is object storage; EBS provides block storage for EC2; EFS provides shared file storage accessible by multiple compute instances.

</details>

### Q8. RDS vs DynamoDB?
<details><summary>Answer</summary>

RDS provides relational SQL semantics and transactions. DynamoDB is managed distributed NoSQL optimized around key/access patterns and horizontal scale.

</details>

### Q9. Read replicas vs Multi-AZ?
<details><summary>Answer</summary>

Read replicas primarily scale reads and can support certain recovery patterns. Multi-AZ focuses on high availability/failover for supported database configurations.

</details>

### Q10. DynamoDB partition key design?
<details><summary>Answer</summary>

Choose keys that distribute traffic evenly and support required access patterns. Avoid hot partitions caused by highly concentrated keys.

</details>

## Networking & Security

### Q11. Security Group vs NACL?
<details><summary>Answer</summary>

Security Groups are stateful controls associated with network interfaces. NACLs are stateless subnet-level controls.

</details>

### Q12. Public vs private subnet?
<details><summary>Answer</summary>

A public subnet has a route toward an internet gateway; private workloads generally use NAT or private endpoints for outbound access.

</details>

### Q13. What is IAM least privilege?
<details><summary>Answer</summary>

Grant only permissions required for the workload and scope resources/actions as narrowly as practical.

</details>

### Q14. Role vs access key?
<details><summary>Answer</summary>

Roles provide temporary credentials and are preferred for workloads. Long-lived access keys increase credential leakage risk.

</details>

### Q15. How do you secure secrets?
<details><summary>Answer</summary>

Use managed secret/configuration services, IAM controls, rotation and encryption. Never bake credentials into images or source repositories.

</details>

### Q16. What does WAF protect?
<details><summary>Answer</summary>

A web application firewall filters HTTP traffic against configured application-layer threats and abusive patterns.

</details>

## Messaging & Serverless

### Q17. SQS vs SNS?
<details><summary>Answer</summary>

SQS provides queues and consumer decoupling; SNS provides pub/sub fan-out. SNS can deliver to multiple SQS queues.

</details>

### Q18. Standard vs FIFO SQS?
<details><summary>Answer</summary>

Standard prioritizes scale and at-least-once delivery with best-effort ordering. FIFO provides stronger ordering/deduplication semantics within its documented constraints.

</details>

### Q19. When should Lambda be used?
<details><summary>Answer</summary>

Event-driven, bursty or short-lived workloads can fit well. Consider execution limits, startup behavior, concurrency and downstream connection limits.

</details>

### Q20. What is event-driven architecture?
<details><summary>Answer</summary>

Components communicate through events rather than synchronous calls, improving decoupling and buffering at the cost of eventual consistency and operational complexity.

</details>

## Operations & Cost

### Q21. What should CloudWatch monitor?
<details><summary>Answer</summary>

Metrics, logs, alarms and dashboards for service health, latency, errors, saturation and infrastructure behavior.

</details>

### Q22. How do you reduce AWS cost?
<details><summary>Answer</summary>

Right-size, autoscale, remove idle resources, choose appropriate storage classes, optimize transfer and use commitment discounts for predictable workloads.

</details>

### Q23. How do you design DR?
<details><summary>Answer</summary>

Start with RTO/RPO, then choose backup/restore, pilot light, warm standby or multi-site based on business requirements and budget.

</details>

### Q24. What is a multi-account strategy?
<details><summary>Answer</summary>

Separate environments/security boundaries into accounts and govern them centrally. Exact structure depends on organization and compliance needs.

</details>

### Q25. How do you investigate an AWS outage?
<details><summary>Answer</summary>

Correlate service health, application metrics, deployment changes, networking, dependencies, quotas and logs. Use blast-radius reduction and rollback where appropriate.

</details>

## Question Count

**25 experienced-level questions** in this file.

## Quick Revision Checklist

