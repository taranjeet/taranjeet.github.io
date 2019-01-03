---
title: "Django &#58; How to add migration for unique keys"
layout: post
date: 2019-01-03 17:30
tag:
- django
category: blog
author: taranjeet
description: This post is about adding migration for a field which should have unique values.
---

This post is about adding migration for a field which should have unique values.

Lets us consider a `UserProfile` model.

```python
# models.py

from django.db import models
from django.contrib.auth.models import User


class UserProfile(models.Model):

    user = models.OneToOneField(User,
                                primary_key=True,
                                related_name='profile',
                                on_delete=models.CASCADE)
    contact_number = models.CharField(max_length=10, null=True, blank=True)

    class Meta:
        db_table = 'user_profile'

    def __str__(self):
        return self.user.username
```

Now suppose that, we need to add a field which should have unique values. Lets consider that we want to add a new column called `user_id_custom` which is a `UUIDField` and should have unique values of `UUIDField`. The most simple piece of code to accomplish this would look like

```python
# profiles/models.py
import uuid

from django.db import models
from django.contrib.auth.models import User


class UserProfile(models.Model):

    user = models.OneToOneField(User,
                                primary_key=True,
                                related_name='profile',
                                on_delete=models.CASCADE)
    contact_number = models.CharField(max_length=10, null=True, blank=True)
    # this is new field
    user_id_custom = models.UUIDField(default=uuid.uuid4, unique=True)

    class Meta:
        db_table = 'user_profile'

    def __str__(self):
        return self.user.username
```

Now lets create a migration file for this. We generally prefer named migration(max length is 50), so that it becomes easy to track later what the migration was about.

```sh
python manage.py makemigrations profiles --name=add_unique_user_id_custom
```

Here `profiles` is the name of the app. It can be any value according to your usecase. The migration is generated successfully. It can be found in

```sh
tree profiles

profiles
├── __init__.py
├── admin.py
├── apps.py
├── migrations
│   ├── 0001_initial.py
│   ├── 0002_add_unique_user_id_custom.py
│   ├── __init__.py
├── models.py
├── tests.py
└── views.py
```

Now lets run `migrate` to apply changes to the database.

```sh
python manage.py migrate
```

Now when the migration is applied, we get an error. This error may vary, but the error is an `IntegrityError`.

```sh
django.db.utils.IntegrityError: (1062, "Duplicate entry '3c4e98944d1f4474ab50917496124f4b' for key 'user_id_custom'")
```

One way to overcome this problem can be the following steps

* First add a field and set its value to null
* Next write custom code to generate unique uuids
* At last, alter the field to be unique.

The above listed is a manual process where in two migration files will be generated and a custom code(data migration) needs to be written to store unique uuids. This approach can be opted, but since django migration allows us to run custom python code using [RunPython](https://docs.djangoproject.com/en/2.1/ref/migration-operations/#runpython), second way can be an automated way of doing this. The second way will follow the same set of sequence, but only a single migration file will be generated.

In our case, we will be updating the generate migration file (here `0002_add_unique_user_id_custom.py`) and adding some custom code to it.

```python
# profiles/migrations/0002_add_unique_user_id_custom.py

from django.db import migrations, models
import uuid


def create_uuid(apps, schema_editor):
    UserProfile = apps.get_model('profiles', 'UserProfile')
    for user_profile in UserProfile.objects.all():
        user_profile.user_id_custom = uuid.uuid4()
        user_profile.save(update_fields=['user_id_custom'])


class Migration(migrations.Migration):

    dependencies = [
        ('profiles', '0001_initial'),
    ]

    operations = [
        migrations.AddField(
            model_name='userprofile',
            name='user_id_custom',
            field=models.UUIDField(blank=True, null=True, unique=True),
        ),
        migrations.RunPython(create_uuid),
        migrations.AlterField(
            model_name='userprofile',
            name='user_id_custom',
            field=models.UUIDField(unique=True, editable=False)
        )
    ]
```

Now we can apply the above migration by running the `migrate` command

```
python manage.py migrate
```

And this works perfectly.

### Summary

This post is about how can we add a field with unique values to an existing django model. It adds some custom code in the generate migration file and skips detailing the manual way of creating step by step migration file. This post can be applied for any field which should have unique values.

### References

* [Stackoverflow: Django migration with uuid field generates duplicated values](https://stackoverflow.com/questions/35281003/django-migration-with-uuid-field-generates-duplicated-values)
