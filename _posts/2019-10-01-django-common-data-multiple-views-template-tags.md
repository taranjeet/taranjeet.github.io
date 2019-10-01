---
title: "Django &#58; Pass common data from multiple views using template tags"
layout: post
date: 2019-10-01 21:30
tag:
- django
category: blog
author: taranjeet
description: This post explains hot to pass common data from multiple views using template tags.
---

Let's us consider a `library` django app and `Book` model. On each page, we need to show the number of `Books` which are available in the library. We can find such book using the query:

```python
books_available = Book.objects.filter(is_issued=False).count()
```

Let's suppose our html pages are designed in such a way, that we need to show `books_available` at several pages.

One way to do this can be to use a [context processor](https://docs.djangoproject.com/en/2.2/ref/templates/api/#writing-your-own-context-processors). But then it would be available at each page.

Other way to do this can be to use a [custom template tag](https://docs.djangoproject.com/en/2.2/howto/custom-template-tags). This can be used by any page and it is not available for each and every page.

To create a custom template tag, we will first the create the following directory structure.

```sh
library
├── __init__.py
├── admin.py
├── apps.py
├── migrations
│   ├── 0001_initial.py
│   └── __init__.py
├── models.py
├── templatetags
│   ├── __init__.py
│   └── library_extras.py
├── tests.py
└── views.py
```
We can see that we have created a `templatetags` directory and within it `library_extras`. This `library_extras` will hold our custom template tags and can be requested in a html page by using:

```
{% raw %}
{% load library_extras %}
{% endraw %}
```

Now let's write the custom template tag code in `library_extras.py`

```python

from django import template

from library.models import Book

register = template.Library()


@register.simple_tag()
def get_books_available():
    return Book.objects.filter(is_issued=False).count()
```

Now we can use this tag in the html file:

```
{% raw %}
{% load library_extras %}

{% get_books_available %}
{% endraw %}
```

If we don't use custom template tag, then for each page wherever we require `books_available` value, we need to pass it in the context. This is fine for 2-3 views but becomes cumbersome when it needs to be passed in multiple views.

## Conclusion

In this post we learned how we can use a template tag to access common data at several pages.
