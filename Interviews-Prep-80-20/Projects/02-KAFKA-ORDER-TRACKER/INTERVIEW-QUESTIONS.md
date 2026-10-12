# Kafka Order Tracker — 80/20 Interview Questions

**Resume framing:** Python + Apache Kafka + Docker event-driven order tracker with producer/consumer services and streaming order-count/revenue analytics.

**Source inspected:** `producer.py`, `order_consumer.py`, `analytics_consumer.py`, `docker-compose.yml`, `README.md`.

## P0 — Must answer confidently

### 1. Explain the project in 60 seconds.
**Answer cue:** “A Python producer accepts order fields and publishes JSON events to Kafka topic `orders`. Kafka stores the events in a topic with three partitions. One consumer displays order events; a second consumer group processes the same stream to update in-memory order count and revenue. Docker Compose runs a local Kafka broker/controller.” Clarify this is a learning/demo system, not a durable production order database.

### 2. Why Kafka instead of calling the order consumer directly?
**Answer cue:** Kafka decouples producer and consumers. Producer can publish without knowing every downstream service; consumers can process independently and at their own pace. It supports replay and partitioned throughput. The trade-off is broker operations, eventual processing, duplicate handling and more moving parts.

### 3. Explain topic, partition, offset, broker and consumer group.
**Answer cue:** A topic is a named stream; partitions split it into ordered logs; each record has an offset within a partition; a broker stores/serves partition data; consumers read records; a consumer group coordinates consumers so partitions are assigned among group members. Ordering is guaranteed within a partition, not globally across all partitions.

### 4. Why does the producer use `key=region`?
**Answer cue:** The key influences partition selection, so events with the same region are typically routed to the same partition under the default partitioner, preserving per-key order. It can also create skew if one region dominates. The code serializes the key as UTF-8.

### 5. Why are there two consumers?
**Answer cue:** `order_consumer.py` is an operational/event-display consumer. `analytics_consumer.py` has `group_id="analytics"` and maintains analytics counters. Since they are separate groups, each can receive the topic's events independently. This is pub/sub, not two consumers splitting one shared group.

### 6. How are messages serialized?
**Answer cue:** Producer's `value_serializer` converts a Python dict to JSON text and UTF-8 bytes. Consumers decode the bytes and parse JSON with `json.loads`. Schema/version validation and malformed-message handling are not clearly implemented and would be useful improvements.

### 7. Is the analytics result durable and exactly-once?
**Answer cue:** No evidence supports that. `orders` and `revenue` are in-memory counters; they reset when the consumer restarts. The code does not establish exactly-once processing. The analytics consumer has no explicit idempotency or durable aggregate store.

### 8. What does `enable_auto_commit=False` mean in `order_consumer.py`?
**Answer cue:** Automatic offset commits are disabled. However, the code shown does not call `consumer.commit()`, so offsets may not be committed by the application. Explain that explicit commit timing should follow successful processing; otherwise restarts can replay records.

### 9. What happens if the consumer crashes after processing a record but before committing?
**Answer cue:** Depending on the last committed offset, the record can be processed again. This is at-least-once-style behavior unless the system adds idempotency/transactional coordination. Consumers should safely handle duplicates.

### 10. What is the current deployment topology?
**Answer cue:** `docker-compose.yml` runs one Apache Kafka container with broker and controller roles, exposing port 9092 and advertising `localhost:9092`. Topic config in README: three partitions, replication factor one. It is a local development topology, not fault tolerant; replication factor one provides no replica failover.

## P1 — Strong follow-up questions

11. What does `producer.flush()` do?
**Answer cue:** Waits for buffered producer records to be sent/completed before continuing. Calling it after every message simplifies demo feedback but can reduce throughput; batching/asynchronous sends may be better at scale.

12. What if `amount = int(input(...))` receives text or a negative amount?
**Answer cue:** Conversion can raise `ValueError`; negative values pass unless business validation rejects them. Validate fields, ranges, currency representation and required values before publishing.

13. Why is `auto_offset_reset="earliest"` used?
**Answer cue:** It determines where a group without a valid committed offset starts: earliest available offset rather than latest. It does not override an existing committed offset.

