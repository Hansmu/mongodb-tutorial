# Exploring create

Remember, there are three methods:
* insertOne
* insertMany
* insert - should not be used

Now, when you're doing `inserMany`, there are a bunch of concerns.

When you're inserting many elements using the `insertMany` method, then you'll see a certain type of behaviour which is called an "Ordered Insert".

What happens here is that it will try inserting each element in the array one by one, until it hits an error or finishes successfully. If it gets an error, then it will stop inserting any further, but the previous entries will have been committed to the DB.

```bash
db.hobbies.insertMany([
    {_id: "yoga", name: "Yoga"}, # <-- Inserted successfully
    {_id: "football", name: "Football"}, # <-- Inserted successfully
    {_id: "football", name: "Football"}, # <-- Failed due to duplicate
    {_id: "day-drinking", name: "Day drinking"}, # <-- Ignored because of the failure
])
```

When you want to override the default behaviour, then you specify that with the second parameter. Setting ordered to false will not break on the first failure, but it will run until the end and insert anything that it can in the list.

```bash
db.hobbies.insertMany([
    {_id: "yoga", name: "Yoga"}, # <-- Inserted successfully
    {_id: "yoga", name: "Yoga"}, # <-- Failed due to duplicate
    {_id: "football", name: "Football"}, # <-- Inserted successfully
    {_id: "football", name: "Football"}, # <-- Failed due to duplicate
    {_id: "day-drinking", name: "Day drinking"}, # <-- Inserted successfully
], {
    ordered: false
})
```

## Write Concern

When you're writing to MongoDB, then you can request a different level of write confirmation.

So in Mongo, you do your write operation, and get a confirmation back from the server that it has succeeded or failed.

What matters here is that succeed can mean different things based on your confirmation config.

There are three things worth mentioning about the storage engine data storage:
* Memory
* Journal
* Data on Disk

Accessing these three things all take a different amount of time.
* Memory is obviously the fastest.
* Journal is a simple kind of "todo" file, telling the server that these kinds of things need to be saved on disk.
* Saving on the disk requires finding a location, recalculations etc. So this is the heaviest operation.

By default, Mongo gives you a confirmation once your data is in memory.

There is an issue here - the DB can go down because of an event. Maybe a power loss. If data hasn't managed to be written from memory before then, then that means the data will be lost.

If the data is entered into the journal, then once a reboot happens, then the server will continue writing the data onto disk.

So you have trade-offs here between speed and reliability.

Journaling still happens by default, but depending on your config, you can get a confirmation before journaling has happened.

In a perfectly timed situation then, you could get a success, but still find yourself with data loss.

You can configure these options with the second parameter for the insert commands.
```bash
{
    w: 1, # <-- The number of instances a write confirmation should be received from before acknowledging the write as successful
    j: true, # <-- Whether the write operation should wait for a journal write before returning the write operation as successful
    wtimeout: 200 # <-- Write timeout, how long should the wait be before it's considered a failure
}
```

You can even set the write confirmation to 0 instances, which means you instantly get a response, but the acknowledgement is false, because no one acknowledged it.

## Atomicity

Mongo guarantees that your operation is atomic. It succeeds as a whole or fails as a whole. A document would not be written down partially.

Atomicity assurance is given on a per document level.

## Importing data

For importing data, you need `mongoimport`.

When importing, you need to specify whether it's a single document or an array as well.

```bash
mongoimport <name-of-the-file> -d <name-of-the-database> -c <collection-name-where-to-import>
```

Data example is in the `data` folder.

```bash
mongoimport tv-shows.json -d moveData -c movies --jsonArray
```

If you want to do a fresh import, then you can append the `--drop` flag to the command to drop the collection and re-add with the imported data.