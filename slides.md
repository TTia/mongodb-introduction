### First slide

- Splash

---
### Introduction
https://docs.mongodb.com/manual/introduction/

- Document database
- Key features (High performance, High availability, Horizontal Scalability)

---
### Key concepts

- Database
- Collections
- Documents (16Mb)
- Views

---
### CRUD examples

---
### Schema validation

https://docs.mongodb.com/manual/core/schema-validation/index.html

- validation rules
- validation level

---

### Schema

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

