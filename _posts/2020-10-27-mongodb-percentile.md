---
title: 'Creating Percentile Table with a Specified Increment in MongoDB'
date: 2020-10-27
permalink: /posts/2020/10/percentile-with-specified-increment-in-mongodb/
tags:
  - mongodb
  - percentile
---

Here's the scenario.

- We'd like to create a collection (a term of table in MongoDB) that has several columns, such as the N-th percentile, the percentile's value, and the number of observations below the percentile's value.
- There's a dashboard filter that allows users to input the percentile increment e.g. an increment of 25 means that the percentiles will be 25, 50, 75, and 100.
- When the filter is applied, the percentile collection should be updated (re-computed) in accordance with the inputted increment

How'd we achieve this via MongoDB's native query?

I performed a quick investigation on MongoDB's aggregation framework as well as its operators, such as map, range, multiply, push, arrayElemAt, and so on. In general, here's what I coded to accomplish the above task.

Please note that in my case, I'd like to look for the percentile's values for request time in nginx access log.

<b>NB:</b> `((increment))` is a filter variable inputted by the user.

```
[
    {
        "$sort" : { request_time : 1 }
    },
    {
        "$group": {
            "_id": null,
            "list_of_req_time": {
                "$push": "$request_time"
            }
        }
    },
    {
        "$addFields": {
            "percentile": { 
                "$range": [ ((increment)), 101, ((increment)) ] 
            }
        }  
    },
    {
        "$project": {
            "_id": 0,
            "percentile": 1,
            "max_request_time": { 
               "$map": {
                    "input": "$percentile",
                    "as": "percentile_value",
                    "in": { 
                        "$arrayElemAt": [
                            "$list_of_req_time",
                            {
                                "$subtract": [ 
                                    {
                                        "$floor": {
                                            "$multiply": [
                                                {
                                                    $divide: ["$$percentile_value", 100]
                                                }, {"$size": "$list_of_req_time"}
                                            ]
                                        }
                                    } , 1 
                                ]
                            }
                        ]
                    }
                }
            },
            "request_count": {
                "$map": {
                    "input": "$percentile",
                    "as": "percentile_value",
                    "in": {
                        "$floor": {
                            "$multiply": [
                                {
                                    $divide: ["$$percentile_value", 100]
                                }, {"$size": "$list_of_req_time"}
                            ]
                        }
                    }
                }
            }
        }
    },
    { 
        "$unwind": { 
            "path": "$percentile", 
            "includeArrayIndex" : "percentile_idx" 
        } 
    },
    { 
        "$unwind": { 
            "path": "$max_request_time", 
            "includeArrayIndex" : "max_request_time_idx" 
            
        }
    },
    { 
        "$unwind": { 
            "path": "$request_count", 
            "includeArrayIndex" : "request_count_idx" 
            
        }
    },
    { 
        "$project": {
            "_id" : 0,
            "percentile": 1,
            "max_request_time": 1,
            "request_count": 1,
            "valid_m": { 
                "$eq": ["$percentile_idx", "$max_request_time_idx"] 
            },
            "valid_n": { 
                "$eq": ["$max_request_time_idx", "$request_count_idx"] 
            } 
        }
    },
    { 
        "$match": { 
            "$and": [ {"valid_m": true}, {"valid_n": true} ]
        }
    },
    {
        "$project": {
            "valid_m": 0,
            "valid_n": 0
        }
    }
]
```

Let's break down the above query.

## 1st step: Sort the request time in ascending order

The following query sorts the request time column from the smallest to the biggest value.

This is needed since computing the percentile values needs the data to be sorted in ascending order first.

```
{
    "$sort" : { request_time : 1 }
}
```

## 2nd step: Create a new collection that stores a list of request times

We achieve this by simply setting the `group by` with `null`. This means that MongoDB will apply an aggregation on the whole table (not just certain categories).

For instance, in the following query we perform an aggregation that push all the request times to an array called `list_of_req_time`.

```
{
    "$group": {
        "_id": null,
        "list_of_req_time": {
            "$push": "$request_time"
        }
    }
}
```

We might imagine the query results in the following. Note that the request times in the list are already sorted.

