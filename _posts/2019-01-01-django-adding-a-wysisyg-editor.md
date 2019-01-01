---
title: "Django &#58; Adding a WYSIWYG Editor"
layout: post
date: 2019-01-01 18:00
tag:
- django
category: blog
author: taranjeet
description: This post is about adding a wysiwyg editor in django.
---

This post is about adding a wysiwyg editor in django. It uses [Trix](https://github.com/basecamp/trix), which is a WYSIWYG editor by Basecamp.

WYSIWYG stands for __What you see is what you get__. Wikipedia defines WYSIWYG as

> A WYSIWYG editor is a system in which content (text and graphics) can be edited in a form closely resembling its appearance when printed or displayed as a finished product, such as a printed document, web page, or slide presentation.

Lets take a `Book` model and corresponding `BookForm` which is used to add a new Book. The `Book` model looks like

```
# models.py

from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=255)
    description = models.TextField()

```

Here, we are considering that `description` is a rich text edit field, where we will use a wysiwyg editor to store information. `BookForm` will look like

```
# forms.py

from django import forms

class BookForm(forms.Form):
    title = forms.CharField()
    description = forms.CharField(widget=forms.HiddenInput())

```

Note that we have defined a `HiddenInput` widget for `description` field. This is because we are using Trix editor, which [states](https://github.com/basecamp/trix#integrating-with-forms) that the field should be a hidden field with an id attribute.


Now lets consider, that we are using a simple view to render this form.

```
# views.py

from django.shortcuts import render

from .forms import BookForm


def create_book(request):
    template_name = 'book/create.html'
    form = BookForm(request.GET)
    # do further processing here

    return render(request, template_name, {'form': form})
```

To render Trix editor, we will need two files `trix.css` and `trix.js`. We are assuming that the necessary static file settings for a django project are in place and `trix.css` and `trix.js` are placed in a folder named `trix` under static files folder.

Lets define the template which will contain the form, where description field will be a rich text editor.

```
# book/create.html

{% raw %}
{% load staticfiles %}
<html lang="en">
<head>
    <link href="{% static 'trix/trix.css' %}" rel="stylesheet">
    <script src="{% static 'trix/trix.js' %}"></script>
</head>

<body>
    <div style="width:50%; margin-left:10%; margin-top:20px;">
        <form action="" method="POST">
            {% csrf_token %}
            Title : {{form.title}}
            Description : {{form.description}}
            <trix-editor input="{{form.description.auto_id}}"></trix-editor>
        </form>
    </div>
</body>
</html>
{% endraw %}

```

We can see that for a rich text field (`description` here), we have defined a custom html tag `trix-editor` whose `input` attribute values is the id of `description` field. This way we have added a wysiwyg field in django. The value from this field can be processed in the same way as it is processed from any form field.

```
# views.py

def create_book(request):
    ....
    if request.method == 'POST':
        form = BookForm(request.POST)
        if form.is_valid():
            # usual way, in which other form fields are processed
            description = form.cleaned_data.get('description')

```

### Summary

There are various other rich text editors in the market along with some [comparisons](https://github.com/jaredreich/pell#comparisons). One very famous editor is [TinyMCE](https://www.tiny.cloud/) which has a django app called [django-tinymce](https://github.com/aljosa/django-tinymce). We opted for Trix, because it is minimalistic, easy to integrate and fits our use case.

