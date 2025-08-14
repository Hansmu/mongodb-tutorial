# Aggregation

Aggregation runs a pipeline on your collection to transform your data into a form that you need.

```js
db.persons.insertMany([
    {
        name: "Alice Johnson",
        age: 29,
        gender: "female",
        email: "alice.j@example.com",
        address: {
            city: "New York",
            country: "USA"
        },
        hobbies: ["reading", "cycling", "music"],
        scores: [78, 85, 92],
        isActive: true,
        createdAt: ISODate("2023-10-01T10:00:00Z"),
        location: {
            type: "Point",
            coordinates: [-74.006, 40.7128] // New York
        }
    },
    {
        name: "Bob Smith",
        age: 34,
        gender: "male",
        email: "bob.s@example.com",
        address: {
            city: "London",
            country: "UK"
        },
        hobbies: ["football", "travel", "photography"],
        scores: [88, 91, 77],
        isActive: false,
        createdAt: ISODate("2024-01-15T14:23:00Z"),
        location: {
            type: "Point",
            coordinates: [-0.1276, 51.5074] // London
        }
    },
    {
        name: "Carla Mendes",
        age: 42,
        gender: "female",
        email: "carla.m@example.com",
        address: {
            city: "São Paulo",
            country: "Brazil"
        },
        hobbies: ["cooking", "yoga"],
        scores: [90, 87, 84],
        isActive: true,
        createdAt: ISODate("2024-06-20T09:45:00Z"),
        location: {
            type: "Point",
            coordinates: [-46.6333, -23.5505] // São Paulo
        }
    },
    {
        name: "Daniel Wei",
        age: 25,
        gender: "male",
        email: "daniel.w@example.com",
        address: {
            city: "Shanghai",
            country: "China"
        },
        hobbies: ["gaming", "cycling", "tech"],
        scores: [60, 70, 65],
        isActive: true,
        createdAt: ISODate("2025-02-11T19:10:00Z"),
        location: {
            type: "Point",
            coordinates: [121.4737, 31.2304] // Shanghai
        }
    },
    {
        name: "Eva Müller",
        age: 31,
        gender: "female",
        email: "eva.m@example.com",
        address: {
            city: "Berlin",
            country: "Germany"
        },
        hobbies: ["reading", "baking", "hiking"],
        scores: [95, 90, 98],
        isActive: false,
        createdAt: ISODate("2025-06-01T08:30:00Z"),
        location: {
            type: "Point",
            coordinates: [13.4050, 52.5200] // Berlin
        }
    }, {
        name: "Steve Fobs",
        age: 31,
        gender: "male",
        email: "steview.m@example.com",
        address: {
            city: "Münich",
            country: "Germany"
        },
        hobbies: ["reading", "baking", "hiking", 'pottery'],
        scores: [95, 90, 98],
        isActive: false,
        createdAt: ISODate("2025-06-01T08:30:00Z"),
        location: {
            type: "Point",
            coordinates: [13.4050, 52.5200] // Berlin
        }
    }
]);
```

The aggregate operation takes in an array, as we're running a series of steps.

It takes advantage of any defined indexes.

Each stage has access to the data that the previous stage passes.

```js
db.persons.aggregate([
    {
        // $match is to filter the data
        $match: {
            gender: 'female'
        }
    },
    {
        // Allows you to group your data
        $group: {
            // The _id is a keyword for group
            // You specify what fields you want to group by
            // If it's a single field, you don't need an object
            // If it's multiple, then you do
            _id: {
                // The dollar sign tells Mongo that this references a field from the document
                someRandomKey: '$age'
            },
            // Add 1 for each entry in the group
            totalPersons: {$sum: 1}
        }
    },
    {
        $sort: {
            // From the previous $group stage
            totalPersons: 1
        }
    }
]);
```

To use projection, you can use `$project`.
You can select fields and add new fields if needed.

Note that you can access values using `$variableName`.

When you need to convert data, you can use `$convert`.
There are also shorthand ones like `$toString`.
However, you can't configure it as much.

