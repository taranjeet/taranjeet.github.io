---
title: "Django &#58; QuerySet Anti patterns"
layout: post
date: 2019-01-06 13:30
tag:
- django
category: blog
author: taranjeet
description: This post is about the anti-patterns found when using django querysets.
---

This post is about the anti-patterns found when using django querysets.

Lets us consider two models namely `Book` and `Author`.

```python
# publish/models.py
from django.db import models


class Book(models.Model):

    name = models.CharField(max_length=255)
    isbn = models.CharField(max_length=13)


class Author(models.Model):

    name = models.CharField(max_length=255)
    book = models.ForeignKey(Book, related_name='authors', on_delete=models.CASCADE)
```

Now lets us suppose that we have a view which renders the list of book. It renders the name and isbn of the book, and the count of its author. To get the count of author, we will add a property in `Book` model, which will get the author count.

```python
# publish/views.py

from django.db import models


class Book(models.Model):

    name = models.CharField(max_length=255)
    isbn = models.CharField(max_length=13)

    def get_author_count(self):
        return self.authors.count()
```

The corresponding code will look like

```python
# publish/views.py

from django.shortcuts import render

from publish.models import Book


def book_list(request):
    template_name = 'publish/book_list.html'
    books = Book.objects.all()
    return render(request, template_name, {
        'books': books,
    })
```

```html
# publish/book_create.html

{% raw %}
{% load staticfiles %}
<html lang="en">
<head>
<title>Book List</title>
</head>

<body>
    <table>
        {% for book in books %}
            <tr>
                <td>{{book.name}}</td>
                <td>{{book.isbn}}</td>
                <td>{{book.get_author_count}}</td>
            </tr>
        {% endfor %}
    </table>
</body>
</html>
{% endraw %}
```

Now if we see this carefully, here for each instance of `Book`, a count query will be sent. Even if we use `cached_property`, for each instance a query will be sent and then it will be cached later on.

We can reduce this one query per instance by using `annotate`. The updated code can be written as

```python
# publish/views.py

from django.db.models import Count
from django.shortcuts import render

from publish.models import Book


def book_list(request):
    template_name = 'publish/book_list.html'
    books = Book.objects.all().annotate(author_count=Count('authors'))
    return render(request, template_name, {
        'books': books,
    })
```

```html
# publish/book_create.html

{% raw %}
{% load staticfiles %}
<html lang="en">
<head>
    <title>Book List</title>
</head>

<body>
    <table>
        {% for book in books %}
            <tr>
                <td>{{book.name}}</td>
                <td>{{book.isbn}}</td>
                <td>{{book.author_count}}</td>
            </tr>
        {% endfor %}
    </table>
</body>
</html>
{% endraw %}
```

This sends a single query which gets count of authors per book.


### Summary

This post is about various anti-patterns encountered when using django querysets.
