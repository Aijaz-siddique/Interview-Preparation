# Spring Batch Interview Preparation

### Q1. What is Spring Batch?
<details><summary>Answer</summary>

Spring Batch is a framework for reliable batch processing. It provides concepts such as jobs, steps, readers, processors, writers, chunk processing, transactions, restartability and job metadata.
</details>

### Q2. Job vs Step?
<details><summary>Answer</summary>

A Job represents the overall batch process. A Step is an independently configured phase of that job.

A job can contain multiple steps with conditional transitions.
</details>

### Q3. Explain chunk processing.
<details><summary>Answer</summary>

A chunk-oriented step reads items, optionally processes them, and writes them in chunks within transaction boundaries.

For example, a chunk size of 100 may read/process 100 records and commit the writer transaction.

Chunk size affects memory, transaction duration, throughput and restart behavior.
</details>

### Q4. What happens when a chunk fails?
<details><summary>Answer</summary>

The current transaction can roll back. Depending on configuration, retry/skip behavior can determine whether processing continues.

A production design must distinguish retryable transient errors from permanent bad data.
</details>

### Q5. Skip vs retry?
<details><summary>Answer</summary>

**Skip** means an item can be abandoned after a configured failure policy.

**Retry** means the same operation is attempted again because the failure may be transient.

Examples:
- malformed input → possibly skip/quarantine
- temporary network failure → retry
</details>

### Q6. What is restartability?
<details><summary>Answer</summary>

A failed job should be able to resume from a meaningful point rather than reprocessing everything blindly.

Spring Batch stores execution metadata and step state. Correct restartability also depends on reader/writer behavior and idempotency.
</details>

### Q7. How would you process millions of records?
<details><summary>Answer</summary>

Consider:
- paging/cursor readers
- appropriate chunk size
- indexes
- partitioning
- parallel steps
- remote partitioning if necessary
- controlled thread pools
- database connection capacity
- backpressure
- restartability
- monitoring

Don't increase parallelism beyond downstream capacity.
</details>

### Q8. What is partitioning?
<details><summary>Answer</summary>

Partitioning divides a workload into independent partitions, allowing multiple workers to process different ranges.

Examples include ID ranges, date ranges or file partitions.

The partition strategy should avoid overlap and ensure every item is covered exactly as intended.
</details>

### Q9. How do you make batch jobs idempotent?
<details><summary>Answer</summary>

Design operations so retries/restarts do not corrupt state.

Common techniques:
- unique business keys
- upsert
- processed markers
- staging tables
- transactional boundaries
- deterministic transformations
- idempotent external API calls

Assume failures can happen after the external side effect but before the job records success.
</details>

### Q10. How would you monitor a production batch?
<details><summary>Answer</summary>

Track:
- job status
- duration
- records read/processed/written
- skip/retry counts
- failure causes
- throughput
- lag
- resource usage
- restart count

Alert on SLA violations and abnormal record/error rates.
</details>

### Q11. How do you handle bad records?
<details><summary>Answer</summary>

Do not silently discard them.

Depending on requirements:
- skip with limits
- quarantine/DLQ
- error table
- audit record
- generate reconciliation report

The business should be able to identify what failed and why.
</details>

### Q12. Batch vs streaming?
<details><summary>Answer</summary>

Batch processes bounded workloads, often on schedules. Streaming continuously processes events.

Choose based on business latency requirements, volume, ordering, replay and operational complexity.
</details>

## Quick Revision Checklist

Job/Step → chunk → transaction → reader/processor/writer → skip/retry → restartability → partitioning → scaling → idempotency → bad data → monitoring.
