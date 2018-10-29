---
layout: post
title: Adding forms dynamically to a Django formset
date: 2017-07-11 21:00
tag:
- django
category: blog
author: taranjeet
description: This post is about adding forms dynamically to a formset.
---

### Django Forms

Forms in HTML are a collection of input elements that allows us to perform dynamic actions on a website. Django models these forms as an object of [**Form**](https://docs.djangoproject.com/en/2.0/topics/forms/) class. A form is responsible for taking input from a user, validating and cleaning data and taking necessary action as required.

### Formsets

A [formset](https://docs.djangoproject.com/en/2.0/topics/forms/formsets/) is a collection of Django Forms. Each formset has a management form which is used to manage the collection of homogeneous forms contained in it.

Formset stores data like the total number of forms, the initial number of forms and the maximum number of forms in a management form. So whenever we want to add a form dynamically on the frontend, we need to change the total number of forms so that “*Management form has been tampered error is not thrown*”. All the forms in a formset are numbered sequentially, so we need to do a little processing of these numbers to keep the whole structure of a form consistent.

Let's consider an example of a library wherein form is required to fill details of **Book**. The `Book` model looks like

{% gist dbf1010d5c952532d48290d75dc199ad %}

#### Use case 1: Create formset for a normal form

Let's create a view wherein a user can add and store multiple books at once. For this, we will need a form, whose formset can be made. We will first start with a normal form and see how we can make formsets using normal form. We are using [Bootstrap](https://getbootstrap.com/) to power our styling.

![normal-dynamic-formset](/public/img/book_formset.png "Dynamic Formsets")

The form definition will look like this

{% gist dca781f2726abaa15a59a079b6e0744c %}

Let's create a formset for this form using `formset_factory`. The updated `forms.py` will look like this

{% gist 5b2a5e12a1ba439d106c81d5db035613 %}

Now, let's use this form in a view and create an interface where a user can add or store multiple books

{% gist f0f6f3b2b365ea3f94195306a0de1214 %}

The template code to render and iterate over this formset will look like

{% gist aacdbb4dbcae29396e7692e1c6008b3c %}

This code will simply render the form. By default, a single element will be present, since we have passed `extra` as 1 when creating `BookFormset`.

Now we need to add some javascript code, to give the functionality of adding form elements when `+` button is pressed and removing form elements when `-` button is present.

This code will look like

{% gist 71b7826b60f42e5d239cf3b3abbf292f %}

When this code is added in `create_normal.html`, the functionality becomes complete wherein a user can add and remove form elements on the fly.

#### Use case 2: Create formset for a model form

Lets us consider the same problem, where we want to add multiple books, but this time using a `ModelForm`, rather than using a simple form. Since we are using `ModelForm`, a `ModelFormset` will be created.

{% gist 20dbf7c56c250ca0acc54f576dc7eee6 %}

The code for the views will be slightly changed. Here rather than creating a `Book` model instance and then saving it, `form` will be used, since `form` is an instance of `ModelForm`.

{% gist 879b920e56514af67348da503af2b33c %}

The template code(part 3) and javascript code(part 4) remains the same.

#### Use case 3: Create Formset and Form both

Let's consider a more complex case wherein we need to save form along with a formset. This can be easily understood by taking the example of `Book` and `Author`. Here we will create a modelform for `Book` and modelformset for `Author`.

![normal-dynamic-formset](/public/img/book_with_author.png "Form with Formsets")

{% gist c503278293ea77f77fb2e0f88f8d9fc7 %}

The corresponding code for the views will look like

{% gist bf3a42ee8bc682d023fbe4c6efd9c376 %}

The template code to iterate and render both the form and formset will look like

{% gist 6f921607b8f514ef68edfa45e28b7e22 %}

### Conclusion

This post walked in detail of how can the formsets be implemented. Much of the help for the above javascript code is taken from this [stackoverflow answer](https://stackoverflow.com/a/669982/2534102).

The code for this can be found [here](https://github.com/taranjeet/django-library-app)
