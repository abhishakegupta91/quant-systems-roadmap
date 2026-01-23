# Interview Practice Questions

## Overview

This document contains all the practice questions from your interview preparation materials, organized by topic for focused study.

## Token Bucket & Rate Limiting

1. **How would you implement a token bucket rate limiter in Python for a single-process web service?**
   - Create TokenBucket class with capacity, refill_rate, tokens, and last_refill_time attributes.
   - On each request, calculate elapsed time, add tokens proportionally, then check if tokens >= 1.
   - Complexity: O(1) per request, uses time.time() for refill calculations.

2. **How would you adapt your rate limiter to work correctly across multiple instances behind a load balancer?**
   - Use Redis for shared state storage with atomic operations to prevent race conditions.
   - Implement Lua script for atomic token updates (get-calculate-set) across distributed instances.
   - Store rate limit data in Redis with keys like "rate_limit:{user_id}" and TTL for cleanup.

3. **How would you design rate limits per user, per API key, and per IP at the same time?**
   - Implement separate token buckets for each dimension using different Redis key patterns.
   - Check all three limits sequentially - if any limit exceeded, reject request immediately.
   - Use Redis keys: `rate_limit:user:{id}`, `rate_limit:api_key:{key}`, `rate_limit:ip:{addr}`.

4. **What's the difference between token bucket and leaky bucket, and when would you use each?**
   - Token bucket allows bursts up to capacity while enforcing average rate over time.
   - Leaky bucket processes requests at fixed rate, smoothing traffic but preventing bursts.
   - Use token bucket for APIs with variable traffic patterns, leaky bucket for steady streams.

5. **How would you expose rate-limit information to clients (headers, error codes, retry behavior)?**
   - Return HTTP 429 Too Many Requests with Retry-After header indicating wait time.
   - Include X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset headers in responses.
   - Provide clear error message with rate limit details and retry guidance.

6. **What happens if a user sends a burst of requests when the bucket is at capacity?**
   - Token bucket allows burst up to capacity (e.g., 100 tokens accumulated during idle time).
   - After burst, user must wait for tokens to refill at the configured rate.
   - Example: Capacity 100, rate 10/sec - can send 100 requests instantly, then limited to 10/sec.

7. **How would you handle a user with unlimited tokens? Prevent abuse?**
   - Implement hard caps on maximum requests per second regardless of token accumulation.
   - Use circuit breaker pattern to temporarily block consistently abusive users.
   - Monitor patterns and implement progressive penalties for repeated abuse.

8. **Compare token bucket, leaky bucket, sliding window, and fixed window approaches.**
   - Token bucket: Burst-friendly, enforces average rate, good for variable traffic.
   - Leaky bucket: Smooth output rate, no bursts, ideal for steady processing.
   - Sliding window: Accurate rate limiting over rolling time window, higher complexity.
   - Fixed window: Simple implementation, but allows bursts at window boundaries.

## System Design: Video Streaming

1. **Walk through end-to-end flow when user uploads a video to when another user can play it.**
   - User uploads to pre-signed S3 URL, backend enqueues transcode job to message queue.
   - Worker service transcodes to multiple resolutions (240p, 480p, 720p, 1080p) using FFmpeg.
   - Generates HLS segments (~10 second chunks) and manifest (.m3u8) file, uploads to S3.
   - Updates database status to READY, viewer requests manifest URL and streams segments from CDN.

2. **How would you handle a viral video with sudden spike in viewers?**
   - CDN caches video segments at edge locations near users globally for low latency.
   - Database uses read replicas to handle increased metadata query load without affecting writes.
   - S3 automatically scales to serve massive parallel segment requests without manual intervention.
   - API gateway load balances across multiple backend instances for authentication and metadata.

3. **Design the database schema for videos, users, and watch history. Discuss indexes.**
   - videos table: id, user_id, title, description, status, s3_key, duration, created_at.
   - Index on user_id for user video queries, status for processing queue operations.
   - watch_history table: user_id, video_id, watched_until, watched_at with composite index (user_id, watched_at).
   - Separate video_stats table for view counts to avoid expensive COUNT(*) operations.
   - watch_history: user_id, video_id, watched_until.
   - Composite index on (user_id, watched_at) for fast "recent videos" queries.

