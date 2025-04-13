# 📚 MongoDB Cheat Sheet

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

### Basic Structure

```js
db.collection.aggregate([
  { $match: { status: "active" } },
  { $group: { _id: "$category", total: { $sum: "$price" } } },
  { $sort: { total: -1 } },
]);
```

### Common Stages

| Stage        | Purpose                                    |
| ------------ | ------------------------------------------ |
| `$match`     | Filters documents                          |
| `$group`     | Groups by fields and computes aggregations |
| `$sort`      | Sorts documents                            |
| `$project`   | Reshapes documents                         |
| `$limit`     | Limits the number of documents             |
| `$skip`      | Skips specified number of docs             |
| `$unwind`    | Deconstructs array fields                  |
| `$lookup`    | Joins documents from other collections     |
| `$addFields` | Adds new fields                            |

### Example: Lookup with Join

```js
db.orders.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "userId",
      foreignField: "_id",
      as: "user",
    },
  },
  { $unwind: "$user" },
  { $project: { _id: 0, orderId: 1, userName: "$user.name" } },
]);
```

---

## 🔹 Mongoose `.populate()`

### Usage

If you define a Mongoose schema like this:

```js
const TaskSchema = new mongoose.Schema({
  description: String,
  owner: { type: mongoose.Schema.Types.ObjectId, ref: "User" },
});
```

You can populate the `owner` field like so:

```js
Task.find().populate("owner");
```

### Custom Options

```js
Task.find().populate({
  path: "owner",
  select: "name email",
  match: { age: { $gte: 18 } },
  options: { limit: 5, sort: { name: 1 } },
});
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

- `_id` is automatically indexed and required.
- Use `ObjectId("...")` when querying by `_id`.
- Use indexes to speed up large queries.
- Use `$project` and `.select()` to limit returned fields.
- Prefer `.populate()` in Mongoose for cleaner code over `$lookup`.
- Avoid deeply nested documents unless necessary.
