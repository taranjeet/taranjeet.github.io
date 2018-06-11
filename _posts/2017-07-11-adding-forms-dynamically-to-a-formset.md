---
layout: post
title: Adding forms dynamically to a Django formset
date: 2017-07-11 21:00
tag:
- django
category: blog
---

### Django Forms

Forms in HTML are a collection of input elements that allows us to perform dynamic actions on a website. Django models these forms as an object of [**Form**](https://docs.djangoproject.com/en/2.0/topics/forms/) class. A form is responsible for taking input from a user, validating and cleaning data and taking necessary action as required.

### Formsets

A [formset](https://docs.djangoproject.com/en/2.0/topics/forms/formsets/) is a collection of Django Forms. Each formset has a management form which is used to manage the collection of homogeneous forms contained in it.

Formset stores data like total number of forms, the initial number of forms and maximum number of forms in a management form. So whenever we want to add a form dynamically on the frontend, we need to change the total number of forms so that “*Management form has been tampered error is not thrown*”. All the forms in a formset are numbered sequentially, so we need to do a little processing of these numbers to keep the whole structure of a form consistent.

Let's consider an example of a library wherein form is required to fill details of **Book**. The `Book` model looks like

```
# models.py

from django.db import models

class Book(models.Model):

    name = models.CharField(max_length=255)
    isbn_number = models.CharField(max_length=13)

    class Meta:
        db_table = 'book'

    def __str__(self):
        return self.name

```

#### Use case 1: Create formset for a normal form

Let's create a view wherein a user can add and store multiple books at once. For this we will need a form, whose formset can be made. We will first start with a normal form and see how we can make formsets using normal form. We are using [Bootstrap](https://getbootstrap.com/) to power our styling.

![normal-dynamic-formset](/public/img/book_formset.png "Dynamic Formsets")

The form definition will look like this

```
# forms.py
from django import forms


class BookForm(forms.Form):
    name = forms.CharField(
        label='Book Name',
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Enter Book Name here'
        })
    )
```

Let's create a formset for this form using `formset_factory`. The updated `forms.py` will look like this

```
# forms.py :: part 1
from django import forms
from django.forms import formset_factory


class BookForm(forms.Form):
    name = forms.CharField(
        label='Book Name',
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Enter Book Name here'
        })
    )

BookFormset = formset_factory(BookForm, extra=1)
```

Now, let's use this form in a view and create an interface where a user can add or store multiple books

```
# views.py :: part 2
from django.shortcuts import render, redirect

from .forms import BookFormset
from .models import Book


def create_book_normal(request):
    template_name = 'store/create_normal.html'
    heading_message = 'Formset Demo'
    if request.method == 'GET':
        formset = BookFormset(request.GET or None)
    elif request.method == 'POST':
        formset = BookFormset(request.POST)
        if formset.is_valid():
            for form in formset:
                # extract name from each form and save
                name = form.cleaned_data.get('name')
                # save book instance
                if name:
                    Book(name=name).save()
            # once all books are saved, redirect to book list view
            return redirect('book_list')
    return render(request, template_name, {
        'formset': formset,
        'heading': heading_message,
    })
```

The template code to render and iterate over this formset will look like

```
# create_normal.html :: part 3
<form class="form-horizontal" method="POST" action="">
{{ "{% csrf_token " }}%}
{{ "{{ formset.management_form "}}}}
{{ "{% for form in formset " }}%}
<div class="row form-row spacer">
    <div class="col-2">
        <label>{{ "{{form.name.label"}}}}</label>
    </div>
    <div class="col-4">
        <div class="input-group">
            {{ "{{form.name"}}}}
            <div class="input-group-append">
                <button class="btn btn-success add-form-row">+</button>
            </div>
        </div>
    </div>
</div>
{{ "{% endfor " }} %}
<div class="row spacer">
    <div class="col-4 offset-2">
        <button type="submit" class="btn btn-block btn-primary">Create</button>
    </div>
</div>
</form>
```

This code will simply render the form. By default, a single element will be present, since we have passed `extra` as 1 when creating `BookFormset`.

Now we need to add some javascript code, to give the functionality of adding form elements when `+` button is pressed and removing form elements when `-` button is present.

This code will look like

```
# create_normal.html :: part 4
<script type='text/javascript'>

function updateElementIndex(el, prefix, ndx) {
    var id_regex = new RegExp('(' + prefix + '-\\d+)');
    var replacement = prefix + '-' + ndx;
    if ($(el).attr("for")) $(el).attr("for", $(el).attr("for").replace(id_regex, replacement));
    if (el.id) el.id = el.id.replace(id_regex, replacement);
    if (el.name) el.name = el.name.replace(id_regex, replacement);
}

function cloneMore(selector, prefix) {
    var newElement = $(selector).clone(true);
    var total = $('#id_' + prefix + '-TOTAL_FORMS').val();
    newElement.find(':input').each(function() {
        var name = $(this).attr('name').replace('-' + (total-1) + '-', '-' + total + '-');
        var id = 'id_' + name;
        $(this).attr({'name': name, 'id': id}).val('').removeAttr('checked');
    });
    total++;
    $('#id_' + prefix + '-TOTAL_FORMS').val(total);
    $(selector).after(newElement);
    var conditionRow = $('.form-row:not(:last)');
    conditionRow.find('.btn.add-form-row')
    .removeClass('btn-success').addClass('btn-danger')
    .removeClass('add-form-row').addClass('remove-form-row')
    .html('<span class="glyphicon glyphicon-minus" aria-hidden="true"></span>');
    return false;
}

function deleteForm(prefix, btn) {
    var total = parseInt($('#id_' + prefix + '-TOTAL_FORMS').val());
    if (total > 1){
        btn.closest('.form-row').remove();
        var forms = $('.form-row');
        $('#id_' + prefix + '-TOTAL_FORMS').val(forms.length);
        for (var i=0, formCount=forms.length; i<formCount; i++) {
            $(forms.get(i)).find(':input').each(function() {
                updateElementIndex(this, prefix, i);
            });
        }
    }
    return false;
}

$(document).on('click', '.add-form-row', function(e){
    e.preventDefault();
    cloneMore('.form-row:last', 'form');
    return false;
});

$(document).on('click', '.remove-form-row', function(e){
    e.preventDefault();
    deleteForm('form', $(this));
    return false;
});

</script>
```

When this code is added in `create_normal.html`, the functionality becomes complete wherein a user can add and remove form elements on the fly.

#### Use case 2: Create formset for a model form

Lets us consider the same problem, where we want to add multiple books, but this time using a `ModelForm`, rather than using a simple form. Since we are using `ModelForm`, a `ModelFormset` will be created.

```
# forms.py :: part 1

from django.forms import modelformset_factory

BookModelFormset = modelformset_factory(
    Book,
    fields=('name', ),
    extra=1,
    widgets={'name': forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Enter Book Name here'
        })
    }
)
```

The code for the views will be slightly changed. Here rather than creating a `Book` model instance and then saving it, `form` will be used, since `form` is an instance of `ModelForm`.

```
# views.py :: part 2

from django.shortcuts import render, redirect

from .forms import BookModelFormset


def create_book_model_form(request):
    template_name = 'store/create_normal.html'
    heading_message = 'Model Formset Demo'
    if request.method == 'GET':
        # we don't want to display the already saved model instances
        formset = BookModelFormset(queryset=Book.objects.none())
    elif request.method == 'POST':
        formset = BookModelFormset(request.POST)
        if formset.is_valid():
            for form in formset:
                # only save if name is present
                if form.cleaned_data.get('name'):
                    form.save()
            return redirect('book_list')

    return render(request, template_name, {
        'formset': formset,
        'heading': heading_message,
    })

```

The template code(part 3) and javascript code(part 4) remains the same.

#### Use case 3: Create Formset and Form both

Let's consider a more complex case wherein we need to save form along with a formset. This can be easily understood by taking the example of `Book` and `Author`. Here we will create a modelform for `Book` and modelformset for `Author`.

![normal-dynamic-formset](/public/img/book_with_author.png "Form with Formsets")

```
# forms.py :: part 1

from django import forms
from django.forms import modelformset_factory

from .models import Book, Author

class BookModelForm(forms.ModelForm):

    class Meta:
        model = Book
        fields = ('name', )
        labels = {
            'name': 'Book Name'
        }
        widgets = {
            'name': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': 'Enter Book Name here'
                }
            )
        }

AuthorFormset = modelformset_factory(
    Author,
    fields=('name', ),
    extra=1,
    widgets={
        'name': forms.TextInput(
            attrs={
                'class': 'form-control',
                'placeholder': 'Enter Author Name here'
            }
        )
    }
)

```

The corresponding code for the views will look like

```
# views.py :: part 2

def create_book_with_authors(request):
    template_name = 'store/create_with_author.html'
    if request.method == 'GET':
        bookform = BookModelForm(request.GET or None)
        formset = AuthorFormset(queryset=Author.objects.none())
    elif request.method == 'POST':
        bookform = BookModelForm(request.POST)
        formset = AuthorFormset(request.POST)
        if bookform.is_valid() and formset.is_valid():
            # first save this book, as its reference will be used in `Author`
            book = bookform.save()
            for form in formset:
                # so that `book` instance can be attached.
                author = form.save(commit=False)
                author.book = book
                author.save()
            return redirect('store:book_list')
    return render(request, template_name, {
        'bookform': bookform,
        'formset': formset,
    })
```

The template code to iterate and render both the form and formset will look like

```
# create_with_author.html :: part 3

<form class="form-horizontal" method="POST" action="">
    {{ "{% csrf_token " }}%}
<div class="row spacer">
<div class="col-2">
    <label>{{ "{{bookform.name.label"}}}}</label>
</div>
<div class="col-4">
    <div class="input-group">
        {{ "{{bookform.name"}}}}
    </div>
</div>
</div>
{{ "{{ formset.management_form "}}}}
{{ "{% for form in formset " }}%}
<div class="row form-row spacer">
    <div class="col-2">
        <label>{{ "{{form.name.label"}}}}</label>
    </div>
    <div class="col-4">
        <div class="input-group">
            {{ "{{form.name"}}}}
            <div class="input-group-append">
                <button class="btn btn-success add-form-row">+</button>
            </div>
        </div>
    </div>
</div>
{{ "{% endfor " }}%}
<div class="row spacer">
    <div class="col-4 offset-2">
        <button type="submit" class="btn btn-block btn-primary">Create</button>
    </div>
</div>
</form>
```

### Conclusion

This post walked in detail of how can the formsets be implemented. Much of the help for above javascript code is taken from this [stackoverflow answer](https://stackoverflow.com/a/669982/2534102).

The code for this can be found [here](https://github.com/taranjeet/django-library-app)
