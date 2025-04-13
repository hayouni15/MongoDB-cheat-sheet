# 📚 MongoDB + Mongoose Cheat Sheet

## 🔹 Basic CRUD Operations

### Insert

```js
db.collection.insertOne({ name: "John", age: 30 });
db.collection.insertMany([{ name: "Alice" }, { name: "Bob" }]);
```

### Find

```js
db.collection.find(); // All documents
db.collection.find({ age: 25 }); // Filter by field
db.collection.findOne({ name: "John" }); // Find one
```

### Update

```js
db.collection.updateOne({ name: "John" }, { $set: { age: 31 } });
db.collection.updateMany({}, { $set: { status: "active" } });
```

### Delete

```js
db.collection.deleteOne({ name: "John" });
db.collection.deleteMany({ status: "inactive" });
```

---

## 🔹 Query Operators

| Operator | Description              |
| -------- | ------------------------ |
| `$gt`    | Greater than             |
| `$lt`    | Less than                |
| `$gte`   | Greater than or equal    |
| `$lte`   | Less than or equal       |
| `$in`    | Value in array           |
| `$nin`   | Value not in array       |
| `$ne`    | Not equal                |
| `$or`    | Logical OR               |
| `$and`   | Logical AND              |
| `$not`   | Logical NOT              |
| `$regex` | Regular expression match |

```js
db.users.find({ age: { $gt: 18, $lt: 60 } });
db.products.find({ name: { $regex: /phone/i } });
```

---

## 🔹 Projections

```js
db.users.find({}, { name: 1, age: 1, _id: 0 }); // Include only name and age
```

---

## 🔹 Sorting, Limiting, Skipping

```js
db.collection.find().sort({ age: -1 }).limit(10).skip(5);
```

---

## 🔹 Aggregation Pipelines

```js
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $group: { _id: "$customerId", total: { $sum: "$amount" } } },
  { $sort: { total: -1 } },
]);
```

### Common Stages

- `$match`: Filter documents
- `$group`: Aggregate values
- `$project`: Include/reshape fields
- `$sort`, `$limit`, `$skip`, `$unwind`
- `$lookup`: Join with another collection
- `$addFields`: Add computed fields

### Example with `$lookup`

```js
db.tasks.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "ownerEmail",
      foreignField: "email",
      as: "ownerInfo",
    },
  },
  { $unwind: "$ownerInfo" },
]);
```

---

## 🔹 Mongoose `.populate()`

### Basic Usage

```js
Task.find().populate("owner");
```

### Advanced

```js
Task.find().populate({
  path: "owner",
  select: "name email",
  match: { age: { $gte: 18 } },
  options: { limit: 5 },
});
```

---

## 🔹 Mongoose Virtuals

Virtuals are **computed properties** not stored in the database.

### Example: Virtual Field on User → Tasks

```js
const userSchema = new mongoose.Schema({ ... });

userSchema.virtual("tasks", {
  ref: "Task",
  localField: "_id",
  foreignField: "owner"
});
```

Now you can do:

```js
const user = await User.findById(id).populate("tasks");
```

---

## 🔹 Mongoose `.populate()` vs MongoDB `$lookup`

| Mongoose                  | MongoDB                     |
| ------------------------- | --------------------------- |
| `.populate('refField')`   | `$lookup`                   |
| `ref` points to a model   | `from` points to collection |
| Joins by `_id` by default | Can join by any field       |

```js
// Mongoose
Task.find().populate({
  path: "ownerEmail",
  model: "User",
  localField: "ownerEmail",
  foreignField: "email",
  justOne: true,
});

// MongoDB
db.tasks.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "ownerEmail",
      foreignField: "email",
      as: "ownerEmail",
    },
  },
  { $unwind: "$ownerEmail" },
]);
```

---

## 🔹 Indexes

```js
db.collection.createIndex({ email: 1 }, { unique: true });
db.collection.getIndexes();
```

---

## 🔹 Text Search

```js
db.articles.createIndex({ title: "text", body: "text" });
db.articles.find({ $text: { $search: "mongodb cheat sheet" } });
```

---

## 🔹 Schema Validation (optional)

```js
db.createCollection("users", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "email"],
      properties: {
        name: { bsonType: "string" },
        email: { bsonType: "string" },
      },
    },
  },
});
```

---

## 🛠 Useful Mongo Shell Commands

```bash
mongosh                 # Open MongoDB shell
show dbs                # List databases
use mydb                # Switch/create database
show collections        # List collections
db.dropDatabase()       # Delete current DB
db.collection.drop()    # Delete a collection
```

---

## ✅ Tips

- `_id` is auto-generated and indexed.
- Use `ObjectId("...")` when querying `_id`.
- Use `.populate()` in Mongoose for relationships.
- Use `$lookup` in raw MongoDB for joins.
- Use virtuals for inverse relationships in Mongoose.
- Use aggregation when you need stats or joins.
- Avoid storing passwords in plain text – always hash them.
