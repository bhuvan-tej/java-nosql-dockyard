# 01 · Traditional RDBMS vs MongoDB

## 1. Vocabulary map

| RDBMS (MySQL, PostgreSQL) | MongoDB |
|---|---|
| Database | Database |
| Table | Collection |
| Row | Document (BSON, JSON-like) |
| Column | Field |
| Primary key | `_id` (auto `ObjectId` if not given) |
| Foreign key | Reference (stored id, **not enforced**) |
| JOIN | Embedding, or `$lookup` in aggregation |
| `GROUP BY` | Aggregation pipeline (`$group`) |
| Index | Index |
| Schema (DDL) | Optional schema validation (`$jsonSchema`) |
| Transaction | Single-document ops are atomic; multi-document transactions on replica sets |

## 2. Same data, two shapes

**RDBMS**: normalized across tables, assembled with JOINs.

```sql
CREATE TABLE users     (id INT PRIMARY KEY, name VARCHAR(100), email VARCHAR(100), age INT);
CREATE TABLE addresses (id INT PRIMARY KEY, user_id INT REFERENCES users(id), city VARCHAR(50));

SELECT u.name, a.city
FROM users u JOIN addresses a ON a.user_id = u.id
WHERE u.email = 'ada@dockyard.dev';
```

**MongoDB**: data that is read together is stored together.

```js
db.users.insertOne({
  name: "Ada Lovelace",
  email: "ada@dockyard.dev",
  age: 36,
  addresses: [ { city: "London" } ],
  skills: ["math", "programming"]
})

db.users.find(
  { email: "ada@dockyard.dev" },
  { name: 1, "addresses.city": 1 }
)
```

There are no JOINs here: one read returns the whole user.

## 3. Key differences

| | RDBMS | MongoDB |
|---|---|---|
| Schema | Fixed; `ALTER TABLE` to change | Flexible; documents in a collection can differ |
| Data model | Normalized, design around entities | Design around **how the app queries** |
| Relationships | JOINs + foreign keys | Embed (1-to-few) or reference (1-to-many/many-to-many) |
| Scaling | Mostly vertical (bigger server) | Horizontal via **sharding** |
| High availability | Varies by vendor | Built-in **replica sets** (Atlas runs 3 nodes) |
| Query language | SQL | MQL (JSON-style filters + aggregation pipeline) |
| Nested / array data | Extra tables | Native (`addresses.city`, `skills: "math"`) |

## 4. SQL to MQL cheat sheet

| SQL | MongoDB (mongosh) |
|---|---|
| `INSERT INTO users ...` | `db.users.insertOne({...})` / `insertMany([...])` |
| `SELECT * FROM users` | `db.users.find()` |
| `SELECT name, email FROM users` | `db.users.find({}, { name: 1, email: 1, _id: 0 })` |
| `WHERE age = 36` | `find({ age: 36 })` |
| `WHERE age > 30` | `find({ age: { $gt: 30 } })` |
| `WHERE age BETWEEN 18 AND 30` | `find({ age: { $gte: 18, $lte: 30 } })` |
| `WHERE a = 1 AND b = 2` | `find({ a: 1, b: 2 })` |
| `WHERE a = 1 OR b = 2` | `find({ $or: [ { a: 1 }, { b: 2 } ] })` |
| `WHERE city IN ('London','Paris')` | `find({ city: { $in: ["London", "Paris"] } })` |
| `WHERE name LIKE 'Ad%'` | `find({ name: { $regex: "^Ad" } })` |
| `WHERE email IS NULL` | `find({ email: null })` (also matches missing field) |
| `ORDER BY age DESC LIMIT 5 OFFSET 10` | `find().sort({ age: -1 }).skip(10).limit(5)` |
| `SELECT COUNT(*) ... WHERE age > 30` | `db.users.countDocuments({ age: { $gt: 30 } })` |
| `UPDATE users SET age = 37 WHERE email = ...` | `updateOne({ email: "..." }, { $set: { age: 37 } })` |
| `UPDATE users SET age = age + 1` | `updateMany({}, { $inc: { age: 1 } })` |
| `DELETE FROM users WHERE age < 18` | `deleteMany({ age: { $lt: 18 } })` |
| `SELECT city, COUNT(*) FROM ... GROUP BY city` | `aggregate([ { $group: { _id: "$city", count: { $sum: 1 } } } ])` |

## 5. When to pick which

**MongoDB fits** when data is hierarchical or varies in shape (catalogs, profiles, content, events, IoT), when the schema changes often, or when you need to scale writes horizontally.

**An RDBMS fits** when data is highly relational with many ad-hoc JOINs, when strict constraints and foreign keys are core to correctness, or when you run heavy multi-row transactional workloads (ledgers, banking).

## 6. Try it

In Atlas, open **Data Explorer** or connect with `mongosh "<your Atlas URI>"`, then:

```js
use dockyard
db.users.insertOne({ name: "Test", email: "test@dockyard.dev", age: 20 })
db.users.find({ age: { $gte: 18 } })
db.users.deleteOne({ email: "test@dockyard.dev" })
```
