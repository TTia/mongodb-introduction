### Introduction

[MongoDB Reference](https://docs.mongodb.com/manual/introduction/)

Topics:
- Brief introduction to MongoDB as NoSQL db
- CRUD examples and uses
- Schema definition
- Key features (High performance, High availability, Horizontal Scalability)
- (Timeseries implementation - Part 2?)

---

### Key concepts

- Database
- Collections
- Documents (16Mb)
- Views

===

### BSON

BSON is a binary representation of JSON documents, though it contains more data types than JSON. 

*Reference [BSON](http://bsonspec.org/)*

#### ObjectIds
ObjectIds are small, likely unique, fast to generate, and *ordered*. ObjectId values consist of 12 bytes, where the first 4 bytes are a timestamp that reflect the ObjectId’s creation.

#### 

---

### CRUD examples

![Insert One](images\crud\crud-annotated-mongodb-insertOne.bakedsvg.svg)
<!--
db.users.insertOne({
    name: "sue",
    age: 26,
    status: "pending"
})
-->

```
{
    "acknowledged" : true,
    "insertedId" : ObjectId("5d43228ae3a64c3d95b061b2")
}
```

The unique `_id` field that acts as a primary key, by leaving it empty will automatically generated.
Eventually, you can specify your own `_id` values.

- Performing a write against a non-existing collection will create a new collection, with "_id" as PK.
- All write operations are atomic on the level of a single document.

*Reference: https://docs.mongodb.com/manual/tutorial/insert-documents/ *  
*Write concern - Acknoledge in multi-node cluster: https://docs.mongodb.com/manual/reference/write-concern/ *

===
### Find documents

![Find / Select](images\crud\crud-annotated-mongodb-find.bakedsvg.svg)

```
{ "_id" : ObjectId("5d43228ae3a64c3d95b061b2"), "name" : "sue" }
// Note that we are projecting "address" that doesn't exists.
```

<!--
db.users.find(
      { age: {$gt: 18} },
      { name: 1, address: 1}
).limit(5)
-->

**$elemMatch**
```
{ "_id" : 1, "semester" : 1, "grades" : [ 70, 87, 90 ] }
{ "_id" : 2, "semester" : 1, "grades" : [ 90, 88, 92 ] }
...
{ "_id" : 6, "semester" : 2, "grades" : [ 95, 90, 96 ] }

$ db.students.find( { semester: 1, $elemMatch: { grades: { $gte: 85 } } }, { "grades.$": 1 } )
// If you specify a single query predicate in the $elemMatch expression, $elemMatch is not necessary.
{ "_id" : 1, "grades" : [ 87 ] }
{ "_id" : 3, "grades" : [ 85 ] }
```

In general, through *find* and *projection* we are maintaning the document schema. Arrays of documents could be queryied by **array.field** selector. The query processor will applied the operator based on the field type.

*Array operators: https://docs.mongodb.com/manual/reference/operator/query-array/*  
*TX implementation - Error handling and retries: https://docs.mongodb.com/manual/core/transactions/*

===

### Schema Definition - Embedding

![Denormalised schema](images\schema\data-model-denormalized.bakedsvg.svg)

- you have one-to-one relationshipts (UML strong composition);
- you have one-to-many relationships between entities;
- you have a sub-entity that changes in time, by embedding you're taking a "snapshot";
- keep your query in mind, as embedding will provide better performances.

===

### Schema Definition (2) - Normalised Data

![Normalised schema](images\schema\data-model-normalized.bakedsvg.svg)

- when embedding would result in duplication of data but without a performance gain;
- to represent more complex many-to-many relationships;
- to model large hierarchical data sets;
- join, aka lookup reference: https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/#pipe._S_lookup

===

### Schema validation

```
db.runCommand( {
   collMod: "contacts",
   validator: { $jsonSchema: {
      bsonType: "object",
      required: [ "phone", "name" ],
      properties: {
         phone: { bsonType: "string", description: "must be a string and is required" },
         name: { bsonType: "string", description: "must be a string and is required" }
      }
   } },
   validationLevel: "moderate"
})
```

**Strict validation** - Applied to all inserts and updates.

**Moderate validation** - The validation rules are applied only to inserts and to updates to existing documents that already fulfill the validation criteria. With the moderate level, updates to existing documents that do not fulfill the validation criteria are not checked for validity.

Validation is applied in a lazy fashion, for disruptive changes, a migration is the best option.

*Validation reference: https://docs.mongodb.com/manual/core/schema-validation/index.html*

===

### CRUD examples (2)

Update
Upsert for timeseries?

---

### Aggregation pipeline

```json
{ "cust_id": "A123", "amount": 500, "status": "A" },
{ "cust_id": "A123", "amount": 250, "status": "A" },
{ "cust_id": "B212", "amount": 200, "status": "A" },
{ "cust_id": "A123", "amount": 300, "status": "D" }
```

Query for:
- Given order in status "A"
- Calculate the amount per customer

Translates in SQL in:

```sql
select O.cust_id, SUM(amount)
from orders O
where O.status = 'A'
group by O.cust_id
```

===

### Aggregation pipeline

```js
db.orders.aggregate([
   { $match: { status: "A" } },
   { $project: { _id: false, amount: true, cust_id: true } },
   { $group: { _id: "$cust_id", total: { $sum: "$amount" } } }
])
```

**First Stage:** The $match stage filters the documents by the status field and passes to the next stage those documents that have status equal to "A".

**Second Stage:** The $group stage groups the documents by the cust_id field to calculate the sum of the amount for each unique cust_id.

<video style="width:60%;" 
      src="https://docs.mongodb.com/manual/_images/agg-pipeline.mp4" 
      poster="images/mongodb_logo.png" 
      controls> </video>

===

### Aggregation pipeline optimization

#### Indexing
- background
- unique
- sparse, hash, geo, tree
- geo

---
### Timeseries implementation

https://www.mongodb.com/blog/post/time-series-data-and-mongodb-part-1-introduction
https://www.mongodb.com/blog/post/time-series-data-and-mongodb-part-2-schema-design-best-practices
https://www.mongodb.com/blog/post/time-series-data-and-mongodb-part-3--querying-analyzing-and-presenting-timeseries-data

---
### Timeseries implementation (2)

- Java SDK (vs moongose?)
- Statement implementation

---
### Good to know

#### 4.0
- Profiler
- Capped collection
- Retention
- Subscription

#### 4.2 (RC)
- Materialized views with merge strategies
- Wildcard indexes
- Update aggregation (dynamic validation)

```
$ docker run -d --name mongodb-rc3-bionic \
      -p "27017:27017" \
      mongo:4.2.0-rc3-bionic
# or      
$ docker run -d --name mongodb-rc3-bionic \
      -p "27017:27017" \
      mongo
      
$ docker exec -it ta-fleet-mongodb mongo admin
$ db.createUser({user:'mongodb', pwd:'mongodb', roles:[{role:'userAdminAnyDatabase',db:'admin'}] });
$ exit
$ mongo
```

---
### References

* ...