4. **How would you implement "adaptive bitrate streaming"?**
   - Transcode video into multiple bitrates (240p @ 200kbps, 480p @ 800kbps, 720p @ 2.5mbps, 1080p @ 5mbps).
   - Create separate segment files for each bitrate, generate master playlist listing all variants.
   - Video player monitors network download speed and switches between quality levels mid-stream.
   - Ensures smooth playback regardless of network conditions by automatically adjusting quality.

5. **How to prevent video piracy / unauthorized downloads?**
   - Generate time-limited pre-signed URLs for video segments that expire after short duration.
   - Require authentication tokens for each segment request, validate against user permissions.
   - Encrypt video segments using AES-128 keys, deliver decryption keys only to authenticated users.
   - Implement DRM (Digital Rights Management) for premium content using Widevine or FairPlay.

6. **How would you design a "recommendation" system recommending videos to users?**
   - Collect user watch history, likes, watch duration, and search patterns for behavioral data.
   - Implement collaborative filtering: find users with similar viewing habits and recommend their watched videos.
   - Use content-based filtering: analyze video metadata (tags, categories, descriptions) for similar content.
   - Train machine learning model on historical data to predict user preferences and rank recommendations.

7. **How to handle simultaneous uploads from millions of users?**
   - Use direct-to-S3 uploads with pre-signed URLs to bypass backend servers and reduce load.
   - Implement S3 multipart upload for large files, allowing parallel upload of file chunks.
   - Distribute uploads across multiple S3 buckets in different regions to reduce per-region load.
   - Use message queue (SQS/Kafka) to process upload completion events asynchronously.

8. **Where would you use a CDN and what kind of content would you cache?**
   - Cache video segments (HLS chunks) at CDN edge locations for low latency streaming.
   - Cache video thumbnails, metadata, and playlist files to reduce backend load.
   - Cache popular content permanently, less popular content with shorter TTL based on access patterns.
   - Don't cache user-specific content, admin endpoints, or real-time data.

9. **How would you track and store watch history efficiently for millions of users?**
   - Use dedicated watch_history table with composite index on (user_id, watched_at) for fast queries.
   - Batch watch history updates using bulk inserts to reduce database load and improve performance.
   - Consider time-series database (InfluxDB) for analytics, relational DB for user-facing features.
   - Archive old history (> 6 months) to cold storage (S3 Glacier) to reduce storage costs.

10. **How would you design the upload pipeline to handle large files (GBs) reliably?**
    - Implement S3 multipart upload with configurable part size (5-100MB) for optimal performance.
    - Track upload progress in database, support pause/resume functionality for interrupted uploads.
    - Use checksums (MD5/SHA256) to verify file integrity during and after upload completion.
    - Implement retry logic with exponential backoff for failed parts, automatic cleanup on abandoned uploads.

## JIRA-like Service & TDD

1. **Design the DB schema for a JIRA-like system.**
   - Users table (id, name, email), Projects table (id, name, description, created_by), Issues table (id, project_id, title, description, status, assignee_id, version).
   - Comments table (issue_id, author_id, content, created_at), Audit table (issue_id, field_name, old_value, new_value, changed_by).
   - Composite indexes on (project_id, status) for filtering, (assignee_id, status) for user dashboards.

2. **How would you implement optimistic locking to prevent lost updates?**
   - Add version column to issues table, increment on each successful update.
   - Before updating, check WHERE version = expected_version; if 0 rows affected, return 409 Conflict.
   - Client must read current version, include in update request, and handle conflict by retrying.

3. **Describe the TDD approach: write tests first, then code.**
   - Write failing test for specific functionality (e.g., test_create_issue()).
   - Run test to confirm it fails (red), implement minimal code to pass test (green).
   - Refactor code for clarity while maintaining test coverage (refactor), repeat for each feature.

4. **How to design API for filtering issues (status, assignee, priority)?**
   - GET /projects/{id}/issues?status=IN_PROGRESS&assignee=user123&priority=high&limit=20&offset=0.
   - Build dynamic SQL query based on provided filters, use parameterized queries for security.
   - Implement pagination with limit/offset, return total count for client-side pagination controls.

5. **Implement an audit trail tracking who changed what and when.**
   - Create issue_audit table with columns: issue_id, changed_by, field_name, old_value, new_value, changed_at.
   - On each issue update, compare old vs new values, insert audit records for changed fields.
   - Use triggers or service layer to automatically capture changes, provide audit API for compliance.

