---
title: "Elasticsearch&#58; Building Autocomplete functionality"
layout: post
date: 2017-12-31 20:00
tag:
- elasticsearch
category: blog
author: taranjeet
description: This post is about different ways in which autocompletion can be implemented.
---

## **What is Autocomplete ?**

Let’s take a very common example. Whenever you go to google and start typing, a drop-down appears which lists the suggestions. Those suggestions are related to the query and help the user in completing his query.

![Suggestions when typing on Google](https://cdn-images-1.medium.com/max/2000/1*Vq_ETVxcROG7fBtMkiqJuw.png)*Suggestions when typing on Google*

Autocomplete as the [wikipedia](https://en.wikipedia.org/wiki/Autocomplete) says
> Autocomplete, or word completion, is a feature in which an application predicts the rest of a word a user is typing

It is also known as **Search as you type** or **Type Ahead Search**. It helps in navigating or guiding a user by prompting them with likely completions and alternatives to the text as they are typing it. It reduces the amount of character a user needs to type before executing any search actions, thereby enhancing the search experience of users.

AutoCompletion can be implemented by using any database. In this post, we will use [**Elasticsearch**](https://www.elastic.co/products/elasticsearch) to build autocomplete functionality.

Elasticsearch is an open source, distributed and JSON based search engine built on top of [Lucene](https://lucene.apache.org/core/).

## **Approaches**

There can be various approaches to build autocomplete functionality in Elasticsearch. We will discuss the following approaches.

* Prefix Query

* Edge Ngram

* Completion Suggester

**Prefix Query**

This approach involves using a [prefix query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-prefix-query.html) against a custom field. The value for this field can be stored as a keyword so that multiple terms(words) are stored together as a single term. This can be accomplished by using [keyword tokeniser](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-keyword-tokenizer.html). This approach has some disadvantages.

* Since the matching is supported only at the beginning of the term, one cannot match the query in the **middle** of the text.

* This type of query is not optimised for **large dataset** and may result in increased latency.

* Since this is a query, **duplicate results** won’t be filtered out. One workaround to deal with this approach can be using an aggregation query to group results and then filtering out results. This involves a bit of processing though on the server side.

**Edge Ngrams**

This approach involves using **different analysers** at index and search time. When indexing the document, a custom analyser with an [edge n-gram](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-edgengram-tokenizer.html) filter can be applied. At search time, [standard analyser](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-analyzer.html) can be applied. which prevents the query from being split.

Edge N-gram tokeniser first breaks the text down into words on custom characters (space, special characters, etc..) and then keeps the n-gram from the start of the string only.

This approach works well for matching query in the middle of the text as well. This approach is generally fast for queries but may result in slower indexing and in large index storage.

**Completion Suggester**

Elasticsearch is shipped with an in-house solution called [Completion Suggester](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters-completion.html). It uses an in-memory data structure called [Finite State Transducer](https://en.wikipedia.org/wiki/Finite-state_transducer)(FST). Elasticsearch stores FST on a per segment basis, which means suggestions scale horizontally as more new nodes are added.

Some of the things to keep in mind when implementing Completion Suggester

* The autosuggest items should have completion types as its field type.

* An input field can have various canonical or alias name for a single term.

* Weights can be defined with each document to control their ranking.

* Storing all the terms in lowercase helps in the case-insensitive match.

* Context suggesters can be enabled to support filtering or boosting by certain criteria.

This approach is the ideal approach to implement autocomplete functionality, however, it also has certain disadvantages

* Matching always starts at the beginning of the text. So search for america in marvels movie dataset will not yield any result. One way to overcome is tokenizing the input text on space and keep all the phrases as canonical names. This way Captain America: Civil War will be stored as

{% gist 960017acf0996d120afbf278c7df942d %}

* Highlighting of the matched words are not supported.

* No sorting mechanism is available. The only way to sort suggestions is via weights. This creates a problem when any custom sorting like alphabetical sort or sort by context is required.

## **Implementation**

Let’s implement the above approaches in Elasticsearch. We will be using Marvels movie data to build our sample index. For easy reference, here is the

* Spider-Man: Homecoming

* Ant-man and the Wasp

* Avengers: Infinity War Part 2

* Captain Marvel

* Black Panther

* Avengers: Infinity War

* Thor: Ragnarok

* Guardians of the Galaxy Vol 2

* Doctor Strange

* Captain America: Civil War

* Ant-Man

* Avengers: Age of Ultron

* Guardians of the Galaxy

* Captain America: The Winter Soldier

* Thor: The Dark World

* Iron Man 3

* Marvel’s The Avengers

* Captain America: The First Avenger

* Thor

* Iron Man 2

* The Incredible Hulk

* Iron Man

We will be creating an index `movies` with type `marvels`.

{% gist aeffcf1525f6285c5449057b2092ccd6 %}

If we see the mapping, we will observe that name is a nested field which contains several field, each analysed in a different way.

* Field `name.keywordstring` is analysed using a Keyword tokenizer, hence it will be used for **Prefix Query** Approach

* Field `name.edgengram` is analysed using Edge Ngram tokenizer, hence it will be used for **Edge Ngram** Approach.

* Field `name.completion` is stored as a completion type, hence it will be used for Completion Suggester.

We will index all our movies by using

{% gist 68ff58a8e17048286d217043c7d7d24a %}

Let’s start with Prefix Query approach and try finding movie beginning with `th`.

Query will be

{% gist 0402e1ad9bae7e65c876ce9ecbdd900a %}

This will result in the following movie

* Thor: The Dark World

* Thor: Ragnarok

* The Incredible Hulk

* Thor

The result is fair, but some movies like *Captain America: The Winter Soldier*, *Guardians of the Galaxy* are **missed** because prefix query only matches at the beginning of the text and not in the middle.

Let's try finding another movie beginning with `am`.


{% gist 0eae921adb5617f88465710d7deec71b %}

Here we do not get any results, although *Captain America* satisfy this condition. This confirms the point that Prefix query cannot be used to match in the middle of the text.

Lets run the same search `am` but with Edge Ngram Approach.


{% gist 67d115e8b76095c78aee5ca94a3779f4 %}
```
{
  "query": {
    "match": {
      "name.edgengram": "am"
    }
  }
}
```

Here we get the following result

* Captain America: The First Avenger

* Captain America: Civil War

* Captain America: The Winter Soldier

Let’s try finding for Captain America again, but this time with a bigger phrase `captain america the`

Using Edge N-gram approach, we get the following movies

* Captain America: The Winter Soldier

* Captain America: The First Avenger

* Captain America: Civil War

* Thor: The Dark World

* Captain Marvel

* Guardians of the Galaxy

* The Incredible Hulk

* Guardians of the Galaxy Vol 2

* Ant-man and the Wasp

* Marvel’s The Avengers

If we observe our phrase, only the first two suggestion makes sense. The reason for so many terms getting matched is the functioning of `match` clause. match includes all the documents which contain `captain OR america OR the`. Since the field is analysed using ngram, more suggestions(if present) will get included as well.

Let’s try using the suggestion query for the same phrase `captain america the` . Suggestion query is written in a slightly different way.


{% gist ebc42072c89e62b59b384baeb95ff14c %}

We get the following movies as result

* Captain America: The First Avenger

* Captain America: The Winter Soldier

Let’s try the same query, but this time with a typo `captain amrica the`.

The above `movie-suggest` returns no result because no support for fuzziness is present. We can update the query to include support for fuzziness in the following way

{% gist c7337379a810fda232162c8f320b4887 %}

The above query returns the following results

* Captain America: The First Avenger

* Captain America: The Winter Soldier

## Conclusion

Various approaches can be used to implement autocomplete functionality in ElasticSearch. Completion Suggester covers most of the cases which are required in implementing a fully functional and fast autocomplete.
