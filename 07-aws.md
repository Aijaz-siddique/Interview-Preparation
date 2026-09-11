# AWS Interview Preparation

### Q1. Explain an AWS production architecture for a scalable REST API.
<details><summary>Answer</summary>

A common architecture is:

Client → CDN/WAF → Load Balancer/API layer → autoscaling application instances → cache/database → async queue/workers.

Use multiple Availability Zones, managed services where appropriate, IAM least privilege, monitoring and backups.

The exact services depend on requirements rather than following a fixed template.
</details>

### Q2. EC2 vs ECS vs EKS vs Lambda?
<details><summary>Answer</summary>

EC2 gives VM-level control.

ECS is AWS's managed container orchestration option.

EKS provides managed Kubernetes control-plane infrastructure.

Lambda is event-driven serverless execution.

Choose based on operational control, workload shape, team expertise, startup behavior, networking and cost.
</details>

### Q3. S3 storage classes and lifecycle?
<details><summary>Answer</summary>

S3 provides object storage with different cost/access characteristics.

Lifecycle policies can transition objects to cheaper storage or expire them based on age.

Consider retrieval frequency, latency, retention and compliance requirements.
</details>

### Q4. RDS vs DynamoDB?
<details><summary>Answer</summary>

RDS is relational and provides SQL, transactions and relational modeling.

DynamoDB is a distributed NoSQL database optimized around access patterns and key-based operations.

Choose based on query patterns, consistency, transactions, scaling and data relationships.
</details>

### Q5. What is IAM?
<details><summary>Answer</summary>

IAM controls authentication and authorization through identities, roles and policies.

Production best practice is least privilege and short-lived credentials/roles where possible rather than embedding long-lived access keys.
</details>

### Q6. What is the difference between Security Groups and Network ACLs?
<details><summary>Answer</summary>

Security Groups are stateful controls associated with network interfaces/resources.

Network ACLs operate at subnet level and are stateless.

Security groups are generally the primary fine-grained workload access control mechanism.
</details>

### Q7. SQS vs SNS?
<details><summary>Answer</summary>

SQS is a queue used to decouple producers and consumers.

SNS is a pub/sub notification service that can fan out messages to multiple subscribers.

They are often combined: SNS publishes an event to multiple SQS queues.
</details>

### Q8. What is an Availability Zone vs Region?
<details><summary>Answer</summary>

A Region is a geographic AWS area. Availability Zones are isolated infrastructure locations within a Region.

Deploying across multiple AZs improves resilience to an AZ-level failure.
</details>

### Q9. How would you secure an AWS workload?
<details><summary>Answer</summary>

Use:
- least-privilege IAM
- private subnets where appropriate
- security groups
- encryption at rest/in transit
- secrets management
- WAF
- logging/auditing
- patching
- network segmentation
- backup and recovery controls

Security should be layered rather than dependent on one control.
</details>

### Q10. How do you reduce AWS cost?
<details><summary>Answer</summary>

Measure first.

Common levers:
- right-size compute
- autoscale
- use appropriate storage classes
- remove idle resources
- reserved/savings options for stable workloads
- optimize data transfer
- improve caching
- lifecycle old data
- monitor cost allocation

Cost optimization must preserve required reliability/performance.
</details>

### Q11. What is CloudWatch used for?
<details><summary>Answer</summary>

CloudWatch provides metrics, logs, alarms and operational visibility.

Use dashboards and alerts around user-facing SLOs as well as infrastructure metrics.
</details>

### Q12. How do you design disaster recovery?
<details><summary>Answer</summary>

Define RTO and RPO first.

Then select a strategy such as backup/restore, pilot light, warm standby or multi-site active/active.

Validate recovery through drills. A backup that has never been restored is not a proven recovery strategy.
</details>

## Quick Revision Checklist

IAM → VPC → AZ/Region → EC2 → ECS/EKS/Lambda → S3 → RDS/DynamoDB → SQS/SNS → CloudWatch → security → DR → cost.
