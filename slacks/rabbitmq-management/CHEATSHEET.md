# RabbitMQ Cheatsheet

Quick reference for RabbitMQ concepts, management UI, and CLI tools.

## What is RabbitMQ?

RabbitMQ is a **message broker** — it receives messages from producers and routes them to consumers. Think of it as a post office: you send letters (messages) to the post office, and it delivers them to the right mailbox (queue).

## Core Concepts

| Concept | Description | Analogy |
|---------|-------------|---------|
| **Producer** | Application that sends messages | Mail sender |
| **Consumer** | Application that receives messages | Mail recipient |
| **Queue** | Buffer that stores messages | Mailbox |
| **Exchange** | Routes messages to queues | Post office sorting |
| **Binding** | Rule connecting exchange to queue | Delivery address rule |
| **VHost** | Virtual host (isolated namespaces) | Separate post offices |
| **AMQP** | Protocol for messaging | Postal service rules |

## Message Flow

```
Producer → Exchange → Binding → Queue → Consumer
                    ↘ Binding → Queue → Consumer
```

## Exchange Types

| Type | How it routes | Example |
|------|---------------|---------|
| **direct** | Exact match on routing key | `order.created` → queue with same key |
| **fanout** | Broadcast to all queues | Newsletter to all subscribers |
| **topic** | Pattern match with wildcards | `*.error` matches `db.error`, `api.error` |
| **headers** | Match on message headers | Custom header-based routing |

## RabbitMQ Management UI (http://localhost:15672)

### Dashboard
- Overview of connections, channels, queues, exchanges
- System metrics (CPU, memory, disk)

### Queues Tab
- List all queues with message counts
- View queue details: messages ready, pending, delivered
- Purge messages or delete queues

### Exchanges Tab
- List all exchanges and their types
- Declare new exchanges
- View bindings

### Admin Tab
- Create/manage users and permissions
- Create virtual hosts (VHosts)
- Set policies and quotas

## CLI Tools

### `rabbitmqadmin` (built-in)

Download from the management UI: **Admin → rabbitmqadmin**

```bash
# Declare a queue
rabbitmqadmin declare queue name=my-queue durable=true

# Declare an exchange
rabbitmqadmin declare exchange name=my-exchange type=direct

# Bind them
rabbitmqadmin declare binding source=my-exchange destination=my-queue routing_key=my.key

# Publish a message
rabbitmqadmin publish exchange="" routing_key=my-queue body="Hello!"

# Consume messages (blocks until Ctrl+C)
rabbitmqadmin consume queue=my-queue

# List queues
rabbitmqadmin list queues

# List exchanges
rabbitmqadmin list exchanges

# List bindings
rabbitmqadmin list bindings
```

### `rabbitmqctl` (server CLI)

```bash
# Run inside the container
docker exec -it rabbitmq rabbitmqctl status

# List users
docker exec -it rabbitmq rabbitmqctl list_users

# List vhosts
docker exec -it rabbitmq rabbitmqctl list_vhosts

# Create a new user
docker exec -it rabbitmq rabbitmqctl add_user myuser mypassword

# Set user as admin
docker exec -it rabbitmq rabbitmqctl set_user_tags myuser administrator

# Create a vhost
docker exec -it rabbitmq rabbitmqctl add_vhost my-vhost

# Set permissions for a user on a vhost
docker exec -it rabbitmq rabbitmqctl set_permissions -p my-vhost myuser ".*" ".*" ".*"

# Clear a queue (remove all messages)
docker exec -it rabbitmq rabbitmqctl purge_queue my-queue
```

## Common Patterns

### 1. Simple Request/Reply

```
Producer → Queue → Consumer
Consumer → Reply Queue → Producer
```

### 2. Work Queues (Task Distribution)

```
Producer → Queue → Consumer 1
              → Consumer 2
              → Consumer 3
```

Multiple consumers compete for messages in the same queue. Use `prefetch_count` to control distribution.

### 3. Pub/Sub (Broadcast)

```
Producer → Fanout Exchange → Queue A → Consumer 1
                      → Queue B → Consumer 2
                      → Queue C → Consumer 3
```

All consumers get all messages.

### 4. Routing (Filtered Delivery)

```
Producer → Direct Exchange → Queue A (routing_key: error)
                      → Queue B (routing_key: warning)
                      → Queue C (routing_key: info)
```

Only matching consumers get messages.

### 5. Topic Exchange (Pattern Matching)

```
Producer → Topic Exchange → Queue A (*.error)
                      → Queue B (#.critical)
```

- `*` matches exactly one word
- `#` matches zero or more words
- `*.error` matches `db.error` but not `db.error.critical`
- `#.critical` matches `db.critical`, `api.db.critical`, etc.

## Creating in the Management UI

### Create a Queue
1. Go to **Queues → Add a new queue**
2. Name: `my-queue`
3. Type: **Standard** (default) or **Quorum** (highly available)
4. Durable: ✅ (persists across restarts)
5. Click **Add queue**

### Create an Exchange
1. Go to **Exchanges → Add a new exchange**
2. Name: `my-exchange`
3. Type: **direct**, **fanout**, **topic**, or **headers**
4. Durable: ✅
5. Click **Add exchange**

### Create a Binding
1. Click on an exchange name
2. Scroll to **Bindings** section
3. Queue: select your queue
4. Routing key: enter the key (e.g., `order.created`)
5. Click **Bind**

## Node.js Connection Example

```javascript
import amqp from 'amqplib';

// Connect
const connection = await amqp.connect('amqp://guest:guest@localhost');
const channel = await connection.createChannel();

// Declare queue (idempotent)
await channel.assertQueue('my-queue', { durable: true });

// Send a message
channel.sendToQueue('my-queue', Buffer.from('Hello RabbitMQ!'));

// Consume messages
channel.consume('my-queue', (msg) => {
  console.log('Received:', msg.content.toString());
  channel.ack(msg);  // Confirm processing
});
```

## Docker Tips for RabbitMQ

```bash
# View logs
docker logs -f rabbitmq

# Shell into container
docker exec -it rabbitmq sh

# Check broker status
docker exec -it rabbitmq rabbitmqctl status

# Reset everything (DANGER!)
docker exec -it rabbitmq rabbitmqctl stop_app
docker exec -it rabbitmq rabbitmqctl reset
docker exec -it rabbitmq rabbitmqctl start_app
```

## Useful Dashboard Ideas

1. **Request Rate**: `sum by (handler) (rate(http_requests_total[5m]))`
2. **Error Rate**: `sum(rate(http_requests_total{status=~"5.."}[5m])) * 100`
3. **P95 Latency**: `histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))`
4. **Active Users**: `count(unique(user_id))`
5. **Queue Depth**: `queue_length`

## Stop & Clean Up

```bash
docker compose down          # stop containers
docker compose down -v       # stop and remove volumes (wipes data)
```
