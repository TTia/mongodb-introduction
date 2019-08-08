## MongoDB Introduction
#### Plus a timeseries implementation

[Mattia Barrasso](https://www.linkedin.com/in/mattiabarrasso/)  
*mattia.barrasso@flairbit.io*  
<small>08/08/2019</small>

![asd](images/mongodb_logo.png)


---

### Introduction to MongoDB

##### Topics:
- Key features
- CRUD examples and uses
- Aggregation pipeline
- Schema definition
- (Timeseries implementation)

[MongoDB Reference](https://docs.mongodb.com/manual/introduction/)

===

#### Main characteristics
- NoSQL Database  
- Data is stored as "Documents", composed by key-value pairs, key-array or nested documents  
- Dynamic schema  
- Horizontal sharding & Strong consistency  
- Querying by object-oriented APIs

![Architecture](http://www.ibmbpnetwork.com/hubfs/mongodb_architecture.png)

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

```js
{
    "acknowledged" : true,
    "insertedId" : ObjectId("5d43228ae3a64c3d95b061b2")
}
```

The unique `_id` field that acts as a primary key, by leaving it empty will automatically generated.
Eventually, you can specify your own `_id` values.

All write operations are atomic on the level of a single document.

*Reference: https://docs.mongodb.com/manual/tutorial/insert-documents/ *  
*Write concern - Acknowledge in multi-node cluster: https://docs.mongodb.com/manual/reference/write-concern/ *

===

### BSON

BSON is a binary representation of JSON documents, though it contains more data types than JSON. 

*Reference [BSON](http://bsonspec.org/)*

#### ObjectIds

```js
{
    "acknowledged" : true,
    "insertedId" : ObjectId("5d43228ae3a64c3d95b061b2")
}
```

ObjectIds are:

- small
- likely unique
- fast to generate
- and *ordered*.  

ObjectId values consist of 12 bytes, where the first 4 bytes are a timestamp that reflect the ObjectId’s creation.

===

### Find documents

![Find / Select](images\crud\crud-annotated-mongodb-find.bakedsvg.svg)

```js
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
```js
{ "_id" : 1, "semester" : 1, "grades" : [ 70, 87, 90 ] }
{ "_id" : 2, "semester" : 1, "grades" : [ 90, 88, 92 ] }
...
{ "_id" : 6, "semester" : 2, "grades" : [ 95, 90, 96 ] }

$ db.students.find( { semester: 1, $elemMatch: { grades: { $gte: 85 } } }, { "grades.$": 1 } )
// If you specify a single query predicate in the $elemMatch expression, $elemMatch is not necessary.
{ "_id" : 1, "grades" : [ 87 ] }
{ "_id" : 3, "grades" : [ 85 ] }
```

Through *find* and *projection* we are maintaining the document schema.

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

```js
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

### CRUD examples - Update

```js
{
  _id: 1,
  item: "TBD",
  stock: 0,
  info: { publisher: "1111", pages: 430 },
  tags: [ "technology", "computer" ],
  ratings: [ { by: "ijk", rating: 4 }, { by: "lmn", rating: 5 } ],
  reorder: false
}
```

```js
db.books.update(
   { _id: 1 },    // find a book-stock document with a given _id
   {
     $inc: { stock: 5 },      // increase the available #stock
     $set: {                  // set fields values
       item: "ABC123",
       "info.publisher": "2222",    // access to an embedded obj
       tags: [ "software" ],
       "ratings.1": { by: "xyz", rating: 3 }    // access to an array element
     }
   }
)
```

===

### CRUD examples - Update

```js
db.books.update(
   { _id: 1 },    // find a book-stock document with a given _id
   {
     $inc: { stock: 5 },      // increase the available #stock
     $set: {                  // set fields values
       item: "ABC123",
       "info.publisher": "2222",    // access to an embedded obj
       tags: [ "software" ],
       "ratings.1": { by: "xyz", rating: 3 }    // access to an array element
     }
   }
)
```

Outcome:

```json
{
  "_id" : 1,
  "item" : "ABC123",
  "stock" : 5,
  "info" : { "publisher" : "2222", "pages" : 430 },
  "tags" : [ "software" ],
  "ratings" : [ { "by" : "ijk", "rating" : 4 }, { "by" : "xyz", "rating" : 3 } ],
  "reorder" : false
}
```

---

### Timeseries implementation

```
[{ val: 50, time: 1535530412},
   { val: 55, time : 1535530415},
   { val: 56, time: 1535530420},
   { val: 55, time : 1535530430},
   { val: 56, time: 1535530432}]
```

#### One entry per document

```
{
  "_id" : ObjectId("5b4690e047f49a04be523cbd"),
  "p" : 56.56,
  "symbol" : "MDB",
  "d" : ISODate("2018-06-30T00:00:01Z")
},
{
  "_id" : ObjectId("5b4690e047f49a04be523cbe"),
  "p" : 56.58,
  "symbol" : "MDB",
  "d" : ISODate("2018-06-30T00:00:02Z")
}
```

- High insertion rate 👎🏻 - An update cost way less than an insert
- Large indexes 👎🏻
- Low abstraction level 👍🏻

===

### Time-based bucketing of one document

#### Per day

```
sample = {val: 59, time: 1535530450}
day = ISODate("2018-08-29")
db.iot.updateOne({deviceid: 1234, sensorid: 3, nsamples: {$lt: 200}, day: day},
                 {$setOnInsert: { deviceid: 1234 },
                  $setOnInsert: { sensorid: 3 },
                  $setOnInsert: { day: day },

                  $push: {samples: sample},
                  $min: {first: sample.time},
                  $max: {last: sample.time},
                  $inc: {nsamples: 1}},
                  {upsert: true} )
```

- Reduce by 200 (i.e.) the index dimension 👍🏻
- Flexible, buckets could be adjusted based on the transmission frequency 👍🏻
- Reducing memory and data storage 👍🏻

===

### Time-based bucketing of one document - Take 2

#### Per minute

The keys of `p` represent the timestamp (i.e. second) of the measure.

```
{
      "_id" : ObjectId("5b57a8fae303d36d6df69cd3"),
      "p" : {
              	"0" : 58.75,
              	"1" : 58.75,
              	"2" : 59.45,
                  ...
              	"58" : 58.57,
              	"59" : 59.01
      },
      "symbol" : "FB",
      "d" : ISODate("2018-06-30T00:01:00Z")
}
```

- Strong preconditions 👎🏻
- Complex aggregation queries 👎🏻

===

### Time-based bucketing of one document per day 
#### Our take in Flairkit

```
db.canmessages.updateOne(
{
  resourceId: resourceId,
  capabilityId: capabilityId,
  timestamp: bucketTs // per day
},
{
  $setOnInsert: { resourceId: resourceId },
  $min: { tstart: ISODate(ts) },
  $max: { tend: ISODate(ts) },
  $addToSet: {
    	values: can_message
  },
  $inc: { samplesCount: 1 }
}, {
    upsert: true
});
```

*References:*
- Blog posts by MongoDB ([Part 1](https://www.mongodb.com/blog/post/time-series-data-and-mongodb-part-1-introduction), [Part 2](https://www.mongodb.com/blog/post/time-series-data-and-mongodb-part-2-schema-design-best-practices), [Part 3](https://www.mongodb.com/blog/post/time-series-data-and-mongodb-part-3--querying-analyzing-and-presenting-timeseries-data))
- [Paper by MongoDB](https://www.mongodb.com/collateral/time-series-best-practices)

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

#### Thumb rules

- use **$match** to limit the cardinality as first step
- **$sort** after `match` phases, before map the original schema to a different format
- **$project** only the needed fields
- **$limit** the output

<img src="https://i-love-png.com/images/green-thumbs-up.png" style="width: 20%; border: none; box-shadow: none;">

===

### Aggregation pipeline optimization

#### Indexing

Create a secondary index to support you query.

```js
db.orders.find(...).explain(verbosity)
db.orders.aggregate([...]).explain(verbosity)
```

Verbosity options are: 
- `queryPlanner` ~ calculate the best query plan, dry run
- `executionStats` ~ calculate and execute the best query plan
-  and `allPlansExecution` ~ calculate and execute the best query plan, but provides statistics on the other candidates (`.hint()`)

**Indexes characteristics**:
- background generation
- unique - will trigger validation error while inserting duplicates
- sparse, hash, geo, tree
- partial matching is supported by the query planner

---
### Experiment!

#### 4.0

```bash
$ docker run -d --name mongodb -p "27017:27017" mongo
```

#### 4.2 (RC)

```bash
$ docker run -d --name mongodb-rc3-bionic -p "27017:27017" mongo-rc3-bionic
```

##### Create the root user

```bash
$ docker exec -it ta-fleet-mongodb mongo admin
$ db.createUser({user:'mongodb', pwd:'mongodb', roles:[{role:'userAdminAnyDatabase',db:'admin'}] });
$ exit
$ mongo
```
#### Client

**[NoSQL Booster for MongoBD](https://nosqlbooster.com/features)** (30d trial mode)

`brew cask install nosqlbooster-for-mongodb`

**[Robo 3T](https://robomongo.org/)** (free & lightweight)

`brew cask install robo-3t`