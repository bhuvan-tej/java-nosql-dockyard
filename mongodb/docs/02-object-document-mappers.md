# 02 · Object Mappers in MongoDB (ODM)

## 1. ORM vs ODM

- **ORM** (Object-Relational Mapper, e.g. Hibernate/JPA) maps classes to **tables and rows**. It has to translate object graphs into JOINs.
- **ODM** (Object-Document Mapper) maps classes to **documents**. Nested objects and lists map directly to embedded documents and arrays, so there is far less to translate.

```
Java object  <->  ODM  <->  BSON document  <->  MongoDB
```

## 2. Options in Java

| Option | What it is | Use when |
|---|---|---|
| **MongoDB Java Driver** | Official low-level driver; `Document` or POJO codecs | You want full control, no framework |
| **Spring Data MongoDB** | ODM on top of the driver: `MongoRepository`, `MongoTemplate` | Spring Boot apps |
| **Morphia** | Standalone ODM with annotations | Non-Spring apps |
| **Eclipse JNoSQL** | Jakarta NoSQL standard API | Jakarta EE / vendor-neutral code |

## 3. Mapping annotations (Spring Data MongoDB)

| Annotation | Purpose | JPA analogue |
|---|---|---|
| `@Document("users")` | Class maps to the `users` collection | `@Entity @Table` |
| `@Id` | Maps to `_id` (`String` gets an auto-generated `ObjectId`) | `@Id` |
| `@Field("e_mail")` | Different field name in the DB | `@Column(name=)` |
| `@Indexed(unique = true)` | Create an index (see note below) | `@Column(unique=)` |
| `@DocumentReference` / `@DBRef` | Reference another document | `@ManyToOne` |
| `@Transient` | Don't persist | `@Transient` |

Note: automatic index creation is **off** by default in Spring Boot. Enable it with `spring.data.mongodb.auto-index-creation=true`, or create indexes explicitly (covered in the indexing module).

## 4. How a Java object becomes a document

```java
@Document("users")
public record User(@Id String id, String name, String email, int age, Instant createdAt) {}

userRepository.save(new User(null, "Ada Lovelace", "ada@dockyard.dev", 36, Instant.now()));
```

This is stored as:

```js
{
  _id: ObjectId("..."),
  name: "Ada Lovelace",
  email: "ada@dockyard.dev",
  age: 36,
  createdAt: ISODate("..."),
  _class: "com.dockyard.mongodb.crud.User"   // type hint added by Spring Data
}
```

## 5. Three ways to query, and the MQL each produces

**a) Derived queries.** The method name becomes the query:

| Repository method | MQL filter |
|---|---|
| `findByEmail(e)` | `{ email: e }` |
| `findByAgeGreaterThan(a)` | `{ age: { $gt: a } }` |
| `findByAgeBetween(a, b)` | `{ age: { $gt: a, $lt: b } }` (exclusive by default) |
| `findByNameStartingWith(p)` | `{ name: { $regex: "^p" } }` |
| `findByEmailIn(list)` | `{ email: { $in: list } }` |
| `findByNameAndAge(n, a)` | `{ name: n, age: a }` |
| `countByAgeGreaterThan(a)` | `countDocuments({ age: { $gt: a } })` |

**b) `@Query` with raw MQL.** You write the query yourself, which is best for learning:

```java
@Query("{ 'age': { $gte: ?0, $lte: ?1 } }")
List<User> findAdultsInRange(int min, int max);
```

**c) `MongoTemplate` + `Criteria`.** A query builder for dynamic queries:

```java
Query q = new Query(Criteria.where("age").gte(18).and("name").regex("^Ada"));
mongoTemplate.find(q, User.class);   // -> { age: { $gte: 18 }, name: { $regex: "^Ada" } }
```

## 6. Verify it in mongosh

Insert a user and query it back. This is exactly what `userRepository.save(...)` and `findByEmail(...)` do behind the scenes.
With JPA/Hibernate, the same calls would generate the SQL below.

**SQL**
```sql
INSERT INTO users (name, email, age, created_at) VALUES ('Ada Lovelace', 'ada@dockyard.dev', 36, NOW());
SELECT * FROM users WHERE email = 'ada@dockyard.dev';
SELECT name, age FROM users WHERE age BETWEEN 30 AND 40;
```

**MongoDB**
```js
use dockyard
db.users.insertOne({ name: "Ada Lovelace", email: "ada@dockyard.dev", age: 36, createdAt: new Date() })
db.users.find({ email: "ada@dockyard.dev" })
db.users.find({ age: { $gte: 30, $lte: 40 } }, { name: 1, age: 1, _id: 0 })
db.users.deleteOne({ email: "ada@dockyard.dev" })   // clean up when done
```
