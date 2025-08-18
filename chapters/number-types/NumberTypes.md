# Number types

## Different number types

The main types that exist are integers, longs, and doubles.

Important thing to note that you should always pass the number in quotation marks if you're casting.

The reason is that the default numbers JS works with are double64s. If you do `NumberLong(9223372036854775807)`, this'll mean that JS will overflow as it'll try to use a double64.

If you add it as a string, then you don't have that issue.

When you're doing any sort of calculations with the number, then that calculation should also use the casting method.
Otherwise, it might change the type.

```js
db.persons.insertOne({ age: NumberInt("29") });
db.persons.updateOne({}, { $inc: { age: NumberInt("1") } });
```

### Integers - int32 

This takes only full numbers.

The range that it's in will be -2,147,483,648 to 2,147,483,647.

Use it for "normal" integers.

If you want to save a value as an int, then you can wrap it in `NumberInt`.

You won't run into an issue when inserting as a number here, but for long you will.

```js
db.persons.insertOne({ age: NumberInt(29) });
```

### Longs - int64 

This only takes full numbers.

The value range is 9,223,372,036,854,775,807 to 9,223,372,036,854,775,807.

Use it for large integers.

If you want to save a value as an int, then you can wrap it in `NumberLong`.

Remember to always pass the number in as a string here. Otherwise, if you reach the max value, JS itself will overflow.

```js
db.companies.insertOne({ valuation: NumberLong("29") });
```

### Doubles

There are two types of doubles - 64 bit and 128 bit.

#### 64 bit

The default type used by the MongoDB shell regardless if it's an integer or not.
This is because it's written in JS. If you're using a different programming language, then the default might be different.

It's for numbers with decimal places.

The decimal values are approximated, they're not guaranteed.

Use for floats where high precision is not required.

#### 128 bit

These are high precision decimals.

Use for floats where high precision is required (monetary or science calculations for example).

```js
db.companies.insertOne({ availableFunds: NumberDecimal("0.3") });
```