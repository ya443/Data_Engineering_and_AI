# Introduction to MongoDB

> A beginner-friendly guide to getting started with MongoDB — from core concepts to querying, validation, and data modelling.

---

## Table of Contents

1. [What is MongoDB?](#what-is-mongodb)
2. [Advantages and Disadvantages](#advantages-and-disadvantages)
3. [Common Use Cases](#common-use-cases)
4. [Connecting to MongoDB with Compass](#connecting-to-mongodb-with-compass)
5. [Creating a Database and Collection](#creating-a-database-and-collection)
6. [Adding Documents](#adding-documents)
7. [Validation](#validation)
8. [Searching, Updating and Deleting Documents](#searching-updating-and-deleting-documents)
9. [Embedding vs Referencing](#embedding-vs-referencing)

---

## What is MongoDB?

MongoDB is a **document-oriented NoSQL database** that stores data as JSON-like documents (called BSON — Binary JSON). Unlike traditional relational databases (e.g. PostgreSQL, MySQL), MongoDB does not require a fixed schema, giving you the flexibility to store differently-shaped data in the same collection.

Key characteristics:

- Data is stored as **documents** (similar to JSON objects)
- Documents are grouped into **collections** (similar to tables in SQL)
- Collections live inside a **database**
- No rigid schema required — fields can vary between documents

```json
{
  "_id": "ObjectId('64f1a2b3c4d5e6f7a8b9c0d1')",
  "name": "Yas Akilakulasingam",
  "course": "Data Engineering",
  "skills": ["Python", "SQL", "MongoDB"]
}
```

---

## Advantages and Disadvantages

###  Advantages

- **Document Oriented Storage** — data is stored naturally as JSON-like objects, making it intuitive for developers
- **Easy Horizontal Scaling** — MongoDB can scale out across many servers (sharding) with minimal configuration
- **Fast and Efficient** — optimised for read/write-heavy workloads
- **Open Source** — publicly available with an active community:
  - Source code can be viewed, edited, and distributed freely
  - Free to use at scale — no per-seat licensing fees
- **Flexible Schema** — documents in the same collection can have different fields, ideal for evolving data models

###  Disadvantages

- **High Memory Usage** — documents can duplicate data (denormalisation), leading to larger storage footprints
- **Can be Inconsistent** — by default, MongoDB favours availability over strict consistency (eventual consistency)
- **Limited Transaction Support** — while multi-document ACID transactions are now supported (v4.0+), they are more complex to use compared to relational databases

---

## Common Use Cases

MongoDB excels in scenarios where data is varied, fast-moving, or document-shaped:

-  **Social Media posts/data** — user-generated content with varying structure
-  **API data (JSON)** — store API responses directly without transformation
-  **Mobile App Backends** — flexible schema suits rapidly changing mobile apps
-  **Caching** — e.g. shopping cart data that changes frequently
-  **Product catalogues** — products with different attributes per category
-  **Media files/data** — metadata for images, videos, audio
-  **CMS systems** — content management with varied page structures
-  **CRM systems** — customer records with heavy text and varied contact data
-  **IoT and Sensor data** — high-volume, time-series data from devices
-  **Logs and monitoring data** — application and server event streams
- **Gaming data** — player profiles, game state, leaderboards
-  **Chat systems/apps** — messages, threads, user metadata
- ...and much more

---

## Connecting to MongoDB with Compass

**MongoDB Compass** is the official GUI tool for MongoDB. It allows you to visually manage databases, collections, and documents without writing shell commands.

### Steps to Connect

1. Download and install [MongoDB Compass](https://www.mongodb.com/products/compass)
2. Open Compass and click **"New Connection"**
3. Enter your connection string. For a local MongoDB instance:

```
mongodb://localhost:27017
```

4. Click **"Connect"**

You should now see your local MongoDB server with any existing databases listed in the left-hand panel.

> If you haven't installed MongoDB locally, download it from [mongodb.com/try/download/community](https://www.mongodb.com/try/download/community)

---

## Creating a Database and Collection

### Using MongoShell (mongosh)

By default, `mongosh` will open connected to the `test` database. You can see which database is active at any time using:

```mongosh
db
```

To **create and switch to a new database**, use the `use` command:

```mongosh
use sparta
```

> Once you run this, the prompt changes from `test>` to `sparta>`

To **create a new collection** within that database:

```mongosh
db.createCollection("institute")
```

Expected output:

```json
{ ok: 1 }
```

### Using Compass

1. In the left panel, click **"+"** next to "Databases"
2. Enter a database name (e.g. `sparta`) and a collection name (e.g. `institute`)
3. Click **"Create Database"**

---

## Adding Documents

### Insert a Single Document

```mongosh
db.institute.insertOne({ 
  name: "Yas Akilakulasingam", 
  course: "Data Engineering", education: { masters: "MSc Financial Mathematics with Data Science, University of Bath (2025-2026)", bachelors: "BSc Mathematical Sciences, Upper Second Class Honours, Gold Scholar, University of Bath (2021-2025)" }, 
  skills: ["Python", "RStudio", "SQL", "Power BI", "Excel", "Machine Learning", "MongoDB"], 
  languages: ["English", "Tamil"]
})
```

Expected output:

```json
{
  acknowledged: true,
  insertedId: ObjectId('6a1d8ebf3527ae8ae1221cfb')
}
```

### Insert Multiple Documents

Use `insertMany` to add several documents in one command:

```mongosh
db.institute.insertMany([
    { "course": "Data Engineering" },
    { "course": "Data Analysis" }
])
```

Expected output:

```json
{
  acknowledged: true,
  insertedIds: {
    '0': ObjectId('6a1d90263527ae8ae1221cfc'),
    '1': ObjectId('6a1d90263527ae8ae1221cfd')
  }
}
```

> **Bonus:** The legacy `insert()` command also exists and can accept either a single document or an array (like `insertOne` and `insertMany` combined). However, it is **deprecated** as of MongoDB 5.0 — prefer `insertOne` or `insertMany` instead.

---

## Validation

MongoDB allows you to enforce rules on the shape and types of documents inserted into a collection using **schema validation**.

### Creating a Validated Collection — `students`

You can apply validation rules using `$jsonSchema` when creating a collection:

```mongosh
db.createCollection("students", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "age", "course"],
      properties: {
        name: {
          bsonType: "string",
          description: "must be a string and is required"
        },
        age: {
          bsonType: "int",
          minimum: 16,
          description: "must be an integer >= 16 and is required"
        },
        course: {
          bsonType: "string",
          enum: ["Data Engineering", "Data Analysis", "Cloud Computing"],
          description: "must be one of the allowed courses and is required"
        }
      }
    }
  }
})
```

###  Invalid Entry

Attempting to insert a document that violates the validation rules:

```mongosh
db.students.insertOne({
    name: "John Smith",
    age: 14,
    course: "Data Engineering"
})
```

Output:

```
MongoServerError: Document failed validation
Additional information: {
  failingDocumentId: ObjectId('...'),
  details: {
    operatorName: '$jsonSchema',
    schemaRulesNotSatisfied: [
      {
        operatorName: 'properties',
        propertiesNotSatisfied: [
          { propertyName: 'age', description: 'must be an integer >= 16 and is required' }
        ]
      }
    ]
  }
}
```

> The document was **rejected** because `age: 14` is below the minimum of 16.

###  Valid Entry

```mongosh
db.students.insertOne({
    name: "John Smith",
    age: 22,
    course: "Data Engineering"
})
```

Output:

```json
{
  acknowledged: true,
  insertedId: ObjectId('...')
}
```

> The document was **accepted** because all validation rules were satisfied.

---

## Searching, Updating and Deleting Documents

### Creating a Films Collection with Validation

```mongosh
db.createCollection("films", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["title", "year", "genre", "rating"],
      properties: {
        title:  { bsonType: "string" },
        year:   { bsonType: "int", minimum: 1888 },
        genre:  { bsonType: "string" },
        rating: { bsonType: "double", minimum: 0, maximum: 10 }
      }
    }
  }
})
```

### Inserting Film Documents

Using `insertOne`:

```mongosh
db.films.insertOne({
    title: "Interstellar",
    year: 2014,
    genre: "Sci-Fi",
    rating: 8.7,
    director: "Christopher Nolan"
})
```

Using `insertMany`:

```mongosh
db.films.insertMany([
    { title: "The Dark Knight",   year: 2008, genre: "Action",  rating: 9.0, director: "Christopher Nolan" },
    { title: "Inception",         year: 2010, genre: "Sci-Fi",  rating: 8.8, director: "Christopher Nolan" },
    { title: "Parasite",          year: 2019, genre: "Thriller",rating: 8.5, director: "Bong Joon-ho" },
    { title: "Everything Everywhere All at Once", year: 2022, genre: "Sci-Fi", rating: 7.8, director: "Daniel Kwan" }
])
```

### Searching for Documents

Find **all** documents in a collection:

```mongosh
db.films.find()
```

Find documents matching a condition:

```mongosh
db.films.find({ genre: "Sci-Fi" })
```

Find documents with a rating above 8.5:

```mongosh
db.films.find({ rating: { $gt: 8.5 } })
```

Find one document:

```mongosh
db.films.findOne({ title: "Parasite" })
```

### Updating Documents

Update a **single** document:

```mongosh
db.films.updateOne(
    { title: "Interstellar" },
    { $set: { rating: 9.0 } }
)
```

Update **multiple** documents at once:

```mongosh
db.films.updateMany(
    { genre: "Sci-Fi" },
    { $set: { genre: "Science Fiction" } }
)
```

Expected output for `updateMany`:

```json
{
  acknowledged: true,
  matchedCount: 3,
  modifiedCount: 3
}
```

### Deleting Documents

Delete a **single** document:

```mongosh
db.films.deleteOne({ title: "Everything Everywhere All at Once" })
```

Delete **multiple** documents matching a filter:

```mongosh
db.films.deleteMany({ genre: "Science Fiction" })
```

>  **Warning:** `deleteMany({})` with an empty filter will delete **all** documents in the collection.

---

## Embedding vs Referencing

When modelling data in MongoDB, there are two main strategies for handling relationships between data: **Embedding** and **Referencing**.

### Embedding

Embedding means storing related data **directly inside** a parent document as a nested sub-document or array.

```json
{
  "_id": "ObjectId('...')",
  "name": "Yas Akilakulasingam",
  "address": {
    "street": "123 Bath Road",
    "city": "Bath",
    "postcode": "BA1 1AA"
  },
  "courses": [
    { "title": "Data Engineering", "grade": "Distinction" },
    { "title": "Cloud Computing",  "grade": "Merit" }
  ]
}
```

**When to use embedding:**

- Data is always accessed together (e.g. a user and their address)
- The nested data belongs exclusively to the parent (one-to-one or one-to-few)
- You want to avoid extra queries — all data comes back in a single read
- The embedded data is unlikely to grow without bound

**Advantages:**

-  Fast reads — single query returns everything
-  Atomic writes — the whole document updates in one operation
-  Simpler application code

**Disadvantages:**

-  Document size limit of 16MB in MongoDB
-  Data duplication if the same sub-document is shared across many parents

---

### Referencing

Referencing means storing related data in **separate collections** and linking them using an `ObjectId` (similar to a foreign key in SQL).

**Parent document (students collection):**

```json
{
  "_id": "ObjectId('abc123')",
  "name": "Yas Akilakulasingam",
  "course_id": "ObjectId('def456')"
}
```

**Related document (courses collection):**

```json
{
  "_id": "ObjectId('def456')",
  "title": "Data Engineering",
  "duration_weeks": 12,
  "instructor": "Joe Bloggs"
}
```

**When to use referencing:**

- Data is shared across many documents (e.g. many students on the same course)
- The related dataset is large or grows unboundedly (e.g. order history)
- You want to avoid data duplication
- You need to update the related data in one place only

**Advantages:**

- No data duplication — the course record exists once
-  Easier to update shared data
-  Avoids hitting the 16MB document size limit

**Disadvantages:**

-  Requires multiple queries or a `$lookup` aggregation to join data
-  Slightly more complex application code

---

### Embedding vs Referencing — Summary

| | Embedding | Referencing |
|---|---|---|
| **Query complexity** | Simple (single query) | More complex (`$lookup` needed) |
| **Data duplication** | Possible | Avoided |
| **Update ease** | Updates one document | Update in one place, reflects everywhere |
| **Best for** | One-to-few relationships | One-to-many / many-to-many |
| **Performance** | Faster reads | Can be slower (multiple queries) |

>  **Rule of thumb:** If you find yourself always querying two collections together, consider embedding. If the related data is shared or grows large, use referencing.

