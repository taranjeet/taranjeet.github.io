---
layout: post
title: Elasticsearch Analyzers
date: 2017-05-09 23:00
tag:
- elasticsearch
category: blog
---

# Es Ananlyzers

A process performed on the document body before the document is sent off to be added into the inverted index. This is called Analysis. ES performs a number of steps, which are

* Character filtering
    - transforms the character using a character filter.
    - involves converting an arbitrary number of characters into other chars
    - Eg : Convert & to and

* Tokenization
    - Breaks text into a set of one or more tokens

* Token filtering
    - Transforms each token using a token filter.
    - Takes a token as input and can modify, add or remove more tokens
    - Eg : lowercasing

* Token indexing
    - Stores those tokens into the index.

### Add an analyzer when index is created.

```
curl -XPOST 'localhost:9200/myindex' -d '
{
  "settings": {
    "number_of_shards": 2,
    "index": {
      "analysis": {
        "filter": {
          "my_custom_filter": {
            "type": "lowercase"
          }
        },
        "analyzer": {
          "my_custom_analyzer": {
            "type": "custom",
            "tokenizer": "my_custom_tokenizer",
            "filter": [
              "my_custom_filter"
            ],
            "char_filter": [
              "my_customchar_filter"
            ]
          }
        },
        "tokenizer": {
          "my_custom_tokenizer": {
            "type": "letter"
          }
        },
        "char_filter": {
          "my_customchar_filter": {
            "type": "mapping",
            "mappings": [
              "u=>you"
            ]
          }
        }
      }
    }
  },
  "mappings": {
    # mapping here
  }
}
'

```

Analyzers can be added to ElasticSearch in two ways

* Specify analyzer at the time of index creation. This way one can always make changes to the analyzers __without restarting__ ElasticSearch

* Add analyzers into the ElasticSearch configuration file. But to reflect changes, one needs to __restart ElasticSearch__.


How to use analyzer for a field in the mapping ?

One example of such mapping is

```
{
  "mappings": {
    "document": {
      "properties": {
        "description": {
          "type": "string",
          "analyzer": "my_custom_analyzer"
        }
      }
    }
  }
}
```

### Analyze Api

Below are some examples which indicates how to use a particular analyzer or filter.

```
$ curl -XPOST 'localhost:9200/_analyze?analyzer=standard' -d 'this is a text meant to be analyzed by the standard analyzer'

$ curl -XPOST 'localhost:9200/_analyze?tokenizer=whitespace&filters=lowercase,reverse' -d 'this is again A sample text to BE passed through reverse filter'

```

### Built Ins


##### Analyzers

There analyzers are shipped by default with ES.

* Standard

    - Combines standard tokenizer, standard token filter, lower case token filter, and stop token filter.

* Simple

* Whitespace

* Stop

* Keyword

* Pattern

* Language and Multilingual

* Snowball

##### Tokenization

* Standard Tokenizer

* Keyword

* Letter

* Lowercase

* Whitespace

* Pattern

Eg : To split at `.-`

```
curl -XPOST 'localhost:9200/pattern' -d '
{
  "settings": {
    "index": {
      "analysis": {
        "tokenizer": {
          "pattern1": {
            "type": "pattern",
            "pattern": "\\.-"
          }
        }
      }
    }
  }
}
'
```

* UAX URL EMAIL

* Path Hierarchy

* Unigram, Bigram, Trigram

* Edge Gram

* Shingles
    - ngrams at token level instead of the character level

##### Token filters

* Standard

* Lowercase

* Length

* Stop

* Truncate, Trim, Limit Token Count

* Reverse

* Unique

* Ascii folding

* Synonym

##### Stemming

* Snowball

* Porter Stemmer

* Kstem



