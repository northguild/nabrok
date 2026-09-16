# MongoDB Cheatsheet

A crash course in MongoDB — what it is, how it differs from SQL databases, and the commands you'll actually use.

## What is MongoDB?

MongoDB is a **NoSQL document database**. Instead of tables and rows, it stores data as **documents** (JSON-like objects) in **collections**.

### SQL vs MongoDB: The Big Difference

| Relational (SQL) | Document (MongoDB) |
|-----------------|-------------------|
| Database | Database |
| Table | Collection |
| Row | Document |
| Column | Field |
| JOIN | Embedding or Reference |
| Schema (fixed) | Schema-less (flexible) |

### Why "Document"?

In MongoDB, each row is a **document** — a flexible JSON-like object. Unlike SQL rows that must match a fixed schema, each document can have different fields:

```json
// Document 1
{
  "_id": ObjectId("..."),
  "name": "Alice",
  "email": "alice@example.com",
  "age": 30
}

// Document 2 (different structure!)
{
  "_id": ObjectId("..."),
  "name": "Bob",
  "email": "bob@example.com",
  "address": {
    "street": "123 Main St",
    "city": "Portland",
    "state": "OR"
  },
  "phones": ["555-0100", "555-0200"]
}
```

This flexibility is MongoDB's superpower — but it also means you lose some guarantees that SQL databases provide (like foreign keys and transactions... well, MongoDB has multi-document transactions now, but they're more expensive).

### When to Use MongoDB?

| Use MongoDB ✅ | Use PostgreSQL/MySQL ✅ |
|---------------|------------------------|
| Flexible/evolving schemas | Strict schema needed |
| Hierarchical data (nested objects) | Heavy JOINs between tables |
| Rapid prototyping | Complex analytics/reporting |
| Content management (CMS) | Financial transactions |
| Real-time analytics | ACID compliance critical |

## MongoDB Shell (`mongosh`)

Connect from inside Docker:

```bash
docker exec -it mongodb mongosh
```

## Crash Course: Creating Data

### 1. Use a Database

```javascript
// Switch to (or create) a database
use appdb

// List databases
show dbs

// List collections in current database
show collections
```

### 2. Insert Documents

```javascript
// Insert one document
db.users.insertOne({
  name: "Alice",
  email: "alice@example.com",
  age: 30
});

// Insert multiple documents
db.users.insertMany([
  { name: "Bob", email: "bob@example.com", age: 25 },
  { name: "Charlie", email: "charlie@example.com", age: 35 }
]);

// Insert a document with nested data
db.users.insertOne({
  name: "Dave",
  email: "dave@example.com",
  address: {
    street: "456 Oak Ave",
    city: "Seattle",
    state: "WA"
  },
  hobbies: ["coding", "hiking", "photography"]
});
```

## Crash Course: Reading Data

### Find Documents

```javascript
// Get ALL documents in a collection
db.users.find();

// Pretty-print (format the output)
db.users.find().pretty();

// Get ONE document
db.users.findOne({ name: "Alice" });

// Filter by field
db.users.find({ age: 30 });

// Multiple conditions (AND — just list them)
db.users.find({ age: { $gte: 25 }, age: { $lte: 35 } });

// OR conditions
db.users.find({ $or: [{ age: 25 }, { age: 35 }] });

// Match by nested field
db.users.find({ "address.city": "Seattle" });

// Match array element
db.users.find({ hobbies: "coding" });

// Sort results (1 = ascending, -1 = descending)
db.users.find().sort({ age: -1 });  // Oldest first

// Limit results
db.users.find().limit(5);

// Skip results (for pagination)
db.users.find().skip(10).limit(10);  // Page 2, 10 per page

// Project: only return specific fields
db.users.find({}, { name: 1, email: 1, _id: 0 });
// Returns: { name: "Alice", email: "alice@example.com" }
```

### Comparison Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `$eq` | Equal | `{ age: { $eq: 30 } }` |
| `$gt` | Greater than | `{ age: { $gt: 25 } }` |
| `$gte` | Greater than or equal | `{ age: { $gte: 25 } }` |
| `$lt` | Less than | `{ age: { $lt: 35 } }` |
| `$lte` | Less than or equal | `{ age: { $lte: 35 } }` |
| `$ne` | Not equal | `{ status: { $ne: "deleted" } }` |
| `$in` | In a list | `{ age: { $in: [25, 30, 35] } }` |
| `$nin` | Not in a list | `{ status: { $nin: ["spam", "deleted"] } }` |

### Logical Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `$and` | All conditions true | `{ $and: [{ age: { $gt: 25 } }, { age: { $lt: 35 } }] }` |
| `$or` | Any condition true | `{ $or: [{ city: "Portland" }, { city: "Seattle" }] }` |
| `$not` | Negate a condition | `{ age: { $not: { $gt: 30 } } }` |
| `$nor` | None true | `{ $nor: [{ city: "Portland" }, { city: "Seattle" }] }` |

### Array Operators

```javascript
// Array has exact order and elements
db.users.find({ hobbies: ["coding", "hiking"] });

// Array contains at least one element
db.users.find({ hobbies: { $all: ["coding", "hiking"] } });

// Array length
db.users.find({ phones: { $size: 2 } });

// Nested array elements
db.users.find({ "phones.0": "555-0100" });  // First phone
```

## Crash Course: Modifying Data

### Update Documents