```js
db.persons.aggregate([
    {
        $project: {
            _id: 0,
            gender: 1,
            createdAt: 1,
            createdAtString: {
                $convert: {
                    input: '$createdAt',
                    to: 'string',
                    onError: '-',
                    onNull: 'missing'
                }
            },
            createdAtStringShorterSyntax: {
                $toString: '$createdAt'
            },
            longitude: {
                // Or $slice
                $arrayElemAt: ['$location.coordinates', 0]
            },
            latitude: {
                $arrayElemAt: ['$location.coordinates', 1]
            },
            fullName: {
                $concat: [
                    {$toUpper: "$gender"},
                    " - ",
                    {$toUpper: "$name"}
                ]
            }
        }
    },
    {
        $group: {
            _id: {
                createdAtYear: {
                    $isoWeekYear: '$createdAt'
                }
            },
            numPersons: {
                $sum: 1
            }
        }
    }
])
```

Remember, a `$group` is many to one, and `$project` is 1:1.

You can also do a lot of operations with arrays in aggregation pipelines.

`$unwind` can be used to pull the elements out of an array.

* If you use `$unwind: "$fieldName"`, then it'll create duplicate documents, each of which has a single element of the
  array.
    * Example - { name: 'Steve', hobbies: 'Cooking' }, { name: 'Steve', hobbies: 'Pottery' } etc.

If you want to avoid duplicate values, then you can use `$addToSet` instead of `$push`.

```js
db.persons.aggregate([
    {
        $unwind: '$hobbies'
    },
    {
        $group: {
            _id: '$age',
            allHobbies: {
                $push: '$hobbies'
            },
            allUniqueHobbies: {
                $addToSet: '$hobbies'
            }
        }
    }
])
```

You can use `$filter` to filter out elements of an array.

```js
db.persons.aggregate([
    {
        $project: {
            _id: 0,
            scoresAbove80: {
                $filter: {
                    input: '$scores',
                    as: 'sc', // Local variable when filtering
                    cond: {
                        // Notice that it has $$, not $
                        // If it'd have $, then it'd look for a field
                        // $$ references a local variable
                        $gt: ['$$sc', 60]
                    }
                }
            }
        }
    }
])
```

To put your data in buckets for analysis, you can use `$bucket`.

```js
db.persons.aggregate([
    {
        $bucket: {
            groupBy: '$age',
            boundaries: [0, 10, 20, 30, 40, 50, 60],
            output: {
                names: {
                    $push: "$name"
                },
                averageAge: {
                    $avg: "$age"
                },
                numPersons: {
                    $sum: 1
                }
            }
        }
    }
])
```

If you don't want to specify the boundaries on your own and just get it to distribute the data, then `$bucketAuto` can
be used.

Mongo tries to distribute equally.

```js
db.persons.aggregate([
    {
        $bucketAuto: {
            groupBy: '$age',
            buckets: 5,
            output: {
                names: {
                    $push: "$name"
                },
                averageAge: {
                    $avg: "$age"
                },
                numPersons: {
                    $sum: 1
                }
            }
        }
    }
])
```

To paginate, you can add `$skip` and `$limit` stages

```js
db.persons.aggregate([
    {
        $project: {
            name: 1
        },
    },
    {
        $skip: 10
    },
    {
        $limit: 10
    }
])
```

Mongo applies a bunch of optimizations to aggregation pipelines:

https://www.mongodb.com/docs/manual/core/aggregation-pipeline-optimization/

If you want to funnel your results straight into a new collection, then you could use `$out`.

```js
db.persons.aggregate([
    {
        $project: {
            name: 1
        },
    },
    {$out: 'someCollection'}
])
```

The issue with `$out` is that it overwrites the collection.

However, if you'd like to add to a collection or update the documents in it, then `$merge` could be used.

```js
db.orders.aggregate([
  { $match: { status: "pending" } },
  { $merge: {
      into: "pendingOrders",
      on: "_id",
      whenMatched: "merge",
      whenNotMatched: "insert"
  } }
])
```

You can do geometry queries using `$geoNear`.

Since only the first step has access to the collection, then this has to be first.

```js
db.persons.aggregate([
    {
        // Has to be the first step
        $geoNear: {
            // Specify the geometry you're trying to find data close to
            near: {
                type: 'Point',
                coordinates: [13.404, 52.523]
            },
            // Maximum distance from the point in meters
            maxDistance: 10000,
            // Since you can't run a match first
            // Then you can define a query here
            query: { 
                gender: 'female'
            },
            // $geoNear will give us back the distance from the point
            // So we can specify where it's saved
            distanceField: 'distanceFromOurLittlePoint'
        }
    },
])
```