6. **How to implement full-text search for issue descriptions?**
   - Add FULLTEXT index on issue.title and issue.description columns for native search.
   - Alternatively, integrate Elasticsearch for advanced search with relevance scoring and faceting.
   - Support phrase matching, boolean operators, and search result highlighting.

7. **How would you handle "issue dependencies" (Issue A blocks Issue B)?**
   - Create issue_dependencies table with blocking_issue_id and blocked_issue_id columns.
   - Implement business logic to prevent closing blocked issues until blockers are resolved.
   - Add notifications when blocker status changes, visualize dependency graph in UI.

8. **Explain how you would structure a Python service layer and repository layer for issues.**
   - Repository layer: IssueRepository class with methods like find_by_id(), create(), update(), find_by_project().
   - Service layer: IssueService class with business logic like create_issue(), update_status(), assign_issue().
   - Controller layer: FastAPI endpoints that call service methods, handle HTTP requests/responses.

9. **How would you implement search/filter for issues (by status, assignee, labels, full text)?**
   - Use database queries for structured filters (status, assignee, priority) with proper indexes.
   - Implement Elasticsearch integration for full-text search across title and description.
   - Combine both approaches: filter in database first, then full-text search on filtered results.

10. **How would you design configuration and environment management for a service deployed to multiple environments?**
    - Use environment variables for sensitive data (DB credentials, API keys), config files per environment.
    - Implement configuration classes with validation, use dependency injection for testability.
    - Separate dev/staging/prod configurations, use secrets manager for production secrets.
   - Increment version on successful update.

3. **Describe the TDD approach: write tests first, then code.**
   - Write test: test_create_issue().
   - Test should fail initially (no implementation).
   - Write minimal code to pass test.
   - Refactor code for clarity.
   - Repeat for each feature.

4. **How to design API for filtering issues (status, assignee, priority)?**
   - GET /projects/{id}/issues?status=IN_PROGRESS&assignee=user123.
   - Build query incrementally based on filters.
   - Use indexes to optimize.
   - Support pagination (skip, limit).

5. **Implement an audit trail tracking who changed what and when.**
   - On every update, insert row in issue_audit.
   - Record field_name, old_value, new_value, changed_by, changed_at.
   - Use for compliance, debugging, "undo" feature.

6. **How to implement full-text search for issue descriptions?**
   - Add FULLTEXT index on issue.description.
   - Use MySQL MATCH AGAINST(...) or Elasticsearch.
   - Support phrase search, Boolean operators.

7. **How would you handle "issue dependencies" (Issue A blocks Issue B)?**
   - Add issue_dependencies table: (blocking_issue_id, blocked_issue_id).
   - When blocking issue moves to DONE, notify blocked issue assignee.
   - Prevent closing blocked issue if blocker not done.

8. **Explain how you would structure a Python service layer and repository layer for issues.**
   - Repository: Database operations (CRUD).
   - Service: Business logic, validation, orchestration.
   - Controller: HTTP endpoints, request/response handling.
   - Models: Data structures, validation.

9. **How would you implement search/filter for issues (by status, assignee, labels, full text)?**
   - Database queries for structured filters (status, assignee).
   - Elasticsearch for full-text search.
   - Combine both for comprehensive search.
   - Cache frequent queries.

10. **How would you design configuration and environment management for a service deployed to multiple environments?**
    - Environment variables for config.
    - Separate config files per environment.
    - Use dependency injection for config.
    - Secrets management for sensitive data.

## AWS S3 & Large File Uploads

1. **Explain multipart upload and why it's useful.**
   - Split large file into parts (e.g., 5MB each).
   - Upload parts in parallel.
   - S3 assembles into single object.
   - Faster upload, resilient to failures.

2. **Implement a Python script to multipart upload a 1GB file.**
   - Initiate upload (CreateMultipartUpload).
   - Split file into parts.
   - Upload parts in parallel (ThreadPoolExecutor).
   - Collect ETags.
   - Complete upload (CompleteMultipartUpload).

3. **How would you implement "pause and resume" for uploads?**
   - Store uploadId in client.
   - Track which parts uploaded (in memory or DB).
   - On resume, upload remaining parts.
   - Specify part-specific range headers if resuming after network failure.

4. **What happens if one part fails during upload?**
   - Retry that specific part (not entire file).
   - Exponential backoff.
   - After max retries, abort upload (AbortMultipartUpload).

