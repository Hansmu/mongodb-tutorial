# Transactions

Transactions help you run all DB operations together as a single unit. If one fails, they all fail.

For a transaction, we need a session.

```js
const session = db.getMongo().startSession();
session.startTransaction();

// Note that you're getting a reference to the collection off of the session.
const usersCollection = session.getDatabase('blog').users;
const postsCollection = session.getDatabase('blog').posts;

usersCollection.deleteOne({ _id: ObjectId('...') });
postsCollection.deleteMany({ userId: ObjectId('...') });

// If you do a find operation outside of the session, then you'd see that the entry is still in the DB.

session.commitTransaction();
```

Since it takes extra resources, then use it only when you need it.