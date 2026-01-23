This round is testing core backend + system design fundamentals: rate limiting, large-scale APIs, and practical AWS usage.[1][2]

Below are plain-language explanations for each topic, plus a set of likely client-round questions you can practice.

***

## Token Bucket algorithm

Token Bucket is a simple way to control how many requests are allowed per second while still allowing short bursts.[3][1]

- Imagine a **bucket** that holds tokens, with a maximum capacity \(C\).[4][3]
- Tokens are added at a fixed rate \(R\) (e.g., 5 tokens per second) until the bucket is full.[3][4]
- Every incoming request must take 1 token from the bucket to be allowed. If a token is available, the request passes and 1 token is removed.[1][3]
- If the bucket is empty (no tokens), the request is either rejected or delayed until tokens refill.[4][3]

Key points you can say in an interview:

- It enforces an **average** rate (because tokens add steadily) but allows **bursts** if tokens have accumulated during idle time.[1][4]
- Two main parameters:  
  - Capacity \(C\): max burst size.  
  - Rate \(R\): average allowed requests per second.  
- It’s often implemented in memory (e.g., Redis) with:  
  - last_refill_timestamp  
  - current_tokens  

***

## Rate limiting (using Token Bucket)

Rate limiting means controlling how many requests a user/service can make in a time window so you protect your backend and enforce fair usage.[4][1]

Common strategies:

- Fixed window: Allow N requests per fixed time window, like 100 requests per minute. Simple but has edge cases at window boundaries.  
- Sliding window: Track requests over the last rolling interval (e.g., last 60 seconds) for smoother limiting.  
- **Token Bucket**:  
  - Store `tokens`, `last_refill_time`.  
  - On each request:  
    1) Calculate how many tokens to add since last refill: `tokens = min(capacity, tokens + rate * delta_time)`.  
    2) If `tokens > 0`, allow and decrement; else reject/429 or queue.[4]

Talking points they expect:

- Where to implement:  
  - At API gateway (per client/IP).  
  - At service level (per user, per API key, per tenant).  
- Storage choices:  
  - Single-node: in-memory map.  
  - Distributed: Redis (atomic INCR, Lua scripts) for consistency across instances.  
- Responses:  
  - Use HTTP 429 Too Many Requests, maybe with retry-after header.  

***

## Design a video streaming platform

Think “mini YouTube/Netflix”: focus on upload pipeline, storage, and delivery via CDN.[5][2]

### 1. Requirements (say these first)

- Functional:  
  - Users upload videos.  
  - Users watch videos with play/pause/seek.  
  - Basic metadata: title, description, thumbnail.  
- Non-functional:  
  - Highly **scalable** (millions of users).  
  - Low latency playback using CDN.  
  - Cost-efficient storage.  

### 2. High-level components

- API gateway / backend service: handles upload requests, metadata, auth.  
- Storage:  
  - Raw uploads and processed video files in object storage like S3.  
- Transcoding service:  
  - Converts uploaded video into multiple resolutions/bitrates (e.g., 240p, 480p, 720p).  
  - Generates HLS/DASH segments and playlists (e.g., `.m3u8`).[2]
- CDN:  
  - Caches video segments close to users for low latency.[5][2]
- Database:  
  - Stores video metadata, user info, watch history.  

### 3. Upload and processing flow

- User uploads video to backend or directly to storage through a pre-signed URL.  
- Backend enqueues a “transcode job” (message queue).  
- Worker pulls job, reads video from storage, transcodes to different formats, and writes processed outputs back to storage.[2]
- Once ready, backend updates video status to “READY” so the client can start streaming.  

### 4. Playback flow

- Client requests video; backend returns URL of HLS/DASH manifest (e.g., `.m3u8`).[2]
- Player fetches manifest from CDN/storage, then requests small chunks (segments) of the video.  
- Adaptive bitrate streaming: client switches quality based on network speed.  

If they ask for more detail, mention:

- Sharding videos by video ID prefix in storage,  
- Using a relational DB for metadata + maybe Elasticsearch for search,  
- Using caching (Redis) for popular video metadata.  

***

## Implement a JIRA-like service (codebase)

They likely want you to think about basic entities, REST APIs, and clean structure in Python.

### 1. Core entities

Simple version of JIRA:

- User  
- Project  
- Issue (ticket):  
  - id, title, description  
  - status (TO_DO, IN_PROGRESS, DONE)  
  - type (BUG, TASK, STORY)  
  - priority  
  - assignee (user)  
  - created_at, updated_at  

### 2. Basic API endpoints

You can describe something like:

- `POST /projects` – create a project.  
- `GET /projects/{id}` – get project details.  
- `POST /projects/{id}/issues` – create an issue in a project.  
- `GET /issues/{id}` – get issue details.  
- `PATCH /issues/{id}` – update fields: status, assignee, etc.  
- `GET /projects/{id}/issues?status=...&assignee=...` – list/filter issues.  

### 3. Python structure and TDD

- Use a web framework (FastAPI / Flask / Django REST):  
  - `models.py`: ORM models (Issue, Project, User).  
  - `schemas.py`: request/response DTOs (pydantic models in FastAPI).  
  - `routes.py` or `views.py`: endpoints using services.  
  - `services.py`: business logic (e.g., change status, validate transitions).  