5. **How to securely allow direct client uploads to S3?**
   - Backend generates pre-signed URL (time-limited, limited scope).
   - Client uploads directly to pre-signed URL.
   - Backend never sees file content.
   - Pre-signed URL expires after 15 minutes.

6. **Design a flow to upload video directly to S3 and transcode.**
   - User gets pre-signed upload URL from backend.
   - User uploads directly to S3.
   - S3 triggers event (SNS/SQS).
   - Lambda or worker picks up event, transcodes.
   - Stores result back to S3.
   - Backend polls or uses webhook when ready.

7. **When would you choose multipart upload vs a simple `put_object` in S3?**
   - Multipart: Files > 100MB, need parallel uploads, want resume capability.
   - Simple: Small files (< 100MB), single request is sufficient.

8. **How would you implement direct-to-S3 uploads from a frontend app using pre-signed URLs?**
   - Backend generates pre-signed URLs for each part.
   - Frontend uploads parts directly to S3.
   - Frontend sends ETags to backend.
   - Backend completes multipart upload.

9. **What are the size limits for S3 multipart upload?**
   - Part size: 5MB to 5GB per part.
   - Maximum parts: 10,000 parts.
   - Maximum object size: 5TB.
   - Recommended: Use multipart for files > 100MB.

10. **How would you handle upload progress tracking for large files?**
    - Track completed parts vs total parts.
    - Calculate percentage complete.
    - Estimate remaining time based on upload speed.
    - Provide real-time feedback to user.

## Distributed Systems: Consensus & Replication

1. **Explain CAP theorem. Give examples for each combination (CP, AP).**
   - **CP (Consistency + Partition Tolerance):** Bank account. Block transactions if can't sync replicas.
   - **AP (Availability + Partition Tolerance):** Social media. Always respond, eventual consistency.
   - **Trade-off:** Can't have all three. Network partitions happen; choose C or A.

2. **How would you design a distributed key-value store?**
   - **Partitioning:** Consistent hashing. Key hashes to node.
   - **Replication:** Store key on node + 2 replicas for fault tolerance.
   - **Consistency:** Quorum read/write. Write to leader + replicas; require majority ACK.
   - **Leader election:** Use Raft or Paxos.

3. **Explain master-slave replication. What's a downside?**
   - Master accepts writes; slaves replicate; slaves serve reads.
   - Downside: Slave lag. Read from slave might return stale data.
   - Mitigation: Read from master after write, or wait for replica to catch up.

4. **How to handle leader failure in master-slave setup?**
   - Detect failure (heartbeat timeout).
   - Promote highest-priority slave to master.
   - Notify clients of new master.
   - Synced slaves catch up; unsynced slaves discarded.
   - Use Raft or external arbiter (e.g., Zookeeper) to prevent split-brain.

5. **Explain Raft consensus algorithm at high level.**
   - Time divided into terms (like election rounds).
   - Nodes vote for leader candidate.
   - Leader elected if gets majority vote.
   - Leader sends heartbeats to keep authority.
   - On leader failure, new election starts.

6. **What's "split-brain"? How to prevent?**
   - **Problem:** Network partition. Both halves think they're real cluster. Both accept writes.
   - **Solution:** Require quorum (majority). Minority partition stops serving requests.
   - Example: 5-node cluster splits 3-2. Only 3-node partition continues.

7. **Design a distributed counter (like view count).**
   - **Naïve:** Single DB row. Bottleneck.
   - **Solution:** Use local counters on each server. Periodically flush to DB.
   - **Accuracy tradeoff:** Eventually consistent (exact count after flush).
   - **Scale:** Billions of increments/second across world.

8. **How to ensure "read-after-write" consistency in distributed system?**
   - After write, always read from master (or wait for replica).
   - Or: store write timestamp; read from replica only if replica's lamport clock >= timestamp.
   - Trade-off: Slower reads, but always see your writes.

## Distributed Systems: Problems & Solutions

1. **How to prevent cascading failures in microservices?**
   - **Circuit breaker:** Stop calling failing service; fail fast.
   - **Bulkheads:** Isolate failures (separate thread pools per service).
   - **Load shedding:** Reject low-priority requests during overload.
   - **Retry with backoff:** Exponential backoff (1s, 2s, 4s, 8s...).
   - **Monitoring/alerting:** Detect anomalies early.

