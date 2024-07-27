# MongoDB Aggregation Operations:
***(ref: Mongodb official documentation & Hitesh Choudhary YT)***

Aggregation operations process multiple documents and return computed results. You can use aggregation operations to:

- Group values from multiple documents together.

- Perform operations on the grouped data to return a single result.

- Analyze data changes over time.


## Aggregation Pipelines
An aggregation pipeline consists of ***one or more stages*** that process documents:

- **Each stage performs an operation on the input documents.** (Ex. a stage can filter documents, group documents, and calculate values.)

- **The documents that are output from a stage are passed to the next stage.**

- **An aggregation pipeline can return results for groups of documents.** (Ex. return the total, average, maximum, and minimum values.)

```js
db.order.aggregate( [
   // Stage 1: Filter pizza order documents by pizza size
   {
      $match: { size: "medium" }
   },
   // Stage 2: Group remaining documents by pizza name and calculate total quantity
   {
      $group: { _id: "$name", totalQuantity: { $sum: "$quantity" } }
   }
] )
```

> **Note:** $match, $count, $group... etc. are mongodb operators.

## Data on MongoDB
```js
// users: 
{
    _id: Number,
    name: String,
    index: Number
    age: Number,
    isActive: Boolean
    eyeColor: String,
    favoriteFruite: String,
    gender: String,
    registered: Date,
    company: {
        email: String,
        phone: String,
        title: String,
        location: {
            address: String,
            country: String
        }
    },
    tags: String[]
}


//books
{
    _id: Number,
    author_id: Number,
    genre: String,
    title: String
}

//authors
{
    _id: Number,
    birth_year: Number,
    name: String
}
```




### Q.1 Ex. Count number users who are active
```js
db.users.aggregate([
    {
        $match: {
            isActive: true
        }
    },
    {
        $count: "activeUsers"
    }
])

// OUTPUT: {activeUsers: 516}
```


### Q.2 Ex. Calculate average age of all users
>**NOTE:** ***$group*** : basically this groups into different documents base on given 1st field. Ex. _id in below example.

```js
[
    {
        $group: {
            _id: "$gender",   // we group by this field
            averageAge: {   // By accumulator we perform operation on these groups
                $avg: "$age"
            }
        }
    }
]

// OUTPUT:
[
    {
        _id: female,
        averageAge: 29.8102
    },
    {
        _id: male,
        averageAge: 29.6272
    }
]


// If we provide null then it consider only one group-- single document

[
    {
        $group: {
            _id: null,
            averageAge: {
                $avg: "$age"
            }
        }
    }
]

// OUTPUT: 
{
  _id: null,
  averageAge: 29.835
}
```

>**NOTE:** "$" by you can access collection field



### Q.3 Ex. List the top 5 most common favorite fruits among the users

```js
[
  { // group by based on favoriteFruit
    $group: {
        _id: "$favoriteFruit",
        count: { // Along with count no. of users whose favourite fruit is $favoriteFruit
            $sum: 1 // whenever a founds it adds 1. this way $sum works
        }
    }
  },
  { // sort based on field count, -1:descending, 1:ascending
    $sort: {
        count: -1
    }
  },
  { // take only top two documents, simi., $limit: 5 for top 5
    $limit: 2
  }
]
```



## Stages

1. **$group:** The ***$group*** stage separates documents into groups according to a "group key". The output is one document for each unique group key.
    ```js
    {
        $group:
        {
            _id: <expression>, //Group key
            <field1>: { 
                <accumulator1> : <expression1> 
            },
            ...
        }
    }
    ```

2. **$match:** Filters the documents to pass only the documents that match the specified condition(s) to the next pipeline stage.
    ```js
    {
        $match: { <query> }
    }
    ```

3. **$limit:** Limits the number of documents passed to the next stage in the pipeline.
    ```js
    {
        $limit: <positive 64-bit number>
    }
    ```

4. **$count:** Passes a document to the next stage that contains a count of the number of documents input to the stage.
    ```js
    {
        $count: <string>
    }
    ```

5. **$sort:** Sorts all input documents and returns them to the pipeline in sorted order.
    ```js
    {
        $sort: <value> 
        //-1: descending
        // 1: ascending
    }
    ```

6. **$skip:** Skips over the specified number of documents that pass into the stage and passes the remaining documents to the next stage in the pipeline.
    ```js
    {
        $skip: <positive 64-bit number>
    }
    ```



## Operators

1. **$avg:**: Returns the average value of the numeric values. $avg ignores non-numeric values.
    ```js
    { $avg: <expression> }
    ```

2. **$:**

