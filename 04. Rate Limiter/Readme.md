# Chapter 4: Design a Rate Limiter

## Introduction
This chapter explores the design and implementation of a rate limiter—a system component used to control traffic rates sent by clients or services.Rate limiting means controlling how many requests a user can make within a certain time. Rate limiters are crucial for preventing abuse, reducing costs, and ensuring the stability of server resources. Examples of their use include limiting posts, account creations, and reward claims.


## Benefits of Rate Limiting
- **Preventing DoS Attacks:** Blocking excess calls to avoid resource starvation.
- **Cost Reduction:** Limiting unnecessary requests to reduce server expenses.
- **Preventing Overloads:** Filtering out excessive requests to stabilize server performance.

## Step 1: Understanding the Problem
### Key Features
- Server-side API rate limiter.
- Support for multiple throttle rules.
- Handle large-scale systems in distributed environments.
- Option for a standalone service or application-level code.
- Inform users when throttled.

### Requirements
- Accurate request throttling.
- Minimal latency.
- Low memory usage.
- Distributed capability.
- Clear exception handling.
- High fault tolerance.

## Step 2: High-Level Design
### Placement Options
<div style="margin-left:2rem">
    <img src="./images/rate_limiter_architecture.png"  alt="Rate Limiting Middleware Architecture" width="550">
</div>

1. **Client-Side Implementation:** Unreliable due to potential misuse.
2. **Server-Side Implementation:** Preferred for control and reliability.
3. **Middleware (API Gateway):** A flexible option for integrated rate limiting.


### Guidelines for Placement
- Evaluate current tech stack and choose efficient options.
- Select appropriate algorithms based on business needs.
- Use an API gateway if microservices are employed.
- Opt for commercial solutions if resources are limited.(If your team doesn't have enough time, people, or expertise to build and maintain something, consider using an existing commercial/managed solution.)

## Step 3: Rate Limiting Algorithms
### 1. Token Bucket
<div style="margin-left:2rem">
  <img src="./images/token-bucket.png"  alt="Token Bucket Algorithm" width="550">
</div>

- **Description:** Tokens are added to a bucket at a fixed rate; each request consumes a token.
- **Parameters:** Bucket size(Maximum number of tokens the bucket can hold) and refill rate(How quickly new tokens are added.).
- **Pros:** Easy to implement, memory-efficient, supports traffic bursts.
- **Cons:** Requires careful parameter tuning(Bucket size and refill rate).



### 2. Leaking Bucket
<div style="margin-left:2rem">
  <img src="./images/leaking-bucket.png"  alt="Leaking Bucket Algorithm" width="550">
</div>

- **Description:** Processes requests at a fixed rate using a FIFO queue.
- **Pros:** Memory-efficient, stable outflow rate, Useful for APIs and distributed systems.
- **Cons:** Traffic bursts may delay recent requests.(Example - 100 requests arrive at once and 5 request/sec)
  

  Example: https://github.com/uber-go/ratelimit



### 3. Fixed Window Counter
<div style="margin-left:2rem">
  <img src="./images/fixed-window-counter.png"  alt="Fixed Window Counter" width="550">
</div>

- **Description:** Divides time into fixed intervals and uses counters to limit requests.
- **Pros:** Simple to implement, memory efficient, Easy to understand, Good for straightforward rate-limiting requirements.
- **Cons:** Traffic spikes at window edges can exceed limits.

- Sudden burst of traffic at the edges of time windows
could cause more requests than allowed quota to go through.

  <img src="./images/fixed-window-issue.png"  alt="Fixed Window Issue" width="550">


### 4. Sliding Window Log
<div style="margin-left:2rem">
  <img src="./images/sliding-window-log.png"  alt="Sliding Window Log" width="550">
</div>

- **Description:** Tracks timestamps to allow a rolling time window.(store the timestamp of each request.)
- **Pros:** Accurate rate limiting.
- **Cons:** High memory consumption.
  


### 5. Sliding Window Counter
<div style="margin-left:2rem">
  <img src="./images/sliding-window-counter.png"  alt="Fixed Window Counter" width="550">
</div>

- **Description:** Combines fixed window and sliding log methods for smoothing spikes.
- **Pros:** Memory-efficient, handles traffic bursts.
- **Cons:** Approximation may not be perfectly strict.
  



## High-Level Architecture
<div style="margin-left:2rem">
  <img src="./images/architecture.png" style="margin-left: 40px; margin-top: 40px; margin-bottom: 20px;" alt="Architecture" width="550">
</div>

- **Data Storage:** Use in-memory caching (e.g., Redis) for fast counter operations.
- **Steps:**
  1. Client sends request to middleware.
  2. Middleware checks counters in Redis.
  3. Request is processed or rejected based on limits.


## Advanced Considerations
### Distributed Environments
- **Challenges:** Race conditions, synchronization issues.
- **Solutions:** Use locks, Lua scripts, or sorted sets in Redis. Employ centralized data stores for synchronization.

### Performance Optimizations
- Multi-data center setups for reduced latency.
- Eventual consistency models for synchronization.

### Monitoring
- Regular analytics to ensure algorithm effectiveness and adjust rules as needed.

## Definition
- **DoS(Denial of Service) attack:** A DoS attack happens when an attacker sends a huge number of requests to a server so that it becomes slow or unavailable to normal users.
- A server-side API is an API that runs on the server and allows other applications or clients to communicate with the server and access its functionality or data.
- **Standalone Service:** The API runs as its own separate application/service.
- **Application-Level Code:** The API is built inside an existing application.
      <div>
          Mobile App
                ↓
            Main Application
               ├── User API
               ├── Order API
               └── Payment API
                    ↓
            Database
    </div>
- **Redis:** It is used to store the request counters because it is very fast.
- **Race Condition:** A race condition happens when multiple requests access/update the same data simultaneously and the result depends on the timing.
- **locks:** Only one process can modify this data at this moment.
- **Redis Sorted Sets:** A Sorted Set can store request timestamps.