14. Why does `analytics_consumer.py` have a group ID but `order_consumer.py` does not?
**Answer cue:** Analytics group has a stable identity for offset tracking. Without a group ID, behavior and offset management differ; for robust service behavior, explicitly configure group IDs and commit strategy. Verify client defaults for the installed `kafka-python` version.

15. What does three partitions achieve?
**Answer cue:** Potential parallelism and throughput across consumer instances in the same group. One consumer group can have at most one active consumer per partition. More partitions increase parallelism but add overhead and do not automatically improve performance.

16. Why is replication factor 1 a limitation?
**Answer cue:** If the only replica/broker data is lost, data may be unavailable/lost. Production setup usually needs multiple brokers and replication, with careful ISR and acknowledgment settings.

17. How would you make revenue analytics survive restarts?
**Answer cue:** Persist aggregates in a database or compacted topic, rebuild from the retained event stream, or use a stream-processing engine/state store. Define event identity and duplicate handling before incrementing counters.

18. What if the same order event is published twice?
**Answer cue:** The current counter would count it twice. Add a unique event/order ID and idempotent aggregation, or persist processed IDs/unique constraints. Clarify whether updates and lifecycle events should count as new orders or transitions.

19. What is consumer lag?
**Answer cue:** Difference between latest available offsets and a consumer group's committed/current processing offsets. It indicates the consumer is behind, but interpretation is partition-specific.

20. How would you handle malformed messages or poison events?
**Answer cue:** Validate schema, catch decode/business errors, log metadata safely, and choose bounded retries or a dead-letter topic. Do not retry permanent invalid data forever. Dead-letter processing is a proposal, not shown as implemented.

21. What is backpressure in a streaming system?
**Answer cue:** Producers can outpace consumers; lag and memory/processing queues grow. Monitor lag, scale consumers within partition limits, tune batch processing, and decide how producers react to broker pressure.

22. What is the role of Docker Compose here?
**Answer cue:** Declaratively configures and starts the local Kafka service with ports and environment settings. It improves reproducibility but is not equivalent to production orchestration or a multi-broker cluster.

23. How would you test the project?
**Answer cue:** Unit-test serialization and validation; integration-test producing and consuming a known event; test consumer restart/offset behavior; test duplicate and malformed records; verify aggregate outputs from a deterministic sequence. The current repo doesn't show an automated test suite.

24. What if Kafka is unavailable when producer sends?
**Answer cue:** Send may fail or time out depending on client configuration. Add explicit error handling, timeouts, retry/backoff, delivery callbacks and clear user feedback. Do not print “Order Sent” as confirmed unless delivery success is confirmed.

25. What is the difference between at-most-once, at-least-once and exactly-once?
**Answer cue:** At-most-once can lose records but avoids reprocessing; at-least-once can replay duplicates; exactly-once requires coordinated transactional/idempotent processing within defined boundaries. Do not claim exactly-once for this project.

## P2 — Code-level cross-questions

26. What does the serializer lambda receive and return?
27. What happens if `data["amount"]` is missing or a string?
28. Why might using region as key lead to uneven partition distribution?
29. Can a consumer group read from every partition simultaneously?
30. What changes if both consumers use the same `group_id`?
31. Why is the analytics consumer's state lost on restart?
32. What does the broker's advertised listener mean, and why can `localhost` break for clients in other containers?
33. Why does `docker-compose.yml` define controller and broker roles in one node?
34. How would you process `Created`, `Paid`, `Shipped` and `Cancelled` events without double-counting revenue?
35. Which producer settings affect durability and throughput (`acks`, retries, batching, idempotence)?
36. How do you choose partition count and key?
37. What is a consumer rebalance and when might it happen?
38. Why should event schemas be versioned?
39. How would you monitor throughput, error rate, consumer lag and broker health?
40. What is the first design change you would make before calling this production-ready?

## Final readiness test

Explain questions 2–10 without notes. Then run producer and both consumers in separate terminals, publish a few events, stop/restart analytics consumer, and observe what resets/replays. Do not claim persistence or delivery guarantees that the experiment does not demonstrate.
