---
title: "Implementing view only permissions in Django"
layout: post
date: 2016-02-14 10:00
tag:
- django
category: blog
author: taranjeet
description: This post implements read only permission built on top of Django ACL.
---

Django ships with a nice and easy to plug [admin](https://docs.djangoproject.com/en/1.11/ref/contrib/admin/) interface. Access to admin panel is managed through Django ACL(Access Control List) using
permission which operates on per model. By default, Django comes with three
types of permission namely add, change and delete.

This post is about implementing **view** only permissions in Django which are
actually missing. **View** only permission come in handy when only view (or read-only)
permission is required for a model.

The part can be split into major sub-parts. Firstly adding permission in a
generic way. Secondly linking the corresponding read-only action with the
permission

Let’s start with implementing permission. Permissions can be added for a model
by using its **Meta** class. For example, to add permission to **Book** model,
the following code can be used


```
from django.db import models
class Book(models.Model)
    name = models.CharField(max_lengt=100)
    class Meta:
        permissions = (
            ('CAN_VIEW_BOOK', 'Can View Book'),
        )

```

This works nicely if we want to add permission for a single or 2–3 models. But
our problem is we want to add this permission for all the models present in the
project. Also, we want to make sure that whenever any new model is added, this
permission should already get added for that model also. So how do we achieve
this?

Well, Django is a mature framework and hence it already is shipped with such
feature “[Signals](https://docs.djangoproject.com/en/1.11/topics/signals/)”.
Signals are something like event emitters which happen whenever any event
occurs. There are many signals, but right now we are only concerned with
[post_migrate](https://docs.djangoproject.com/en/1.11/ref/signals/#post-migrate).
As the name indicates, **post_migrate** runs **after** the migration is run. Also,
we can use **post_model** in a generic way, so that it applies to each and every
model.

To receive a signal, we need to register a receiver function that gets called
when the signal is sent by using the  method.

```
from django.db.models.signals import post_migrate
from django.contrib.auth.models import Permission
from django.contrib.contenttypes.models import ContentType


def add_view_only_permission(sender, **kwargs):
    '''This creates a view only permission for sender'''
    for content_type in ContentType.objects.all():
        codename = 'can_view_%s' % content_type.model
        name = 'Can View %s' % content_type.name
        if not Permission.objects.filter(
            content_type=content_type,
            codename=codename):
            Permission.objects.create(
            content_type=content_type,
            codename=codename,
            name=name)

post_migrate.connect(add_view_only_permission)

```

This code can be placed in any file, but it is mostly placed in the file, which gets
loaded initially as signals to be cached also. There is a question on
[stackoverflow](https://stackoverflow.com/questions/2719038/where-should-signal-handlers-live-in-a-django-project)
regarding the placement of signals in a project. As of now, I am placing it in
**<any_app>/models.py**.

Summarizing, the above code will create a default **view only** permission for
each and every model. **post_migrate** signal is emitted whenever migrations
are run. One very important thing to note is that it only creates permission for
a model in a generic way, not a view only permission.

Now comes the second part, wherein we will write our action corresponding to
the above created permission. For this, we will be using **ModelAdmin** class,
since we are dealing with the admin part.

There is a function called *get_readonly_fields, get_list_display* and
*submit_row* which can be overridden and used to fit our use case. Code for
**ViewOnlyAdmin** class will be as follows.

```
class ViewOnlyAdmin(admin.ModelAdmin):
    '''This class makes fields read only'''

    def _is_user_view_only_type(self, perm, request):
        return request.user.has_perm(str(perm)) and request.user.is_superuser

    def get_readonly_fields(self, request, obj=None):
        class_name = self.__class__.__name__.replace('Admin', '').lower()
        for permission in request.user.get_all_permissions():
            head, sep, tail = permission.partition('.')
            perm = 'can_view_%s' % class_name

            if str(perm) == str(tail):
                if self._is_user_view_only_type(perm, request):
                    return flatten_fieldsets(self.declared_fieldsets)
                else:
                    return list(set(
                        [field.name for field in self.opts.local_fields] +
                        [field.name for field in self.opts.local_many_to_many]))
        return self.readonly_fields

    def get_list_display(self, request):
        list_display = super(ViewOnlyAdmin, self).get_list_display(request)

        app_label, model_name = self.opts.app_label.lower(), self.model._meta.object_name.lower()

        perm = '%s.can_view_%s' % (app_label, model_name)

        if self._is_user_view_only_type(perm, request):
            self.list_editable = ()
        return list_display

    @register.inclusion_tag('admin/submit_line.html', takes_context=True)
    def submit_row(context):
        ctx = original_submit_row(context)
        app_name = context['app_label']
        model_name = context['opts'].model_name

        for permission in context['request'].user.get_all_permissions():
            head, sep, tail = permission.partition('.')
            perm = 'can_view_%s' % model_name
            if str(perm) == str(tail):
                if (context['request'].user.has_perm(str(permission)) and
                       not context['request'].user.is_superuser):
                    ctx.update({
                        'show_save_and_add_another' : False,
                        'show_save_and_continue'    : False,
                        'show_save'                 : False,
                        'show_save_as_new'          : False,
                        })
        return ctx

```

To use this class for your models, you need to extend your model’s admin class
from this class.

```
from libs.admin import ViewOnlyAdmin
class BookAdmin(ViewOnlyAdmin):
    pass
```

To use this permission, one needs to assign **two** permission namely
**can_change** and **can_view** for a model. This is because **can_view** is a
negative permission and it only works if the **can_change** is present. So one
can simply infer that **can_view** negates the change effect of **can_change**
to turn every field into a read-only field.

The above code is one way of implementing view only permission in Django. I am
sure there can be other interesting ways to implement it.


