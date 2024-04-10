# Commands

You can use the MongoDB commandline to interact with it.

Some different commands are listed here.

### See existing DBs
To see the existing DBs, you can use:
```bash
show dbs
```

### To select a DB
To select a DB, you can use:
```bash
use <db-name>
```

For example:
```bash
use shop
```

If a DB of this name does not exist, then it will be created.

### Operate on a collection
To operate on a collection, you can use:
```bash
db.<collection-name>.<operation>
```

`db` is a keyword and it refers to the currently selected database.

Same as with selecting a DB, if the collection does not exist, then it'll be created.

An example command would be:
```bash
db.products.insertOne({ name: "Banana potassifier", price: 13.37 })
```

### Insert a document into a collection
To insert a document into a collection, you can use:
```bash
db.<collection-name>.insertOne({ ...data here })
```

For example:
```bash
db.products.insertOne({ name: "Banana potassifier", price: 13.37 })
```

When you run the insert, you're going to get back an object, with an acknowledgement, and an ID of the inserted object.

### To query a collection
In order to query a collection, you can use the `find` method.

In order to get all the results in a single collection, you can use the method with no parameters provided:
```bash
db.<collection-name>.find()
```

For example:
```bash
db.products.find()
```

You can append `.pretty()` to pretty print the JSON.
```bash
db.products.find().pretty()
```