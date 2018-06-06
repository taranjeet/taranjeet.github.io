---
layout: post
title: Adding forms dynamically to a Django formset
date: 2017-07-11 21:00
tag:
- django
category: blog
---

### Django Forms

Forms in HTML are a collection of input elements that allows us to perform dynamic actions on a website. Django models these forms as an object of [**Form**](https://docs.djangoproject.com/en/1.11/topics/forms/) class. A form is responsible for taking input from a user, validating and cleaning data and taking necessary action as required.

### Formsets

A [formset](https://docs.djangoproject.com/en/1.11/topics/forms/formsets/) is a collection of Django Forms. Each formset has a management form which is used to manage the collection of forms contained in it.

Lets consider an example of a library where in form is required to fill details of **Book**. We are using [Bootstrap](https://getbootstrap.com/) to power our styling.

```
from django import forms

class BookForm(forms.Form):
    name = forms.CharField(
         label=’Book Name’,
         widget= forms.TextInput(attrs={
             ‘class’: ‘form-control’,
             ‘placeholder’: ‘Book Name’
         })
    )
```

### Dynamic adding forms to a formsets

Let’s consider a case wherein we need to add a form to a formset on the frontend. Formset stores data like total number of forms, initial number of forms in a management form. So whenever we want to add a form dynamically, we need to change the total number of forms so that “*Management form has been tampered error is not thrown*”. All the forms in a formset are numbered sequentially, so we need to do a little processing of these numbers to keep the whole structure of a form consistent.

Considering the above **BookForm**, the code for views will be as follows

```
from django.forms import formset_factory

def add_book_info(request):

    template_name = 'library/add_book.html'
    formset_obj = formset_factory(BookForm)

    if request.method == 'GET':
        formset = formset_obj(request.GET or None)
        return render(request, template_name, {'formset': formset})

    elif request.method == 'POST':
        formset = formset_obj(request.POST or None)
        if formset.is_valid():
            for form in formset:
                name = form.cleaned_data.get('name')
                # do necessary action here
                # redirect from here or return success message
        # in case the form is not valid, return as it is
        return render(request, template_name, {
            'formset': formset,
            'last_form_counter': len(formset),
        })
```

In the above code, first, we are creating a formset factory for **BookForm** and then using **GET** or **POST** request to create formset object. This formset will be used in templates to display the forms. Also I am passing length of the formset in context so that I can decide for the last row whether an add more icon should be displayed or remove this icon.

Html code for this form will be written as

```
<form class="form-horizontal" method="GET" action="">
{{ formset.management_form }}
{% for form in formset %}
<div class="row form-row">
    <div class="col-sm-2">
        <label>{{form.name.label}}</label>
    </div>
    <div class="col-sm-4">
        <div class="input-group">
            {{form.name}}
            {# used when the form is submitted with error #}
            {# if value of name field exists then show remove icon else add icon #}
            {# if it is last form then display plus sign #}
            {% if form.name.value and forloop.counter != last_form_counter %}
                <span class="input-group-btn"><button class="btn btn-danger remove-form-row"><span class="glyphicon glyphicon-minus" aria-hidden="true"></span></button></span>
            {% else %}
                <span class="input-group-btn"><button class="btn btn-success add-form-row"><span class="glyphicon glyphicon-plus" aria-hidden="true"></span></button></span>
            {% endif %}
        </div>
    </div>
</div>
{% endfor %}
</form>
```

We have used Bootstrap here to layout our html template. The if condition is used to check whether a add more icon should be displayed or remove this. If *name* field has value and it is not the last form, then display remove this icon, else display add mode icon

Javascript code for this will be written as

```
# make sure jquery is included before

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

Much of the help for above javascript code is taken from this [stackoverflow answer](https://stackoverflow.com/a/669982/2534102). This code registers two handlers on click for `add` and `remove` form.

That’s all for adding forms to a Django formset dynamically.

