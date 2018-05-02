---
title: "Elasticsearch&#58; Using Completion Suggester to build AutoComplete"
layout: post
date: 2018-01-26 20:00
tag:
- elasticsearch
category: blog
author: taranjeet
description: "Elasticsearch&#58; Using Completion Suggester to build AutoComplete"
---

If you are interested in more such blogs about Elasticsearch, join Elasticsearch Whatsapp Group by following this [link](https://chat.whatsapp.com/BBN3bFkXMjN27AIQ918npK).

In an earlier [post](/elasticsearch-autocomplete/), we discussed various approaches to implement Autocomplete functionality. We came to a conclusion that Completion Suggester covers most of the cases required in implementing a fully functional and fast autocomplete. This post explains in detail about what is Completion Suggester and how to use them practically.

## Completion Suggester

[Completion Suggester](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters-completion.html) is a type of suggester in [Elasticsearch](https://www.elastic.co/products/elasticsearch), which is used to implement autocomplete functionality. It uses an in-memory data structure called [Finite State Transducer](https://en.wikipedia.org/wiki/Finite-state_transducer). Elasticsearch stores FST on a per segment basis, which means suggestions scale horizontally as more new nodes are added.

### Mapping

To use Completion Suggester, a special type of mapping type called `completion` is defined. Let’s take example of Marvel movie data and define an index named `movies` with type as `marvels` . Complete movie list can be accessed from [here](https://github.com/taranjeet/data/blob/master/marvel_movies_v2.json)

```
PUT movies
{
  "mappings": {
    "marvels": {
      "properties": {
        "name": {
          "type": "completion"
        },
        "year": {
          "type": "keyword"
        }
      }
    }
  }
}
```

Here `name.completion` is a type of completion field. On this field, we can add various other mapping parameters like analyzer, search_analyzer, etc.

### Indexing Data

To index data, a slightly different syntax is used. A suggestion field is made of an `input` and an optional `weight` parameter. Let’s index a movie into our `movies` index.

```
POST movies/marvels
{
  "name": {
    "input": ["Iron Man"]
  },
  "year": 2008
}

# another movie

POST movies/marvels
{
  "name": {
    "input": ["Thor"]
  },
  "year": 2011
}
```

We can also define a `weight` for each field. This weight can help us in controlling the ranking of documents when querying.

```
POST movies/marvels
{
  "name": {
    "input": ["Iron Man 2"],
    "weight": 2
  },
  "year": 2010
}
```

We can also index multiple suggestions for a document at the same time

```
POST movies/marvels
{
  "name": [
    {
      "input": [
        "The Incredible Hulk"
      ]
    },
    {
      "input": [
        "Hulk"
      ]
    }
  ],
  "year": 2008
}
```

### Querying

To query document, we need to specify suggest type as `completion`. Let’s query for `thor` in our `movies` index. `movies` index contains all the 22 movies from Marvel Cinematic Section of this [page](http://marvel.com/movies/all).

```
GET movies/marvels/_search
{
  "suggest": {
    "movie-suggest": {
      "prefix": "thor",
      "completion": {
        "field": "name"
      }
    }
  }
}
```

We get the following movies as result

* Thor

* Thor: Ragnarok

* Thor: The Dark World

We see that all documents are having `_score` as 1. This means that all the documents in completion suggestor are ranked equally. To give boost to a particular document, or to alter the ranking, we can use the optional parameter called `weight`. We have already indexed `Iron Man`(with no weight) and `Iron Man 2`(with weight as 2). Let’s search for `Iron Man` in our movies index.

```
GET movies/marvels/_search
{
  "suggest": {
    "movie-suggest": {
      "prefix": "iron man",
      "completion": {
        "field": "name"
      }
    }
  }
}
```

We get the following movies as result

* Iron Man 2 (score as 2)

* Iron Man (score as 1).

We can clearly see here how weight is used to control the ranking of documents. This is the reason why `Iron Man 2` is ranked higher than `Iron Man` when searched for `Iron Man` .

We can also specify thesize to control the number of documents returned.

```
GET movies/marvels/_search
{
  "suggest": {
    "movie-suggest": {
      "prefix": "iron man",
      "completion": {
        "field": "name",
        "size": 1
      }
    }
  }
}
```

We can also add fuzziness in completion suggester. This helps us in providing suggestions even when there is a typo. Let’s try searching for `captain amrica the` with fuzzy query

```
GET movies/marvels/_search
{
  "suggest": {
    "movie-suggest-fuzzy": {
      "prefix": "captain amerca the",
      "completion": {
        "field": "name",
        "fuzzy": {
          "fuzziness": 1
        }
      }
    }
  }
}
```

We get the following movies as result

* Captain America: The First Avenger

* Captain America: The Winter Soldier

Let’s try finding suggestion for movie names which contain `america` .

```
GET movies/marvels/_search
{
  "suggest": {
    "movie-suggest": {
      "prefix": "america",
      "completion": {
        "field": "name"
      }
    }
  }
}
```

We get no results. This is because completion suggester support prefix matching. It starts matching from the start of the string and there is no movie which contains `america` at the start of the string. To deal with this type of situation, we can tokenize the input text on space and keep all the phrases as canonical names. This way `Captain America: The First Avenger` will be inserted as

```
POST movies/marvels
{
  "name": {
    "input": [
      "Captain America: The First Avenger",
      "America: The First Avenger",
      "The First Avenger",
      "First Avenger",
      "Avenger"
    ]
  },
  "year": "2011"
}
```

### Filtering Document

In queries, we can filter documents by using filter but filter does not work in Completion Suggester. To understand this better, let’s run a query which finds all movies with name `iron man` released in year `2008`.

```
GET movies/marvels/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "year": "2008"
        }
      }
    }
  },
  "suggest": {
    "movie-suggest": {
      "prefix": "iron man",
      "completion": {
        "field": "name",
        "fuzzy": {
          "fuzziness": 1
        }
      }
    }
  }
}
```

The response received looks like

```
{
  "hits": {
    "total": 3,
    "max_score": 0,
    "hits": [
      {
        "_index": "movies",
        "_type": "marvels",
        "_id": "AWExIPoUXjXDcBJXOfPD",
        "_score": 0,
        "_source": {
          "name": {
            "input": [
              "The Incredible Hulk"
            ]
          },
          "year": 2008
        }
      },
      ...
    ]
  },
  "suggest": {
    "movie-suggest": [
      {
        "text": "iron man",
        "offset": 0,
        "length": 8,
        "options": [
          {
            "text": "Iron Man 2",
            "_index": "movies_v5",
            "_type": "marvels",
            "_id": "AWExJs9sXjXDcBJXOfPH",
            "_score": 14,
            "_source": {
              "name": {
                "input": [
                  "Iron Man 2"
                ],
                "weight": 2
              },
              "year": 2010
            }
          },
          ...
        ]
      }
    ]
  }
}
```

In the response we see that `hits` key along with `suggest` is present. This happened because query and suggest works at the same level parallely. Hence we get both keys in response. So we cannot apply filter in a suggestion query.

To deal with this, Completion Suggester provides [Context Suggester](https://www.elastic.co/guide/en/elasticsearch/reference/6.1/suggester-context.html), which are basically filters for `completion` field. Let’s define another mapping for movies index, this time with `year` as a context suggester for `name` field.

```
PUT movies
{
  "mappings": {
    "marvels": {
      "properties": {
        "name": {
          "type": "completion",
          "contexts": [
            {
              "name": "year",
              "type": "category",
              "path": "year"
            }
          ]
        },
        "year": {
          "type": "text"
        }
      }
    }
  }
}
```

We can index our complete movies data into this index. Let’s find all movies with name `iron man` released in year `2008`.

```
GET movies/marvels/_search
{
  "suggest": {
    "movie-suggest": {
      "prefix": "iron man",
      "completion": {
        "field": "name",
        "contexts": {
          "year": ["2008"]
        }
      }
    }
  }
}
```

We get the following movies as result

* Iron Man

We can also boost context suggester as well. Let’s search for movies with name as `iron man`, released in year `2008` and `2010`, giving a boost of 4 to year `2008` .

```
GET movies/marvels/_search
{
  "suggest": {
    "movie-suggest": {
      "prefix": "iron man",
      "completion": {
        "field": "name",
        "contexts": {
          "year": [
            {"context": "2008", "boost": 4},
            {"context": "2010"}
            ]
        }
      }
    }
  }
}
```

We get the following movies as result

* Iron Man (score as 4)

* Iron Man 2 (score as 1)

## References

* [Completion Suggester Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters-completion.html)

* [Context Suggester](https://www.elastic.co/guide/en/elasticsearch/reference/6.1/suggester-context.html)

If you are interested in more such blogs about Elasticsearch, join **Elasticsearch Whatsapp Group** by following this [link](https://chat.whatsapp.com/BBN3bFkXMjN27AIQ918npK)
