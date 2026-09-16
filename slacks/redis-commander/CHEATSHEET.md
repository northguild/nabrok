# Redis Cheatsheet

Quick reference for Redis commands, data types, and common patterns.

## Core Commands

```bash
# Connect to Redis
redis-cli

# Check server status
PING

# Select database (Redis has 16 by default, 0-15)
SELECT 0

# List all keys
KEYS *

# Check if a key exists
EXISTS mykey

# Get the type of a key
TYPE mykey

# Remove a key
DEL mykey

# Set expiration on a key (in seconds)
EXPIRE mykey 60

# Get remaining TTL (time to live)
TTL mykey

# View all keys with their TTLs
KEYS * | xargs -I {} redis-cli TTL {}
```

## String Commands

```bash
# Set and get a string value
SET name "Nabrok"
GET name

# Increment/decrement a number
SET counter 0
INCR counter        # → 1
INCRBY counter 10   # → 11
DECR counter        # → 10
DECRBY counter 3    # → 7

# Set with expiration in one command
SETEX token 3600 "abc123"

# Append to a string
APPEND greeting " world"

# Get the length of a string
STRLEN greeting
```

## Hash Commands (key-value pairs, like a mini-object)

```bash
# Set fields in a hash
HSET user:1 name "Alice" email "alice@example.com" age 30

# Get all fields
HGETALL user:1

# Get specific fields
HMGET user:1 name email

# Get a single field
HGET user:1 name

# Check if a field exists
HEXISTS user:1 phone

# Delete specific fields
HDEL user:1 age

# Increment a numeric field
HINCRBY user:1 age 1
```

## List Commands (ordered sequence, like an array)

```bash
# Push to list
LPUSH tasks "review PR" "write docs" "fix bug"

# Pop from list
RPOP tasks            # remove from end
LPOP tasks            # remove from start

# Get all items
LRANGE tasks 0 -1     # 0 to -1 means all

# Get list length
LLEN tasks

# Push to specific position
LPUSH tasks "urgent"  # push to front

# Block and wait for item (useful for queues)
BRPOP tasks 30        # block up to 30 seconds
```

## Set Commands (unordered unique values)

```bash
# Create a set
SSET tags "redis" "database" "cache"

# Add members
SADD tags "nosql" "open-source"

# Get all members
SMEMBERS tags

# Check membership
SISMEMBER tags "redis"    # → 1 (true)

# Remove members
SREM tags "cache"

# Set operations
SUNION set1 set2          # union
SINTER set1 set2          # intersection
SDIFF set1 set2           # difference
```

## Sorted Set Commands (unique values with scores)

```bash
# Add member with score
ZADD leaderboard 150 "Alice"
ZADD leaderboard 200 "Bob"
ZADD leaderboard 100 "Charlie"

# Get rank (0-indexed, lowest to highest)
ZRANGE leaderboard 0 -1 WITHSCORES

# Get top 3
ZREVRANGE leaderboard 0 2 WITHSCORES

# Get score for a member
ZSCORE leaderboard "Bob"

# Increment score
ZINCRBY leaderboard 50 "Alice"   # Alice: 200

# Range by score
ZRANGEBYSCORE leaderboard 100 200
```

## Common Patterns

### Caching

```bash
# Cache with TTL
SET cache:key "value" EX 300 NX    # only set if not exists, 5 min TTL

# Check cache, then fetch from DB
GET cache:key                       # → nil (miss)
SET cache:key "fetched_value" EX 300
```

### Rate Limiting

```bash
# Simple rate limiter (10 requests per minute)
INCR rate:limit:user:123
EXPIRE rate:limit:user:123 60
GET rate:limit:user:123           # → 10 means blocked
```

### Pub/Sub (publish/subscribe messaging)

```bash
# Subscribe to a channel
SUBSCRIBE notifications

# Publish a message (from another client)
PUBLISH notifications "Hello!"
```

### Bitmaps (bit-level operations)

```bash
# Track daily activity (set bit for today)
SETBIT activity:2026-09-16 5 1     # set bit 5

# Count active bits
BITCOUNT activity:2026-09-16
```

## Redis Commander Tips

- **Browse keys**: Click on the database number in the sidebar to see all keys
- **Edit values**: Click any key to view/edit its value
- **Delete keys**: Hover over a key and click the trash icon
- **Search**: Use the search box to filter keys by pattern
- **Export**: Right-click a key to export its value

## Useful redis-cli Shortcuts

```bash
# Monitor commands in real-time
MONITOR

# Check memory usage
INFO memory

# Check connected clients
INFO clients

# Flush all data (DANGER!)
FLUSHALL

# Flush current database only
FLUSHDB
```
