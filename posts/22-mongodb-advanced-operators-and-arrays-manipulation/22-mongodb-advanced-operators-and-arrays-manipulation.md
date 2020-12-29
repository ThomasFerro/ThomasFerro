# MongoDB query operators and arrays manipulation (M001 - Part 3)

> When exact matching is not enough.

We already covered the basic CRUD operations [in the previous article](https://dev.to/thomasferro/a-pragmatic-approach-to-documents-manipulation-in-mongodb-m001-part-2-1mcf). Before wrapping this section up, I would like to introduce more advanced concepts that are often used.

## Comparing data

Until now, we almost exclusively search for documents by fields that should be strictly matching the provided query. This is far from the only type of comparison we use day to day.

> **Query operators**: Tools to locale the requested data in a database.

All of the operators we will use begin with the `$` sign.

For instance, here is how to use a query operator: `{ <field>: { <operator>: <value> } }`.

Here are some common operators to use:

- `$eq`: "*Equal to*", for exact matching;
- `$ne`: "*Not equal to*", for exclusion;
- `$gt`: Match if the requested value is "*greater than*" the value provided in the query;
- `$gte`: Match if the requested value is "*greater than or equal to*" the value provided in the query;
- `$lt`: Match if the requested value is "*less than*" the value provided in the query;
- `$lte`: Match if the requested value is "*less than or equal to*" the value provided in the query;

For instance, searching for a city with *more than 10.000 citizens*  can look like this:

```js
db.cities.find({ "citizens": { "$gt": 10000 } })
```

Looking for cities *outside the USA* can be done using this type of query:

```js
db.cities.find({ "country": { "$ne": "USA" } })
```

Wait, what if I need to find the cities with *more than 10.000 citizens*, but *outside of the USA* at the same time? We will need to chain our statements to do so.

## Chaining statements

Please do not stop reading, I promise that I am not going to hit you hard with the theory of *logic gates*.

You may find yourself in a situation where you want documents matching several statements. Sometimes you may need to match all of the statements, sometime at least one of them, or even none.

This is a common need in software development and it is also common when querying a database.

For instance, we can represent the rules behind a gas station serving a customer with the following statement:

```js
const stationCanServeGas = customerPaymentAuthorized() && thereIsGasLeft()
```

MongoDB provides the tools to achieve the same goal using a different syntax and *logic operators*.

> **Logic operators**: Tools to chain statement with specific behaviours

- [`$and`](https://docs.mongodb.com/manual/reference/operator/query/and/): The documents must match *all* of the provided statements;
- [`$or`](https://docs.mongodb.com/manual/reference/operator/query/or/): The documents must match *at least one* of the statements;
- [`$nor`](https://docs.mongodb.com/manual/reference/operator/query/nor/): The documents must match *none* of the statements;
- [`$not`](https://docs.mongodb.com/manual/reference/operator/query/not/): Negates the provided expression.

The first three operators `$and`, `$or` and `$nor` have a similar syntax with an array of statements:

```
{ <operator>: [ { statement1 }, { statement2 }, ... ] }
```

The `$not` operator is completely different, please read through [the documentation](https://docs.mongodb.com/manual/reference/operator/query/not/) for more information about its behavior and syntax.

Going back to the first operators, we can for instance find every valid users by making a query with every needed matching:

```js
db.users.find({
    "$and": [{
        "$or": [{
            "registrationCompleted": true
        }, {
            "ongoingRegistration": true
        }]
    }, {
        "ongoingDeletion": false
    }]
})
```

The `$and` operator is implicit by default, meaning that you can ignore it in order to simplify your requests. Here is the previous request using the implicit `$and`:

```js
db.users.find({
    "$or": [{
        "registrationCompleted": true
    }, {
        "ongoingRegistration": true
    }],
    "$not": { "ongoingDeletion": true }
})
```

We can go even further with the implicit `$and` when doing multiple statements on the same field. Here are two ways of returning courses that take between 5 and 10 hours to complete:

```js
db.courses.find({
    "$and": [{
        "estimateTimeToComplete": {
            "$gt": 5
        }
    }, {
        "estimateTimeToComplete": {
            "$lt": 10
        }
    }]
})

db.courses.find({
    "estimateTimeToComplete": {
        "$gt": 5,
        "$lt": 10
    }
})
```

As you can see, there are several ways to achieve the same goal. Some are more verbose or readable than others. It is up to you to make them maintainable and easy to understand!

## Comparing fields within a document with the expressive query operator

> **[Expressive query operator](https://docs.mongodb.com/manual/reference/operator/query/expr/)**: Adds expressiveness to the query language by allowing us to use *variables* and *conditions* statements. It also allows for the use of aggregation expressions, discussed in a next article.

Here is the basic syntax of this type of operator:

```
{ "$expr": { <statement> } }
```

We can use the `$` symbol inside of an expressive query operator to **reference a document's field value**.

Let us make this concept more concrete by seeing it in action. Here is how we can try to find cities that currently welcome more tourists than citizens:

```js
db.cities.find({
    "$expr": {
        "$gt": ["$population", "$tourists"]
    }
})
```

Notice the change in the syntax, we will cover it in-depth when talking about the aggregation pipeline.

## Querying and manipulating array fields

The MQL syntax offers many operators to query and manipulate arrays. We already used the `$push` operator in the previous article, here is a distilled explanation of it:

> **[`$push`](https://docs.mongodb.com/manual/reference/operator/update/push/)**: Adds one or many elements to an array or turn the field into an array if it was of a different type.

Here is how we can add a student in a course:

```js
db.courses.updateMany({
    "subject": "Mongodb"
}, {
    "$push": {
        "students": "Eliot Horowitz"
    }
})
```

---

To search for a document based on elements in an array, we can use several approaches:

- When looking in an array with one provided element, no matter if there are other elements:
```js
db.<collection>.find({ <the_array>: <one_of_the_element> })
```
- When looking in an array with the exact list of elements provided:
```js
db.<collection>.find({ <the_array>: [<the_first_element>, <the_second_element>, <...>] })
```
- When looking in an array with the provided list of elements in any order:
```js
db.<collection>.find({ <the_array>: { "$all": [<the_first_element>, <the_second_element>, <...>] } })
```

We also have the possibility to search for an array with a specified length using the `$size` operator. For instance, looking for courses with 10 students and with Eliot Horowitz, Kevin Ryan and Dwight Merriman could look like this:

```js
db.courses.find({
    "students": {
        "$size": 10,
        "$all": [
            "Eliot Horowitz",
            "Kevin Ryan",
            "Dwight Merriman",
        ]
    }
})
```

---

One of the advanced tools for array querying is *$elemMatch*.

> **[$elemMatch](https://docs.mongodb.com/manual/reference/operator/query/elemMatch)**: Returns every document with the array field containing at least one element matching the query. 

Let us say that our *students* in the *courses* collection are now complex objects with their name, age and other useful information in this context. For legal purpose, we want to find courses with at least one student under 18. We can achieve that with the following query:

```js
db.courses.find({
    "students": {
        "$elemMatch": {
            "age": {
                "$lte": 18
            }
        }
    }
})
```

--- 

Another useful tool provided by the *MQL* allows us to search for a specific element in an array based on his index.

Say for instance that we want to get every courses where the first student is named "Merriman". We could achieve this goal using the following syntax:

```js
db.courses.find({
    "students.0.name": "Merriman"
})
```

## Leaner find results with projections

We can decide which fields to return in a query by specifying a *projection*.

```js
db.<collection>.find({ <query> }, { <projection> })
```

With the projection being a set of key-value pairs with the value being **1 if we want the field to be included** or **0 if we want it excluded**. Note that you cannot mix zeros and ones in a single projection, except for the "_id" field that can be excluded in a projection with included fields.

For instance, if we only want the students and the subject of a course, we can define the following projection:

```js
db.courses.find({}, {
    "subject": 1,
    "students": 1
})
```

Combining projections and query is also possible. Here is how we can use our previous query, but only return the subject of the matching courses:

```js
db.courses.find({
    "students": {
        "$size": 10,
        "$all": [
            "Eliot Horowitz",
            "Kevin Ryan",
            "Dwight Merriman",
        ]
    }
}, {
    "subject": 1,
})
```

---

This article wraps the first part of the series regarding the MongoDB certification. It has been a long yet interesting journey! I hope that you are as eager as I am to cover basic cluster administration, the aggregation framework and the last subjects of the course 😁
