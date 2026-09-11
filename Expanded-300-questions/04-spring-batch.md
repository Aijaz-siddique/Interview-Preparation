# Spring Batch Interview Preparation — Experienced Level

Focus: chunking, transactions, restartability, partitioning, fault tolerance and production operations.

## Core Concepts

### Q1. What is Spring Batch?
<details><summary>Answer</summary>

A framework for reliable batch processing with jobs, steps, chunk processing, transactions, restartability and execution metadata.

</details>

### Q2. Job vs Step?
<details><summary>Answer</summary>

A Job represents the overall process; a Step is a phase within it. Jobs can contain multiple steps and conditional transitions.

</details>

### Q3. JobInstance vs JobExecution?
<details><summary>Answer</summary>

A JobInstance represents a logical run identified by job name and identifying parameters. A JobExecution represents an attempt to execute that instance and has status/runtime metadata.

</details>

### Q4. What is JobParameters?
<details><summary>Answer</summary>

Parameters identify/configure a job run. Identifying parameters influence which logical JobInstance is selected.

</details>

### Q5. What is ExecutionContext?
<details><summary>Answer</summary>

It stores state needed for restartability, such as reader position or custom checkpoint data.

</details>

## Chunk Processing

### Q6. Explain read-process-write.
<details><summary>Answer</summary>

A chunk step reads items, transforms them optionally, and writes a chunk within transaction boundaries. Commit interval controls transaction frequency.

</details>

### Q7. How does chunk size affect performance?
<details><summary>Answer</summary>

Larger chunks reduce commit overhead but increase transaction duration, memory and rollback cost. Smaller chunks reduce rollback scope but may increase overhead.

</details>

### Q8. ItemReader vs ItemProcessor vs ItemWriter?
<details><summary>Answer</summary>

Reader obtains input, processor transforms/validates it, writer persists or emits output. Keeping these roles separate improves testing and reuse.

</details>

### Q9. What happens when writer fails?
<details><summary>Answer</summary>

The transaction can roll back the current chunk. Retry/skip policies determine whether the item/chunk is retried, skipped or the step fails.

</details>

### Q10. What is a fault-tolerant step?
<details><summary>Answer</summary>

A step configured with skip/retry behavior can continue through selected failures while tracking limits and classifications.

</details>

## Scaling & Recovery

### Q11. How do you process 100 million records?
<details><summary>Answer</summary>

Use database-efficient readers, appropriate chunk size, partitioning/parallelism where safe, indexes, bounded concurrency and monitoring. Validate database and downstream capacity before scaling workers.

</details>

### Q12. What is partitioning?
<details><summary>Answer</summary>

Partitioning divides a step's input into independent partitions, each processed by a worker step. Partitions must be disjoint and collectively complete.

</details>

### Q13. Partitioning vs multi-threaded step?
<details><summary>Answer</summary>

Partitioning gives separate execution contexts/input ranges and can scale work horizontally. A multi-threaded step shares step infrastructure and requires thread-safe components.

</details>

### Q14. What is remote partitioning?
<details><summary>Answer</summary>

Partition definitions can be distributed to remote workers through messaging, allowing horizontal batch execution across processes/nodes.

</details>

### Q15. How do you make a batch restartable?
<details><summary>Answer</summary>

Persist checkpoint state, use deterministic/idempotent writes, ensure readers can resume safely and design around failures occurring between external side effects and checkpoint commits.

</details>

## Errors & Operations

### Q16. Skip vs retry?
<details><summary>Answer</summary>

Skip abandons a known-bad item after policy/limit checks. Retry repeats a potentially transient operation.

</details>

### Q17. How do you classify exceptions?
<details><summary>Answer</summary>

Classify based on recoverability and business semantics. Network timeouts may be retryable; malformed data may be skippable/quarantined.

</details>

### Q18. How should bad records be handled?
<details><summary>Answer</summary>

Quarantine or error-table them with enough context for reconciliation. Never silently discard data.

</details>

### Q19. How do you prevent duplicate batch execution?
<details><summary>Answer</summary>

Use scheduler/job repository semantics, distributed locking where necessary and unique business/job parameters. Never assume only one application instance exists.

</details>

### Q20. How do you monitor batch SLAs?
<details><summary>Answer</summary>

Track job/step status, duration, throughput, read/write counts, skips/retries, lag and resource usage. Alert on missed completion windows and abnormal rates.

</details>

### Q21. How do you design restart after partial external API calls?
<details><summary>Answer</summary>

Persist idempotency keys/status around the external call, make the call idempotent if possible, and reconcile ambiguous outcomes instead of assuming failure means no side effect.

</details>

## Advanced Design

### Q22. When should you use Spring Batch vs Kafka consumers?
<details><summary>Answer</summary>

Batch suits bounded/scheduled workloads; Kafka consumers suit continuous event streams. Requirements around latency, replay, ordering and throughput determine the choice.

</details>

### Q23. How do transactions interact with skip/retry?
<details><summary>Answer</summary>

Transaction boundaries determine what gets rolled back. Retry may repeat a transactional operation; skip requires carefully defining what state is committed after the failure.

</details>

### Q24. What is a job repository?
<details><summary>Answer</summary>

It stores job/step execution metadata used for status, restartability and operational tracking.

</details>

### Q25. How would you test a batch job?
<details><summary>Answer</summary>

Test reader/processor/writer independently, then integration-test job flows, transaction behavior, restart, skip/retry and representative large-data cases.

</details>

## Question Count

**25 experienced-level questions** in this file.

## Quick Revision Checklist

