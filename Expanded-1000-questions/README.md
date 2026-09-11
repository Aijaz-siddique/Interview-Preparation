# 🚀 Master Software Engineering & System Design Interview Preparation Repo

Welcome to the **Ultimate Production & Product MNC (FAANG/MAANG, Tier-1 FinTech, Unicorns) Interview Preparation Repository**. 

This single repository is structured to provide an exhaustive, battle-tested, deep-dive revision kit for Senior Software Engineers, Lead Engineers, Solutions Architects, and Tech Leads.

---

## 📚 Table of Contents & Modules

| # | Topic | File Link | Focus Areas & Topics Covered |
|---|---|---|---|
| 01 | **Java (Core & Advanced)** | [`01-Java.md`](./01-Java.md) | JVM Architecture, Memory Model (G1/ZGC), Concurrency, Collections Internals, Virtual Threads (Project Loom), Reflection, Bytecode |
| 02 | **Spring Framework** | [`02-Spring.md`](./02-Spring.md) | IoC Container, Bean Lifecycle, Circular Dependencies, AOP, Transaction Management, Proxying, Spring WebFlux, Security Context |
| 03 | **Spring Boot** | [`03-SpringBoot.md`](./03-SpringBoot.md) | Auto-Configuration, Custom Starters, Actuator, Embedded Containers, Configuration Properties, Metrics, Graceful Shutdown |
| 04 | **Spring Batch** | [`04-SpringBatch.md`](./04-SpringBatch.md) | Chunk vs Tasklet, Job Repository, Chunk Processing, Skip/Retry Policies, Partitioning, Remote Chunking, Parallel Steps |
| 05 | **Microservices Architecture** | [`05-Microservices.md`](./05-Microservices.md) | Saga Pattern, CQRS, Event-Driven Architecture, Service Discovery, API Gateways, Resiliency (Circuit Breaker, Bulkhead), Distributed Tracing |
| 06 | **Apache Kafka** | [`06-Kafka.md`](./06-Kafka.md) | Log Architecture, Partitioning, Consumer Groups, ISR, Rebalance Protocols, Exactly-Once Semantics (EOS), Kafka Streams, Schema Registry |
| 07 | **Redis & In-Memory Databases** | [`07-Redis.md`](./07-Redis.md) | Data Structures, Persistence (RDB/AOF), Sentinel, Cluster Sharding, Eviction Policies, Distributed Locks (Redlock), Caching Topologies |
| 08 | **Design Patterns & Refactoring** | [`08-DesignPatterns.md`](./08-DesignPatterns.md) | Creational, Structural, Behavioral Patterns, Anti-Patterns, Enterprise Integration Patterns, Real-World Code Refactoring Scenarios |
| 09 | **High Level Design (HLD)** | [`09-HighLevelDesign_HLD.md`](./09-HighLevelDesign_HLD.md) | Scalability, Load Balancing, Database Sharding, Consistency Models, CAP/PACELC, Distributed Caching, Message Queues, Edge Computing |
| 10 | **Low Level Design (LLD)** | [`10-LowLevelDesign_LLD.md`](./10-LowLevelDesign_LLD.md) | Object-Oriented Design, SOLID Principles, Class Diagrams, Schema Design, Thread Safety, Extensible Interfaces, Concurrency Patterns |
| 11 | **Distributed Systems** | [`11-DistributedSystems.md`](./11-DistributedSystems.md) | Consensus (Raft/Paxos), Vector Clocks, Consistent Hashing, 2PC/3PC, Distributed Locks, Rate Limiting Algorithms, Idempotency |
| 12 | **React & Modern Frontend** | [`12-React.md`](./12-React.md) | Fiber Reconciler, Virtual DOM, Hooks Internals, State Management, SSR/SSG/ISR, Performance Optimization, Code Splitting, Web Vitals |
| 13 | **AWS (Amazon Web Services)** | [`13-AWS.md`](./13-AWS.md) | IAM, EC2, S3, RDS/DynamoDB, Lambda, ECS/EKS, VPC Architecture, CloudFront, Route53, KMS, Serverless Design |
| 14 | **Google Cloud Platform (GCP)** | [`14-GCP.md`](./14-GCP.md) | IAM, Compute Engine, GKE, Cloud Spanner, BigQuery, Cloud Pub/Sub, Cloud Run, VPC Service Controls, Firestore |
| 15 | **Kubernetes (K8s)** | [`15-Kubernetes.md`](./15-Kubernetes.md) | Control Plane, Pod Lifecycle, Ingress/Egress, CNI, CSI, CRDs, StatefulSets, HPA/VPA, Service Mesh (Istio), Cluster Security |
| 16 | **Docker & Containerization** | [`16-Docker.md`](./16-Docker.md) | Container Isolation (cgroups/namespaces), Storage Drivers, Multi-stage Builds, Security Hardening, Networking Modes, Daemon vs Rootless |
| 17 | **CI/CD & DevOps Automation** | [`17-CICD.md`](./17-CICD.md) | GitOps (ArgoCD), Blue-Green/Canary Deployments, Pipeline Optimization, Infrastructure as Code (Terraform), Secrets Management |
| 18 | **Application & Network Security** | [`18-Security.md`](./18-Security.md) | OAuth2.0 / OIDC, JWT Hardening, Mutual TLS (mTLS), OWASP Top 10 Mitigation, Cryptography (AES/RSA/HMAC), Zero Trust Architecture |
| 19 | **Git & Version Control** | [`19-Git.md`](./19-Git.md) | Git Internals (Objects/DAG), Rebase vs Merge, Bisect, Cherry-pick, Conflict Resolution, Gitflow vs Trunk-based, Reflog, Hooks |
| 20 | **Behavioral & Leadership (STAR Method)** | [`20-Behavioral_Leadership.md`](./20-Behavioral_Leadership.md) | Amazon Leadership Principles, Conflict Resolution, Technical Debt Trade-offs, Mentorship, Post-Mortems, System Outage Leadership |

---

## 🎯 How to Use This Repo for Last-Minute Revision

1. **Expander Format (`<details>`):** Every question has a collapsed answer. First try answering out loud or on paper, then click the expander to verify against senior-level standards.
2. **Tier-1 MNC Focus:** Questions emphasize **internal working, trade-offs, edge-cases, failure modes, and architectural implications** rather than basic syntax.
3. **Structured Roadmaps:**
   - **1-Week Emergency Revision:** Focus on HLD, LLD, Microservices, Distributed Systems, and your core language (Java/React).
   - **1-Month In-Depth Preparation:** Go through all 20 modules sequentially, completing at least 1 module per 1-2 days.

---

## 💡 Key Product MNC Evaluation Criteria
- **Depth of Knowledge:** Understanding *why* something works (e.g., how ConcurrentHashMap handles rehash locks vs synchronized HashMap).
- **Trade-off Analysis:** Articulating pros and cons (e.g., Kafka vs RabbitMQ, SQL Sharding vs NoSQL, Optimistic vs Pessimistic Locking).
- **Production Safety:** Knowing resilience patterns (Circuit Breakers, Retries with Exponential Backoff + Jitter, Rate Limiting, Graceful Degradation).
- **Communication:** Using clear structure (STAR framework for behavioral, System Design templates for HLD).

---
*Created for engineers preparing for Tier-1 Product MNCs (Amazon, Google, Microsoft, Meta, Uber, Netflix, Atlassian, Salesforce, etc.)*