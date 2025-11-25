# MongoDB + mongo-express

**What it's for:** MongoDB is a **document (NoSQL)** database that stores flexible,
schema-less JSON-like documents (BSON). It's a good fit for rapidly evolving data
models, nested/hierarchical data, catalogs, event logs, and content where a rigid
relational schema would get in the way.

**mongo-express** is a lightweight web-based admin UI for browsing databases,
collections, and documents.

| Service        | Image             | Port   | Access                  |
|----------------|-------------------|--------|-------------------------|
| mongodb        | mongo:7           | 27017  | `localhost:27017`       |
| mongo-express  | mongo-express:1   | 8081   | http://localhost:8081   |

Credentials come from the root `.env` (`MONGO_*`, `ME_CONFIG_*`).
Default root user: **`admin`** / **`Mongo_Pass123!`** (auth DB: `admin`).

---

## Run (from the project root)

```bash
docker compose up -d mongodb mongo-express
docker compose logs -f mongodb
docker compose down            # stop; add -v to delete data
```

> To run from inside this folder instead: `docker compose --env-file ../.env up -d`

## Quick access via Docker

```bash
# Enter the mongo shell
docker exec -it mongodb mongosh -u admin -p '<password>' --authenticationDatabase admin

# Run a single command without entering the shell
docker exec -it mongodb mongosh \
  -u admin -p '<password>' \
  --authenticationDatabase admin \
  --eval "show dbs"
```

---

## Connection & auth (inside mongosh)

```js
// Switch database
use admin
use appdb

// Authenticate manually (if you connected without credentials)
db.auth("admin", "<password>")

// Current database
db

// Current connection / auth status
db.runCommand({ connectionStatus: 1 })
```

## Create an application auth user

```js
db.getSiblingDB("luminate-development").createUser({
  user: "<username>",
  pwd: "<password>",
  roles: [ { role: "readWrite", db: "<database-name>" } ]
})
```

## User management

```js
db.getUsers()                                    // list users in current DB
db.createUser({
  user: "myuser",
  pwd: "mypassword",
  roles: [{ role: "readWrite", db: "appdb" }]
})
db.changeUserPassword("myuser", "newpassword")
db.dropUser("myuser")
```

## Database & collection management

```js
show dbs                          // list databases
show collections                  // list collections in current DB
db.createCollection("mycollection")
db.mycollection.drop()            // drop a collection
db.dropDatabase()                 // drop the current database
```

## CRUD operations

```js
// Insert
db.mycollection.insertOne({ name: "Alice", age: 30 })
db.mycollection.insertMany([{ name: "Bob" }, { name: "Charlie" }])

// Find
db.mycollection.find()
db.mycollection.find({ name: "Alice" })
db.mycollection.findOne({ name: "Alice" })

// Update
db.mycollection.updateOne({ name: "Alice" }, { $set: { age: 31 } })
db.mycollection.updateMany({ age: 30 }, { $set: { active: true } })

// Delete
db.mycollection.deleteOne({ name: "Alice" })
db.mycollection.deleteMany({ active: false })
```

## Indexes

```js
db.mycollection.getIndexes()
db.mycollection.createIndex({ name: 1 })
db.mycollection.createIndex({ email: 1 }, { unique: true })
db.mycollection.dropIndex("name_1")
```

## Stats & admin

```js
db.stats()                          // database stats
db.mycollection.stats()             // collection stats
db.serverStatus()                   // server status
db.adminCommand('ping')             // health ping
db.adminCommand({ usersInfo: 1 })   // users in the current DB
```

---

## Backup & restore

```bash
# Dump the whole server to ./dump on the host
docker exec -t mongodb mongodump -u admin -p '<password>' --authenticationDatabase admin --out /tmp/dump
docker cp mongodb:/tmp/dump ./dump

# Restore a dump directory back into MongoDB
docker cp ./dump mongodb:/tmp/dump
docker exec -t mongodb mongorestore -u admin -p '<password>' --authenticationDatabase admin /tmp/dump
```