- TDD approach:  
  - Write tests first:  
    - Create issue, change status, invalid status transitions, filtering.  
  - Tests hit HTTP endpoints (integration) or services (unit).  
  - Implement minimal code to pass tests and refactor.  

Mention:

- Use migrations for DB schema evolution.  
- Add auth later (e.g., JWT) and role checks if needed.  

***

## AWS S3 multipart upload

Multipart upload is how you efficiently upload large files to S3 by splitting them into parts.[6][7][8]

### What it is

- Instead of sending one huge file in a single request, you split it into smaller parts (e.g., 5–100 MB each).  
- Each part is uploaded independently and can be uploaded in parallel.[7][8][6]
- After all parts are uploaded, S3 combines them into one object. You then see a single file in the bucket.[8][6][7]

### Why it’s useful

- Better throughput: multiple parts upload in parallel, saturating bandwidth.[7][8]
- Resilience: if one part fails, you retry just that part instead of restarting the whole upload.[6][7]
- Mandatory for very large files (e.g., >5 GB recommended to use multipart).[8]

### How to use (high level flow)

For an interview, say these three steps:

- Initiate upload:  
  - Call `CreateMultipartUpload` and get an `uploadId`.[7][8]
- Upload parts:  
  - Split file into parts.  
  - For each part (identified by `partNumber`), call `UploadPart` with the `uploadId`.  
  - Keep track of ETags for each uploaded part.[6][8][7]
- Complete upload:  
  - Call `CompleteMultipartUpload` with the list of part numbers and ETags.  
  - S3 assembles parts and creates the final object.[8][7]

You can also mention:

- If you cancel, use `AbortMultipartUpload` to clean up partial parts.[8]

***

## Practice questions for your client round

Here are targeted questions you can practice. Try to answer them aloud in simple language and focus on clarity, trade-offs, and Python-specific details.

### Token Bucket & rate limiting

1. How would you implement a token bucket rate limiter in Python for a single-process web service?  
2. How would you adapt your rate limiter to work correctly across multiple instances behind a load balancer?  
3. How would you design rate limits per user, per API key, and per IP at the same time?  
4. What’s the difference between token bucket and leaky bucket, and when would you use each?  
5. How would you expose rate-limit information to clients (headers, error codes, retry behavior)?  

### System design: video streaming / large backend

1. Walk through the end-to-end flow when a user uploads a video until another user can play it.  
2. How would you handle a sudden spike in viewers for a single viral video?  
3. Where would you use a CDN and what kind of content would you cache?  
4. How would you design the database schema and indexes for videos, users, and views?  
5. How would you track and store watch history efficiently for millions of users?  

### JIRA-like service / Python backend

1. Design the DB schema for a simple JIRA-like system (tables + important columns + relationships).  
2. Explain how you would implement optimistic locking for issue updates (e.g., two people editing the same issue).  
3. Describe how you would structure a Python service layer and repository layer for issues.  
4. How would you design an audit trail for issues (who changed what and when)?  
5. How would you design search/filter for issues (by status, assignee, labels, full text)?  

### TDD, APIs, and system design thinking

1. In a Python FastAPI or Django project, how do you organize tests for models, services, and routes?  
2. Give an example of how TDD changed your design vs if you had coded directly.  
3. How do you ensure your APIs remain backward compatible as they evolve?  
4. How would you document your APIs for other developers (OpenAPI/Swagger, README, examples)?  
5. How would you design configuration and environment management for a service deployed to multiple environments (dev, staging, prod)?  

### AWS and scalability

1. When would you choose multipart upload vs a simple `put_object` in S3?  
2. How would you implement direct-to-S3 uploads from a frontend app using pre-signed URLs?  
3. How would you make your backend stateless, and where would you store shared state?  
4. How would you design logging, metrics, and tracing for your API so you can debug issues quickly?  
5. How would you design a background job system in Python (e.g., Celery/RQ) for tasks like transcoding or sending emails?  

***

If you want, paste one of your own answers (e.g., for “design a video streaming platform”) and the level of detail you expect in the client round, and that answer can be improved and sharpened to sound more confident and structured.

[1](https://www.geeksforgeeks.org/computer-networks/token-bucket-algorithm/)
[2](https://www.systemdesignhandbook.com/guides/design-youtube/)
[3](https://en.wikipedia.org/wiki/Token_bucket)
[4](https://dev.to/0xtanzim/token-bucket-algorithm-explained-4ceo)
[5](https://www.youtube.com/watch?v=NH-f955mvGg)
[6](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpu-upload-object.html)
[7](https://aws.amazon.com/blogs/compute/uploading-large-objects-to-amazon-s3-using-multipart-upload-and-transfer-acceleration/)
[8](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html)
[9](https://news.ycombinator.com/item?id=28037366)
[10](https://hnhiring.com/locations/remote/months/april-2022)
[11](https://nomadjobboard.com)
[12](https://www.lri.fr/~adecelle/content/teaching/m1info_pstat_info/tps/count_1w.txt)
[13](https://www.exploratory.io/viz/PQd2wHB2BZ/Public-Cloud-Records-viy6NIA6GV?embed=true)
[14](http://jbc.bj.uj.edu.pl/Content/955595/NDIGOC072514_2023.pdf)
[15](https://www.cs.princeton.edu/courses/archive/fall20/cos226/assignments/autocomplete/files/words-333333.txt)

