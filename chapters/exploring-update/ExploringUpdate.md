# Exploring update

## Update
`$set` can be used to update multiple fields.

```js
db.users.updateOne(
    {_id: ObjectId("...")},
    {$set: {age: 40, phoneNumber: 51243124}}
);
```

Incrementing and decrementing values can be done using the `$inc` operator.

```js
db.users.updateOne(
    { name: 'Steve'},
    {
        $inc: {
            age: 1, // Increment
            shoes: -1 // Decrement
        }
    }
);
```

If you have two operations working on the same field, then the query will fail.

You can use `$min` and `$max` to change the value comparatively.

```js
// If the stored value is less than 111, then nothing will get modified.
db["users"].updateOne({ name: "Steve" }, { $min: { age: 111 } });

// If the stored value is greater than 10, then nothing will change.
db["users"].updateOne({ name: "Steve" }, { $max: { age: 10 } });
```

To multiply the stored value, you can use `$mul`.

```js
// Will multiply shoes by 2.
db["users"].updateOne({ name: "Steve" }, { $mul: { shoes: 2 } });
```

To remove a field, you can use `$unset`.

```js
db["users"].updateOne(
    { name: "Steve" },
    { $unset: {
        // The value for the field does not matter.
        // Can be whatever. Falsy values don't matter.
        testingField: 0
    }}
)
```

`$rename` can be used to change a field name.

The key is the existing field, and the string value is the new name.

```js
db["users"].updateOne({ name: "Steve" }, { $rename: { shoes: 'pairsOfShoes' } });
```

## Upsert

To update or insert in one operation, you can use the `upsert` option when updating.

Also, note that the filter that's being provided there will be added to the object.

```js

db["users"].updateOne({ name: "Maria" }, { $set: { age: 23, pairsOfShoes: 10 } }, { upsert: true });

// Results in:

{
    "_id": ObjectId("688f3bd643a0f282af824a14"),
    "name": "Maria",
    "age": 23,
    "pairsOfShoes": 10
}
```

## Arrays

If you find something by an array value, and then you want to modify those found values, you can use the dollar sign operator.

However, note that this only refers to the first match in the array.

If multiple values match, then it only updates the first.

```js
db.users.updateMany(
    {
        hobbies: {
            $elemMatch: {
                title: 'Sports',
                frequency: {
                    $gte: 3
                }
            }
        }
    },
    {
        $set: {
            'hobbies.$.highFrequency': true
        }
    }
);
```

If you want to update all array elements that an entity has, you can use the `$[]` operator.

Also, the property must exist before applying this.

```js
db.users.updateMany(
    {
        age: {
            $gt: 25
        }
    },
    {
        $inc: {
            'hobbies.$[].frequency': 1
        }
    }
);
```

If you want to update all matching nested array elements, then you can use `$[someVariable]`.

The way this works is that the update section says on which data the variable has a reference.

And the `arrayFilters` says which values the variable refers to.

```js
db.users.updateMany(
    {
        age: {
            $gt: 25
        }
    },
    {
        // el will refer to elements of hobbies.
        $set: {
            'hobbies.$[el].goodFrequency': true
        }
    },
    {
        arrayFilters: [
            {
                // el will only use values that satisfy this condition.
                'el.frequency': { $gt: 2 }
            }
        ]
    }
);
```

To add elements to an array, you can use `$push`.

```js
db.users.updateOne(
    { name: 'Maria' },
    { $push: {
        hobbies: {
            title: 'Reading',
            frequency: 4
        }
    }}
)
```

If you want to add multiple, you can add `$each` to it.

```js
db.users.updateOne(
    { name: 'Maria' },
    { $push: {
        hobbies: {
            $each: [
                { title: 'Watching', frequency: 2 },
                { title: 'Waiting', frequency: 4 },
                { title: 'Commiserating', frequency: 1 }
            ],
            // You can also add a sort in case it's dynamic data
            $sort: {
                frequency: -1
            }
        }
    }}
)
```

`$pull` can be used to remove elements from an array.

You can use all the filter operators inside pull.

```js
db.users.updateOne(
    { name: 'Maria' },
    {
        $pull: {
            hobbies: {
                title: 'Commiserating'
            }
        }
    }
)
```

`$pop` can be used to remove the first or last element.

```js
db.users.updateOne(
    { name: 'Maria' },
    {
        $pop: {
            // To remove the first, set it to -1
            // To remove the last, set it to 1
            hobbies: 1
        }
    }
)
```

If you want to have unique values only, then you can use `$addToSet` to push and it'll not push if the value is already present.

```js
db.users.updateOne(
    { name: 'Maria' },
    { $addToSet: {
        hobbies: {
            title: 'Reading',
            frequency: 4
        }
    }}
)
```