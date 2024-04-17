# Structuring documents

MongoDB is schemaless by default.

But that does not mean that you can't use some kind of schema.

## Defining a schema

When deciding what to do with your data, you can have:
* Completely different documents in one collection
* Some required shared fields, and then some extra data outside of the schema
* Full equality, every document has all the same fields

## Data types

There are a couple of different data types that exist in MongoDB.
* Text - string value. There are no limits to this, only the 16 MB limit on the whole document.
* Boolean - true/false
* Number - there are multiple number types
  * Int32 - 32 bit number
  * Long - 64 bit number
  * Decimal128 - 128 bit high precision floating point number
* ObjectId - special type for the generated ID
* ISODate - date 
* Timestamp - used internally
* Embedded document
* Array

To create a value of a certain type, then you have to use its constructor provided by MongoDB. Ex. `someNum: NumberDecimal("12.99")`

When using Mongo from a programming language, then you have to import the constructors from the driver library.

To create dates and timestamps, you have to use the Mongo built-in methods and constructors.

When you're setting a value equal to a number, then the Mongo shell, by default, interprets it as a double.

When you're inserting a number that is too large, then the end can be cut off or nullified (last numbers turned to 0s).

## Schema validation

You can define two properties when doing schema validation:
* validationLevel - which documents get validated?
  * strict - all inserts & updates
  * moderate - inserts & updates to specific documents
* validationAction - what happens if validation fails?
  * error - throw error and deny insert/update
  * warn - log warning but proceed

To add validation, then you'll need to use the `createCollection` command.

Inside that, there is the second parameter. It contains a bunch of different options for configuring. Among them for the validation there is the `validationLevel`, `validationAction`, and `validator` keys.

You can specify the schema under `validator` keyword. Inside that, you can specify which type of validation definition you want to use. It is recommended to use `$jsonSchema`, as others have been deprecated.

In the specific sections, you define the type by using the `bsonType` property. A description of the field can be provided with `description`.

If it's an `opbject`, then you can additionally provide the `required` property to define what fields are required. The `properties` key will allow you to define the field types and descriptions.

```bash
db.createCollection("posts", {
    validationLevel: "strict",
    validationAction: "error",
    validator: {
        $jsonSchema: {
            bsonType: "object",
            required: [
                'title',
                'text',
                'creator',
                'comments'
            ],
            properties: {
                title: {
                    bsonType: 'string',
                    description: 'title of the post'
                },
                text: {
                    bsonType: 'string',
                    description: 'content of the post'
                },
                creator: {
                    bsonType: 'objectId',
                    description: 'post creator user reference'
                },
                comments: {
                    bsonType: 'array',
                    description: 'comments on the post',
                    items: {
                        bsonType: 'object',
                        required: [
                            'text',
                            'author'
                        ],
                        properties: {
                            text: {
                                bsonType: 'string',
                                description: 'content of the comment'
                            },
                            text: {
                                bsonType: 'objectId',
                                description: 'comment creator user reference'
                            }
                        }
                    }
                }
            }
        }
    } 
})
```

In order to modify the validation rules, then the `db.runCommand` can be used.

The `runCommand` method can be used for a whole bunch of different things. But the following example is for modifying the collection rules. The first property can be either a document or a string. We specify what we're doing with that property.

`collMod` specifies that we want to modify a collection or view. On the same level, we can then specify all sorts of things.

```
db.runCommand({
    collMod: "nameOfYourCollection",
    validator: {...},
    validationLevel: "moderate"
})
```

## Designing relations

You have two options:
* Nested - another document is nested in the current document
* References - reference another document

| Criteria                                                | Example                                            | Defines                                                                                                   |
|---------------------------------------------------------|----------------------------------------------------|-----------------------------------------------------------------------------------------------------------|
| What data does my app need or generate?                 | User information, product information, orders etc. | Defines the fields you'll need (and a little bit how they relate)                                         |
| Where do I need my data?                                | Welcome page, products list page, orders page etc. | Defines your required collections (ex. products collection for the products page) + field groupings       |
| Which kind of data or information do I want to display? | Welcome page: Product names                        | Defines which queries you'll need (you'll want to avoid joins)                                            |
| How often do I fetch my data?                           | For every page reload                              | Defines whether you should optimize for easy fetching                                                     |
| How often do I write or change my data?                 | Orders => Often, product data => rarely            | Defines whether you should optimize for easy writing  (often changing things would be better centralized) |

Some example considerations:
* An app has cities and its citizens. You would never want to query a city with all of its citizens. Thus, they would always be separated into different documents.
* To avoid joining, you can go for duplication. For duplication, you need to consider how often the data changes and what properties. Maybe you only need to duplicate the description, which rarely changes.
* Keep the 16 MB limit for the document in mind. Embed too much and you might just hit it.

## Joining

In order to join your documents, you can use the `$lookup` keyword. Allows doing a join in one go, rather than querying twice.

It uses the `aggregate` method, which will be described more in a later chapter.

```bash
db.sourceCollection.aggregate(
    [
        {
            $lookup: {
                from: "collectionWeWantToJoinTo", 
                localField: "fieldInTheSourceCollectionThatHoldsTheReference",
                foreignField: "referenceInTheCollectionWeWantToJoinTo",
                as: "aliasUnderWhichTheJoinedFieldsCanBeSeen"
            }
        }
    ]
)
```

```bash
db.books.aggregate(
    [
        {
            $lookup: {
                from: "authors", 
                localField: "bookAuthors",
                foreignField: "_id",
                as: "creators"
            }
        }
    ]
)
```

```js
{
    "_id": 1,
    "name": "Some book",
    "authors": [
        123,
        456
    ],
    "creators": [
        {
            "_id": 123,
            "name": "Banana lord",
            "age": 137
        },
        {
            "_id": 256,
            "name": "Dog the Frog",
            "age": 56
        }
    ]
}
```