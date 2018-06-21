---
layout: post
title: What I know about ElasticSearch
date: 2017-05-10 22:00
tag:
- elasticsearch
category: blog
---

### What is ES ?

- distributed db

- runs in a cluster manner

- easily horizontally scalable

### Analogies

Linking these terminologies with traditional mysql terms

    databases --> indices
    tables --> types
    rows --> documents
    columns --> fields


### Terminologies

* Node : running instance of ES

* Cluster : consists of one or more nodes with same cluster name, working together to share data

* Master node : one node generally, in charge of managing things like creating/deleting index, or adding/removing nodes from cluster.

* Cluster Health :

    - green : all well

    - yellow : all primary active, not all replica active

    - red : not all primary shard active

### Percolation

Reverse index on queries and then auto notify users if any matching doc arrives

### Querying

In this section we will try understanding how to write queries in ElasticSearch

Below documents are taken from elasticsearch official documentation.

```
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}

PUT /megacorp/employee/2
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}

PUT /megacorp/employee/3
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",s
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}
```

Some of the queries in MYSQL way, so that any one from the relational background can relate it very easily

* Select * from employee where id = 1;

```
GET /megacorp/employee/1
```

* Select * from employee limit 10

```
GET /megacorp/employee/_search
```

* Select * from employee where last_name = "Smith"

```
GET /megacorp/employee/_search
{
    "query":{
        "match": {
            "last_name": "Smith"
        }
    }
}
```

* Select * from employee where last_name = "Smith" and age> 30;

```
GET /megacorp/employee/_search
{
    "query" : {
        "bool": {
            "must": {
                "match" : {
                    "last_name" : "Smith"
                }
            },
            "filter": {
                "range" : {
                    "age" : { "gt" : 30 }
                }
            }
        }
    }
}
```

Here one can also use `filter` for last_name condition. So there is difference between `must` and `filter`. must is used for __scoring__, which means that the document containing Smith will have a higher score and filter is just for the __presence__ of the condition (Yes or No). Summarizing

- __must__ : clause must appear in the matching document and will contribute to the score

- __filter__ : clause must appear in the matching document, however there will be no contribution to the score.

* Difference between query and filter

__query__ : to search docs along with scoring, and for full text search

__filter__ : to basically filter out the docs, and for querying on exact values

Also a filter is applied first

* Difference between must and should

must means AND i.e. clause must appear in the matching docs
should means OR i.e. at least one of the clause should appear in matching docs

```
{
  "size": 0,
  "aggs": {
    "langs": {
      "terms": {
        "field": "language"
      }
    }
  }
}
```

* Difference between term, match phrase and query string

__term__ - matches a single term as it is, value is not analyzed
match_phrase - will analyze the input if analyzers are defined for the queried field and find documents matching the following criteria

* all the terms must appear in the same field

* they must have the same order

__query string__ : keeps all the content of the field in (_all) and then searches over it

### Routing

Basically to control a document will be stored in which shard. Can specify multiple keys in routing.

```
# here routing is done on last name.
GET /megacorp/employee/_search?routing=Smith
```

### Searching

All search requests are multi-index multi-type, meaning that you can search across all indices and all types.

* A search request can have its on timeout value. Eg:

```
GET /megacorp/employee/_search
{
    "size": 10,
    "from": 20,
    "timeout": 10s
    "query": {
        "match": {
            "last_name": "Smith"
        }
    }
}
```

### Sort

Supports sort mode option. Controls what array value is picked for sorting the doc it belongs to.

```
PUT /my_index/my_type/1?refresh
{
   "product": "chocolate",
    "price": [20, 4]
}

POST /_search
{
   "query" : {
      "term" : { "product" : "chocolate" }
   },
   "sort" : [
      {"price" : {"order" : "asc", "mode" : "avg"}}
   ]
}
```

Also can specify what needs to be done if any doc is missing that value.

post_filter is applied to the search hits at the very end of the search result, after aggregations have been calculated. More can be found [at](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-post-filter.html)

Query rescoring can also be done. It is done on top K docs, has an advantage of being fast as rescoring is done only on K docs instead of the whole set of documents.

* Can boost across indices

* __min_score__: fetch only those documents which have a certain minimum score.


### Full Text Queries

* match

* match_phrase

* match_phrase_prefix

* mutli_match

* common_terms

* query_string

* simple_query_string


* match explained

```
{
    "query": {
        "match":{
            "about": "rock climbing"
        }
    }
}
```

match by default uses OR operator, so it will look for documents containing "rock OR climbing". Documents containing both will be ranked higher. To change behavior of `match` to use `AND` operation, use a operator And field.

```
{
    "query": {
        "match": {
            "about": {
                "query": "rock climbing",
                "operator": "and"
            }
        }
    }
}
```

### Points To Keep in Mind

* To select only certain fields, use `stored_fields`. Else ES will load the whole doc and load fields from there when `_source` is used. For stored_fields, you need to state this in the mapping itself.

### Full Text Queries

* Analyzes the string before searching

## Term level queries

* Searches for the exact terms that are stored in the inverted index

### Term query

* Finds docs that contain the exact term specified in the inverted index. individual terms can be boosted also.

### Terms query

* Finds docs that contain any of the specified terms(not analyzed). Also values for terms filter can be fetched from a field in a document with the specified id in the specified type and index.

### Range query

* To fetch values within a particular range

### Exists query

* Fetch documents that have at least one non-null value in the original field


Include about following

* what is sharding, node ?

* types of node? how to create client and master node ?

* how to install es

* important flag in elasticsearch config like cluster name, node name, port, network host, master machine and unicast address

* what is quoram maintainence

* types of locking - node and global level locking

* threadpool

* types of log

* dynamic templates

* id generation algorithm ?

* node stats

* cluster stats

* inverted index and data structure used to store it

* cosine similarlity, vector model and tf-idf


* https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-termvectors.html

* https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-post-filter.html

https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pattern-replace-charfilter.html
