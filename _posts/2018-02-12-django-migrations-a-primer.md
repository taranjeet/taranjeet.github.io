---
title: "Django Migrations&#58; A Primer"
layout: post
date: 2018-02-12 20:00
tag:
- django
category: blog
author: taranjeet
description: This post is about migrations present in django.
---

Migrations in Django are a way of propagating changes made in the model into the database schema.

Official [Django docs](https://docs.djangoproject.com/en/2.0/topics/migrations/) summarizes Migration as
> Migrations are Django’s way of propagating changes you make to your models (adding a field, deleting a model, etc.) into your database schema

Migrations were introduced as a [built-in functionality](https://docs.djangoproject.com/en/2.0/releases/1.7/#schema-migrations) in Django version 1.7. Migrations are created manually by running certain commands but they automate the task of applying changes to the database. We can create migration for any changes in model, like

* Add a new model.

* Add a particular field in model

* Update field attributes in model

* Delete a field in model

* Rename a field in model

Automatically generated migrations are used to apply changes to the database schema. To apply changes in data, one needs to manually write the [data migrations](https://docs.djangoproject.com/en/2.0/topics/migrations/#data-migrations).

Let’s consider an example of a Library system and walk through the whole example of creating and applying migrations. Consider that our project is django_library and there is an app called library in it. We are right now focussing only on the Book model.

{% gist bb6985599297af297e51de4f90b74d43 %}

### How to create migrations

To create the migration, Django provides a management command called makemigrations. It can be run as

{% gist 993374c49036438d9ea28bf7c02a1218 %}

Here `name_of_app` is optional.

Consider our `library` app, migrations for it can be created by running the following command.

{% gist 4d804d2fd9b26a7e7a03942a8cd0371e %}

A file called `0001_initial.py` will be created in `library/migrations` directory.

Since this is the first migration for an app, Django names it as `initial` migration.

For every other migration, Django names it using date and time like `0007_auto-20180208_1613.py`

It is always advised to create a named migration since it becomes easier to infer just by looking at the name of what the migration is about.

We can create named migration by running the following command

{% gist 77c41772c53542835883cec260387427 %}

A small note here, the name of migration should not exceed 50 characters.

Let’s add another field named `publisher` in our `Book` model. Our model will now look like

{% gist 70d579f0391896cc110e279027a120e4 %}

To create named migration for this, we need to run `makemigrations` as

{% gist f7c6836d885e81ee558984ac1a3c2ef4 %}

### How to apply a migrations

Once the migration, we need to apply them, so that changes can be reflected in the databases as well.

To apply migration, we need to run `migrate` command as follows

{% gist aa83dce04442aea709150fc2a3117ab8 %}

By running this command, all the unapplied migration are applied to the database.

In our case for `Book` model, both `0001_initial` and `0002_add_publisher` will be applied.

Whenever any migration is applied, Django records this in a table called `django_migrations`.

### How to undo migrations

Many times, we need to undo the migration as well. Or in simple words, we need to undo the changes which are done to the database schema by undoing that migration

Undoing a migration can be done by using migrate command, but we need to pass app name as well as the name of migration at which we want to rollback.

{% gist ba39e498a4d5b07f5898817823bbe3ae %}

Let’s remove `publisher` field from our `Book` model. This can be done by undoing `0002_add_publisher` migration.
Since we need to undo this migration, we will pass `0001_initial` as an argument to migrate. The command will look like

{% gist dffb70169360634c9694637f8afff7e3 %}

By running this command, `publisher` field will be removed from `book` table in the database schema.

### How to fake migrations

Many times, We just need to run the migration without actually changing the database schema. It happens when we have taken a dump from our staging or production database.
To deal with such cases, Django provides a way called **fake migration**, which applies the migration but does not affect the database schema.

In simpler words, for each migration which is faked, an entry is created in `django_migrations` table but no changes are done in the database schema.

### How to clear migration for an app

Django provides with a built-in functionality, which allows us to clear all the migration for a particular app. To clear migration for an app, we need to run the following command

{% gist 31c6ccf71e76b24cd0ac365742d4c24f %}

Here zero tells migrate command to clear the migration for a particular app.

### How to list all migrations

Django provides a built-in command called `showmigrations` to list and display all migrations. It can be run as

{% gist 23e14f0a20f7c9efad65bd0fb6c2e84d %}

When this command is run for our django_library application, we get the following output.

{% gist 10c64179334dc97ed04ba494fe6a3fc5 %}

Here `[X]` indicates that the migration has been applied and `[]` indicates that the migration has not been applied. In our case, `0002_add_publisher` is not applied.

### Workflow

The basic process of creating and applying migration is the following.

* First, make changes to the model. This can be adding a new model or updating/removing any fields from the model.

* Create migrations by running `makemigrations`.

* Apply migration by running `migrate`.

### Role of django_migrations table

Whenever any migration is applied or unapplied, Django records this in a table called `django_migrations`. This table stores the name of the app, name and of migration and a datetime field, which tells when the migration was applied.

This table is used by Django to keep track of which migration to apply and which migrations have been applied.

A very likely doubt that arises is no changes are reflected in the database schema when a migration file is manually changed and applied again. This happens because Django has already created an entry for it in `django_migrations` table. One way to deal with this situation can be deleting the concerned row and then running the migrations again(this is not the official way, but it works).

### What is a migration file?

Whenever `makemigrations` is run, a python file is created. This file contains a class called `Migration` which contains two attributes

* dependencies

This is an important attribute, which tells that this migration depends on the previous one. It actually allows the migration to be applied in a linear fashion.

* operations

This is an array, which contains all the changes that need to be applied to the database schema.

One important thing to note here is that this migration file needs to be added to the Version Control System(git) as well.

### Practical Use Cases

* Migrations give us the flexibility of propagating database related changes easily across a team. Migration created by a developer can be added to VCS, which later can be applied by another dev, simply by running the `migrate` command.

* Another way of using migration can be to apply changes in the development environment and fake migrations on the production environment. This is especially helpful in a large organization, where the database is managed specifically by Database Administrators.

### Advantages

* Migration gives us the flexibility to track database changes in version control system. This way it becomes easier for us to see the how the database changed over a period of time.

* Migration offloads the process of manually applying a set of changes to the database. More than one migration can be applied at once by simply running the `migrate` command.

### Summary

Django provides a built-in functionality of propagating changes made in the model to the database schema. Migrations first need to be created using `makemigrations`. Once created, migrations can be applied by running `migrate` command. Creating a migration and applying a migration both are two different processes. Automatically generated migrations are used to apply changes only to the database schema. Migrations for data needs to be manually written and applied.