2. **How to handle distributed transactions (spanning multiple DBs)?**
   - **2PC (Two-Phase Commit):** Phase 1: prepare. Phase 2: commit/rollback. Blocks on failures.
   - **Saga Pattern:** Break into steps. If step fails, compensating transactions undo prior steps.
   - **Event sourcing:** Log all events. Rebuild state by replaying. Naturally supports rollback.

3. **How to handle network latency in distributed system?**
   - **Async:** Don't wait for response; use callbacks/futures.
   - **Caching:** Cache frequently accessed data locally.
   - **Batching:** Group multiple requests into one.
   - **Compression:** Reduce payload size.
   - **CDN:** Serve from location near user.

4. **How to implement "exactly-once" semantics (no duplicates)?**
   - **Idempotent keys:** Client sends unique ID per request.
   - **Deduplication cache:** Store (request_id, response). On retry, return cached response.
   - **Idempotent operations:** Design operations so repeated execution = same effect (PUT, DELETE).

5. **How would you implement distributed tracing for debugging?**
   - Assign correlation ID to each request.
   - Pass ID through all services.
   - Log with correlation ID.
   - Use distributed tracing tool (Jaeger, Zipkin) to visualize request flow.

## Algorithms: Questions

1. **Implement binary search. What's time complexity?**
   - O(log n). Eliminate half of remaining elements each iteration.
   - Code: two pointers (left, right), check mid element.

2. **Implement quicksort and explain how it works.**
   - Pick pivot, partition into smaller/equal/larger.
   - Recursively sort left and right.
   - O(n log n) average, O(n²) worst case.

3. **Difference between DFS and BFS. When to use each?**
   - **DFS:** Deep, use stack or recursion. Finds any path, good for backtracking.
   - **BFS:** Level-by-level, use queue. Finds shortest path in unweighted graph.

4. **Implement LRU cache. How does it work?**
   - Use ordered dict or doubly linked list + hash map.
   - On get/put, move to end (most recently used).
   - On insert when full, remove from front (least recently used).
   - O(1) per operation.

5. **Explain dynamic programming. Give an example.**
   - Break problem into overlapping subproblems.
   - Solve each once, store result.
   - Example: Fibonacci with memoization. O(n) instead of O(2^n).

6. **How does consistent hashing work? Why use for distributed systems?**
   - Map keys and servers to positions on a ring.
   - Key goes to nearest server (clockwise).
   - If server dies, only keys between adjacent servers rehash.
   - Minimal disruption compared to hash(key) % num_servers.

7. **Implement Dijkstra's algorithm for shortest path.**
   - Use priority queue (min heap).
   - Start from source, greedily pick closest unvisited node.
   - O((V + E) log V).

8. **What's the Longest Increasing Subsequence (LIS)? How to solve?**
   - Find longest subsequence where elements are in increasing order.
   - DP[i] = LIS length ending at arr[i].
   - dp[i] = max(dp[j] + 1 for all j < i where arr[j] < arr[i]).
   - O(n²) or O(n log n) with binary search.

## System Design: Open-Ended

1. **Design a real-time chat application.**
   - **Messages:** Queue (RabbitMQ/Kafka) for async delivery.
   - **Ordering:** Single queue per conversation guarantees order.
   - **Live notification:** WebSocket or Server-Sent Events (SSE).
   - **Persistence:** Store messages in DB, also write to cache for fast retrieval.
   - **Scaling:** Shard by conversation ID.

2. **Design a rate limiting service used by multiple teams.**
   - Centralized Redis instance(s).
   - Lua scripts for atomic updates.
   - Support multiple strategies: token bucket, sliding window.
   - Support multiple dimensions: per user, per API key, per IP.
   - Dashboard to monitor and adjust limits.