```
_id |               list_of_req_time
-----------------------------------------------------
0   | [<req_time_a>, <req_time_b>, <req_time_c>, ...]
```

## 3rd step: Add a new column storing a list of N-th percentiles

The following query simply adds a new column to the previous aggregated collection.

The new column stores the list of N-th percentile in which the `{{increment}}` variable is inputted by the user.

For instance, if `{{increment}}` is set to 25, then we'd get quartiles, that is `percentile = [25, 50, 75, 100]`.

We're going to iterate on this list to create the percentile collection.

```
{
    "$addFields": {
        "percentile": { 
            "$range": [ ((increment)), 101, ((increment)) ] 
        }
    }  
}
```

## 4th step: Compute the percentile values & number of observations below the percentile value

Take a look at the following query.

We declare `$project` simply to select or discard certain columns. In the below query, we discard `_id` column (since we set it to 0) and select `percentile` (from step #3), `max_request_time` (percentile values), and `request_count` (number of observations below the percentile value). 

```
{
    "$project": {
        "_id": 0,
        "percentile": 1,
        "max_request_time": { 
           "$map": {
                "input": "$percentile",
                "as": "percentile_value",
                "in": { 
                    "$arrayElemAt": [
                        "$list_of_req_time",
                        {
                            "$subtract": [ 
                                {
                                    "$floor": {
                                        "$multiply": [
                                            {
                                                $divide: ["$$percentile_value", 100]
                                            }, {"$size": "$list_of_req_time"}
                                        ]
                                    }
                                } , 1 
                            ]
                        }
                    ]
                }
            }
        },
        "request_count": {
            "$map": {
                "input": "$percentile",
                "as": "percentile_value",
                "in": {
                    "$floor": {
                        "$multiply": [
                            {
                                $divide: ["$$percentile_value", 100]
                            }, {"$size": "$list_of_req_time"}
                        ]
                    }
                }
            }
        }
    }
}
```

Let's take a look at how to compute the `max_request_time`.

```
"max_request_time": { 
   "$map": {
        "input": "$percentile",
        "as": "percentile_value",
        "in": { 
            "$arrayElemAt": [
                "$list_of_req_time",
                {
                    "$subtract": [ 
                        {
                            "$floor": {
                                "$multiply": [
                                    {
                                        $divide: ["$$percentile_value", 100]
                                    }, {"$size": "$list_of_req_time"}
                                ]
                            }
                        } , 1 
                    ]
                }
            ]
        }
    }
}
```

The basic procedure goes like the followings.
- Iterates on `percentile` list created at step #3 (stores the N-th percentile)
- Represents each value in the `percentile` list as `percentile_value`. This is similar to `val` in `for val in my_arr`
- For each `percentile_value`, we divide it by 100 and multiply the result by the length of our list of request time (`list_of_req_time`). Since such an operation might result in a non-integer number, we round it down to the nearest integer. In addition, since the index of `list_of_req_time` array starts from 0, we subtract the rounding result with 1. This whole operation is performed to get the index position of the percentile value in `list_of_req_time` array.
- After we got the index position, we simply take the referred value in the `list_of_req_time` array. For instance, the index position of 20 means that we take the 20th element in `list_of_req_time`
- After the iteration completes, the `max_request_time` column will store a list of percentile values for each N-th percentile

Basically, the similar procedure also applies when we calculate the number of observations lie below a percentile value (`request_count`).

Take a look at the following query.

```
"request_count": {
    "$map": {
        "input": "$percentile",
        "as": "percentile_value",
        "in": {
            "$floor": {
                "$multiply": [
                    {
                        $divide: ["$$percentile_value", 100]
                    }, {"$size": "$list_of_req_time"}
                ]
            }
        }
    }
}
```

As you can see, we simply multiply the N-th percentile (already divided by 100) by the size of `list_of_req_time` to get the number of observations that lie below the N-th percentile value.

## 5th step: Create the percentile collection

In step #4, what we got are several columns, such as `percentile`, `max_request_time`, and `request_count`. Each column stores an array of values corresponding to each column. For instance, `percentile` column stores an array of N-th percentiles. Meanwhile, `max_request_time` stores an array of percentile values (request time at the N-th percentile).

To visualize, take a look at the following collection.

```
percentile        | max_request_time      | request_count
-------------------------------------------------------------
[25, 50, 75, 100] | [0.1, 0.3, 0.5, 0.7]  | [25, 50, 75, 100]
```

We already got what we looked for. How to transform the above collection into the following?

```
percentile  | max_request_time  | request_count
-----------------------------------------------
25          |       0.1         |       25
50          |       0.3         |       50
75          |       0.5         |       75
100         |       0.7         |       100
```

Turns out there's an interesting query in MongoDB called `unwind`. It basically performs the above task.

Here are the three `unwind` queries for each column. Please note that we also include the array index.

```
{ 
    "$unwind": { 
        "path": "$percentile", 
        "includeArrayIndex" : "percentile_idx" 
    } 
},
{ 
    "$unwind": { 
        "path": "$max_request_time", 
        "includeArrayIndex" : "max_request_time_idx" 

    }
},
{ 
    "$unwind": { 
        "path": "$request_count", 
        "includeArrayIndex" : "request_count_idx" 

    }
}
```

The above queries would result similar to the following.

```
percentile  | max_request_time  | request_count | percentile_idx  | max_request_time_idx  | request_count_idx
-------------------------------------------------------------------------------------------------------------
25          |       0.1         |       25      |         0       |           0           |         0
25          |       0.1         |       50      |         0       |           0           |         1
25          |       0.1         |       75      |         0       |           0           |         2
25          |       0.1         |       100     |         0       |           0           |         3
25          |       0.3         |       25      |         0       |           1           |         0
25          |       0.3         |       50      |         0       |           1           |         1
25          |       0.3         |       75      |         0       |           1           |         2
25          |       0.3         |       100     |         0       |           1           |         3
25          |       0.5         |       25      |         0       |           2           |         0
25          |       0.5         |       50      |         0       |           2           |         1
25          |       0.5         |       75      |         0       |           2           |         2
25          |       0.5         |       100     |         0       |           2           |         3
...
...
...
100         |       0.7         |       25      |         3       |           3           |         0
100         |       0.7         |       50      |         3       |           3           |         1
100         |       0.7         |       75      |         3       |           3           |         2
100         |       0.7         |       100     |         3       |           3           |         3
```

## 6th step: Consider only valid array index permutation

As you can see from step #5, the resulting collection (after unwinding) consists of some unintended rows. These rows are those whose `percentile_idx`, `max_request_time_idx`, and `request_count_idx` are not the same. We'll only consider rows with the same values for these three indexes since we'd like to get the following collection at the end.

```
percentile  | max_request_time  | request_count | percentile_idx  | max_request_time_idx  | request_count_idx
-------------------------------------------------------------------------------------------------------------
25          |       0.1         |       25      |         0       |           0           |         0
50          |       0.3         |       50      |         1       |           1           |         1
75          |       0.5         |       75      |         2       |           2           |         2
100         |       0.7         |       100     |         3       |           3           |         3
```

One of the simplest way to achieve this is by appending several new columns that denote whether the values of a pair of indexes are the same.

In the following query, notice that column `valid_m` and `valid_n` perform such a task.

```
{ 
    "$project": {
        "_id" : 0,
        "percentile": 1,
        "max_request_time": 1,
        "request_count": 1,
        "valid_m": { 
            "$eq": ["$percentile_idx", "$max_request_time_idx"] 
        },
        "valid_n": { 
            "$eq": ["$max_request_time_idx", "$request_count_idx"] 
        } 
    }
}
```

## 7th step: Filter out rows that don't have the same index values

We achieve this by simply check whether both of `valid_m` and `valid_n` hold true.

```
{ 
    "$match": { 
        "$and": [ {"valid_m": true}, {"valid_n": true} ]
    }
}
```

## 8th step: Drop non-relevant columns

Last but not least, we filter out all non-relevant columns. In this case, we don't need `valid_m` and `valid_n` columns anymore.

```
{
    "$project": {
        "valid_m": 0,
        "valid_n": 0
    }
}
```