```javascript
// Update ONE document (first match)
db.users.updateOne(
  { email: "alice@example.com" },           // Filter
  { $set: { age: 31 } }                      // Update: set age to 31
);

// Update ALL matching documents
db.users.updateMany(
  { status: "inactive" },                    // Filter
  { $set: { status: "active" } }             // Update
);

// Delete a field
db.users.updateOne(
  { name: "Alice" },
  { $unset: { temporary_field: "" } }
);

// Increment a number
db.users.updateOne(
  { name: "Alice" },
  { $inc: { score: 10 } }                    // Add 10 to score
);

// Push to an array (add element)
db.users.updateOne(
  { name: "Alice" },
  { $push: { hobbies: "gaming" } }           // Add to array
);

// Pull from an array (remove element)
db.users.updateOne(
  { name: "Alice" },
  { $pull: { hobbies: "gaming" } }           // Remove from array
);
```

### Update Operators Quick Reference

| Operator | What it does | Example |
|----------|--------------|---------|
| `$set` | Set a field value | `{ $set: { age: 31 } }` |
| `$unset` | Remove a field | `{ $unset: { temp: "" } }` |
| `$inc` | Increment/decrement | `{ $inc: { views: 1 } }` |
| `$push` | Add to array | `{ $push: { tags: "new" } }` |
| `$pull` | Remove from array | `{ $pull: { tags: "old" } }` |
| `$addToSet` | Add if not exists | `{ $addToSet: { tags: "unique" } }` |
| `$rename` | Rename a field | `{ $rename: { oldName: "newName" } }` |

### Delete Documents

```javascript
// Delete ONE document
db.users.deleteOne({ email: "bob@example.com" });

// Delete ALL matching documents
db.users.deleteMany({ status: "deleted" });

// Delete ALL documents (DANGER!)
db.users.deleteMany({});  // ⚠️ Removes everything!
```

## MongoDB Data Types

| Type | Example | Use For |
|------|---------|---------|
| `String` | `"Hello"` | Text |
| `Integer` | `42` | Whole numbers |
| `Double` | `3.14` | Decimals |
| `Boolean` | `true/false` | True/false |
| `Date` | `new Date()` | Dates/times |
| `Array` | `["a", "b"]` | Lists |
| `Object` | `{ nested: true }` | Embedded documents |
| `ObjectId` | `ObjectId("...")` | Unique ID (default) |
| `Null` | `null` | Missing/empty value |

## ObjectId: MongoDB's Default ID

Every document gets a unique `_id` automatically. MongoDB uses **ObjectId** — a 12-byte identifier:

```javascript
// ObjectId structure:
// 5f9d8c7b6a5e4d3c2b1a0000
// ────────────────────────
// timestamp  machine  counter  increment

// Create a new ObjectId
new ObjectId()

// Query by specific ID
db.users.find({ _id: ObjectId("5f9d8c7b6a5e4d3c2b1a0000") });

// Convert string to ObjectId
ObjectId("5f9d8c7b6a5e4d3c2b1a0000")
```

## Useful MongoDB Shell Commands

```javascript
// Switch database
use appdb

// List databases
show dbs

// List collections
show collections

// Count documents
db.users.countDocuments();

// Drop a collection (DANGER!)
db.users.drop();

// Drop a database (DANGER!)
db.dropDatabase();

// Get help
db.help()
db.users.help()
```

## MongoDB vs SQL: Concept Mapping

| SQL | MongoDB | Example |
|-----|---------|---------|
| `SELECT * FROM users` | `db.users.find()` | Get all users |
| `SELECT * FROM users WHERE age > 25` | `db.users.find({ age: { $gt: 25 } })` | Filter |
| `INSERT INTO users VALUES (...)` | `db.users.insertOne({...})` | Insert |
| `UPDATE users SET age=31 WHERE name='Alice'` | `db.users.updateOne({name:'Alice'}, {$set:{age:31}})` | Update |
| `DELETE FROM users WHERE id=1` | `db.users.deleteOne({_id: ObjectId(...)})` | Delete |
| `JOIN orders ON users.id = orders.user_id` | Embed orders in user document OR use `$lookup` | Relate data |

## MongoDB Express Tips

- **Browse**: Click a collection to see all documents
- **Insert**: Click "Insert Document" to add new documents via form
- **Find**: Use the search bar to filter documents
- **Edit**: Click any document to view/edit it
- **Delete**: Hover over a document → click the trash icon
- **Export**: Click "Export" to download as JSON or CSV
- **Stats**: View collection statistics (document count, size)

## When to Embed vs Reference

This is the **most important design decision** in MongoDB:

### Embed (put data inside the parent document)

```javascript
// Good for: one-to-few, data always accessed together
{
  _id: ObjectId("..."),
  name: "Alice",
  address: {
    street: "123 Main St",
    city: "Portland"
  }
}
```

### Reference (store ID, look up separately)

```javascript
// Good for: one-to-many, data accessed independently, large datasets
// Users collection:
{ _id: ObjectId("..."), name: "Alice" }

// Orders collection:
{ _id: ObjectId("..."), user_id: ObjectId("..."), amount: 99.99 }
```

**Rule of thumb**: Embed if the data is small (< 1MB total) and always used together. Reference if it's large, grows unbounded, or used independently.

## Next Steps

- **Indexes**: Speed up queries (`db.users.createIndex({ email: 1 })`)
- **Aggregation Pipeline**: Complex data transformations
- **Transactions**: Multi-document ACID transactions
- **Validation**: Schema validation on collections