3. **Design a notification system (email, SMS, push).**
   - Queue notifications.
   - Workers consume, send via respective service (SendGrid, Twilio, Firebase).
   - Retry logic, dead letter queue.
   - Deduplication (don't send same notification twice).
   - Unsubscribe/preference management.

4. **Design a search system for billions of documents.**
   - Crawl/ingest documents.
   - Index in Elasticsearch (inverted index).
   - Rank results (relevance, freshness, popularity).
   - Cache frequent queries.
   - Shard index across multiple Elasticsearch nodes.

5. **Design a recommendation system.**
   - Collect user behavior (views, clicks, purchases).
   - Collaborative filtering: if users A and B have similar behavior, recommend B's items to A.
   - Content-based: if user likes "comedy", recommend other comedy.
   - ML model trained on historical data.

## TDD, APIs, and System Design Thinking

1. **In a Python FastAPI or Django project, how do you organize tests for models, services, and routes?**
   - Unit tests for models (test business logic).
   - Service tests (test use cases).
   - Integration tests for routes (test HTTP endpoints).
   - Use pytest fixtures for setup/teardown.

2. **Give an example of how TDD changed your design vs if you had coded directly.**
   - TDD forces you to think about interfaces first.
   - Leads to simpler, more testable code.
   - Example: Started with complex class, TDD led to smaller, focused functions.

3. **How do you ensure your APIs remain backward compatible as they evolve?**
   - Version APIs (v1, v2).
   - Add new fields, don't remove existing ones.
   - Use feature flags for new functionality.
   - Maintain old versions for transition period.

4. **How would you document your APIs for other developers (OpenAPI/Swagger, README, examples)?**
   - Auto-generate OpenAPI/Swagger from FastAPI.
   - Include examples in documentation.
   - Provide Postman collection.
   - Write clear README with getting started guide.

5. **How would you design configuration and environment management for a service deployed to multiple environments (dev, staging, prod)?**
   - Environment variables for sensitive data.
   - Config files per environment.
   - Use dependency injection for config.
   - Secrets management (AWS Secrets Manager, HashiCorp Vault).

## AWS and Scalability

1. **When would you choose multipart upload vs a simple `put_object` in S3?**
   - Multipart: Files > 100MB, need parallel uploads, want resume capability.
   - Simple: Small files (< 100MB), single request is sufficient.

2. **How would you implement direct-to-S3 uploads from a frontend app using pre-signed URLs?**
   - Backend generates pre-signed URLs for each part.
   - Frontend uploads parts directly to S3.
   - Frontend sends ETags to backend.
   - Backend completes multipart upload.

3. **How would you make your backend stateless, and where would you store shared state?**
   - Store session data in Redis.
   - Use JWT for authentication.
   - Store application state in database.
   - Use message queues for async processing.

4. **How would you design logging, metrics, and tracing for your API so you can debug issues quickly?**
   - Structured logging (JSON format).
   - Centralized logging (ELK stack).
   - Metrics collection (Prometheus + Grafana).
   - Distributed tracing (Jaeger, Zipkin).

5. **How would you design a background job system in Python (e.g., Celery/RQ) for tasks like transcoding or sending emails?**
   - Message broker (RabbitMQ, Redis).
   - Worker processes to consume tasks.
   - Task queues with priorities.
   - Retry logic and dead letter queues.
   - Monitoring and alerting for failed tasks.

## Study Tips

### How to Practice

1. **Start with the basics:** Ensure you understand fundamental concepts before moving to complex questions.
2. **Practice out loud:** Explain concepts as if you're in an interview.
3. **Focus on clarity:** Interviewers want to see your thought process, not memorized answers.
4. **Use real-world examples:** Reference actual systems (Netflix, YouTube, AWS) when appropriate.
5. **Consider trade-offs:** Always discuss pros and cons of your design decisions.

### Expected Answer Depth

**For first round (first screening):**
- Basic understanding, code examples, complexity analysis.

**For client round (final round):**
- Deep dive: architecture, trade-offs, deployment considerations.
- Be ready for follow-up questions: "What if load increases 10x?"
- Mention real systems: "Like how Netflix handles concurrent viewers."
- Discuss failure modes: "If this service goes down, what happens?"

### Key Areas to Master

- **System design:** Scalability, reliability, performance trade-offs
- **Distributed systems:** CAP theorem, consensus, replication
- **Algorithms:** Time/space complexity, common patterns
- **Practical implementation:** AWS, databases, caching
- **Communication:** Clear explanation of complex concepts

## Final Tips

- **Be confident but humble:** It's okay to say "I'm not sure, but here's how I'd approach it."
- **Show your thinking:** Talk through your reasoning step by step.
- **Ask clarifying questions:** Make sure you understand the requirements before diving in.
- **Practice regularly:** Consistent practice builds confidence and improves performance.

Good luck with your interviews!
