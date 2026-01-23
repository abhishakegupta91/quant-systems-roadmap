# Token Bucket Algorithm & Rate Limiting

## Overview

Token Bucket is a simple way to control how many requests are allowed per second while still allowing short bursts.

## Token Bucket Algorithm

### How it works

- Imagine a **bucket** that holds tokens, with a maximum capacity \(C\).
- Tokens are added at a fixed rate \(R\) (e.g., 5 tokens per second) until the bucket is full.
- Every incoming request must take 1 token from the bucket to be allowed. If a token is available, the request passes and 1 token is removed.
- If the bucket is empty (no tokens), the request is either rejected or delayed until tokens refill.

### Key points for interviews

- It enforces an **average** rate (because tokens add steadily) but allows **bursts** if tokens have accumulated during idle time.
- Two main parameters:  
  - Capacity \(C\): max burst size.  
  - Rate \(R\): average allowed requests per second.  
- It's often implemented in memory (e.g., Redis) with:  
  - last_refill_timestamp  
  - current_tokens  

### Python Implementation

```python
import time

class TokenBucket:
    def __init__(self, capacity, refill_rate):
        self.capacity = capacity  # max tokens in bucket
        self.refill_rate = refill_rate  # tokens per second
        self.tokens = capacity  # start with full bucket
        self.last_refill_time = time.time()
    
    def allow_request(self):
        # Calculate how many tokens to add since last refill
        now = time.time()
        elapsed = now - self.last_refill_time
        self.tokens = min(
            self.capacity,
            self.tokens + elapsed * self.refill_rate
        )
        self.last_refill_time = now
        
        # Check if we have tokens
        if self.tokens >= 1:
            self.tokens -= 1
            return True
        return False

# Usage
limiter = TokenBucket(capacity=100, refill_rate=5)  # 5 requests/sec
if limiter.allow_request():
    print("Request allowed!")
else:
    print("Rate limit exceeded. Try again later.")
```

## Rate Limiting

### What is it?

Rate limiting means controlling how many requests a user/service can make in a time window so you protect your backend and enforce fair usage.

### Common strategies

#### Fixed Window
- Allow N requests per fixed time window, like 100 requests per minute. Simple but has edge cases at window boundaries.

#### Sliding Window
- Track requests over the last rolling interval (e.g., last 60 seconds) for smoother limiting.

#### Token Bucket (recommended)
- Store `tokens`, `last_refill_time`.  
- On each request:  
  1) Calculate how many tokens to add since last refill: `tokens = min(capacity, tokens + rate * delta_time)`.  
  2) If `tokens > 0`, allow and decrement; else reject/429 or queue.

### Where to implement

- **API Gateway:** Per client IP or API key.
- **Service level:** Per user or per tenant.
- **Storage:** In-memory (single machine) or Redis (distributed).

### Response patterns

When rate limit is exceeded:
- **HTTP Status:** Return `429 Too Many Requests`.
- **Headers:** Include `Retry-After` header to tell client when to retry.
- **Message:** Include reset time in response body or headers.

```python
response_headers = {
    'Retry-After': '60',  # Retry after 60 seconds
    'X-RateLimit-Limit': '100',
    'X-RateLimit-Remaining': '0',
    'X-RateLimit-Reset': '1641234567',
}
```

### Distributed Rate Limiting with Redis

For multiple servers behind a load balancer, use Redis to maintain shared token state:

```python
import redis
import time

class DistributedRateLimiter:
    def __init__(self, redis_client, capacity, refill_rate):
        self.redis = redis_client
        self.capacity = capacity
        self.refill_rate = refill_rate
    
    def allow_request(self, user_id):
        key = f"rate_limit:{user_id}"
        
        # Lua script to atomically update tokens in Redis
        lua_script = """
        local key = KEYS[1]
        local capacity = tonumber(ARGV[1])
        local refill_rate = tonumber(ARGV[2])
        local now = tonumber(ARGV[3])
        
        local data = redis.call('HGETALL', key)
        local tokens = tonumber(data[2]) or capacity
        local last_refill = tonumber(data[4]) or now
        
        local elapsed = math.max(0, now - last_refill)
        tokens = math.min(capacity, tokens + elapsed * refill_rate)
        
        if tokens >= 1 then
            tokens = tokens - 1
            redis.call('HSET', key, 'tokens', tokens, 'last_refill', now)
            redis.call('EXPIRE', key, 3600)
            return 1
        else
            redis.call('HSET', key, 'tokens', tokens, 'last_refill', now)
            redis.call('EXPIRE', key, 3600)
            return 0
        end
        """
        
        result = self.redis.eval(lua_script, 1, key, self.capacity, self.refill_rate, time.time())
        return result == 1

# Usage
redis_client = redis.StrictRedis(host='localhost', port=6379)
limiter = DistributedRateLimiter(redis_client, capacity=100, refill_rate=5)

if limiter.allow_request(user_id="user123"):
    print("Request allowed")
else:
    print("HTTP 429: Too Many Requests")
```

## Interview Practice Questions

### Token Bucket & Rate Limiting

1. **How would you implement a token bucket rate limiter in Python for a single-process web service?**
   - Describe structure: tokens, capacity, refill_rate, last_refill_time.
   - Show how to check if request allowed.
   - Complexity: O(1) per request.

2. **How would you adapt your rate limiter to work correctly across multiple instances behind a load balancer?**
   - Need shared state in Redis.
   - Use Lua script for atomic updates (compare-and-swap).
   - Ensure no race conditions.

3. **How would you design rate limits per user, per API key, and per IP at the same time?**
   - Implement separate token buckets for each dimension.
   - One user maxed out shouldn't affect others.
   - Use different Redis keys: `rate_limit:user:{id}`, `rate_limit:api_key:{key}`, `rate_limit:ip:{addr}`.

4. **What's the difference between token bucket and leaky bucket, and when would you use each?**
   - Token bucket: Burst-friendly, average rate enforced.
   - Leaky bucket: Smooth rate, no bursts.
   - Use token bucket for APIs that can handle bursts, leaky bucket for steady streams.

5. **How would you expose rate-limit information to clients (headers, error codes, retry behavior)?**
   - Use HTTP 429 Too Many Requests.
   - Include retry-after header.
   - Return X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset headers.

6. **What happens if a user sends a burst of requests when the bucket is at capacity?**
   - Discuss token bucket's ability to handle bursts vs fixed window.
   - Example: Bucket capacity 100, refill rate 10/sec. Idle for 10 sec → 100 tokens. Can send 100 requests instantly.

7. **How would you handle a user with unlimited tokens? Prevent abuse?**
   - Implement hard limits (per second throughput cap).
   - Use circuit breaker if user consistently abuses.
   - Return HTTP 429 with Retry-After header.

8. **Compare token bucket, leaky bucket, sliding window, and fixed window approaches.**
   - Token bucket: Burst-friendly, average rate enforced.
   - Leaky bucket: Smooth rate, no bursts.
   - Sliding window: Smooth, accurate, but complex.
   - Fixed window: Simple, but window boundary issues.

## Key Takeaways for Interviews

- **Token bucket parameters:** Capacity (burst size) and refill rate (average rate).
- **Implementation:** Can be in-memory for single server, Redis for distributed.
- **Advantages:** Allows bursts while enforcing average rate, fair across users.
- **Use cases:** API rate limiting, preventing DDoS, resource protection.
- **Trade-offs:** More complex than fixed window, but smoother limiting.
