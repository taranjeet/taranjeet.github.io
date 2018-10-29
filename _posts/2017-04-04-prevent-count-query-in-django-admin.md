---
layout: post
title: Prevent Count Query in Django Admin
date: 2017-04-04 13:00
tag:
- django
category: blog
author: taranjeet
---

Whenever changelist view of any model is opened in Django Admin, it performs a count query on the table. This count query can be a bottleneck on the database if the number of rows is large(especially in MYSQL, Innodb storage engine). This can be prevented and instead of the count query, a explain query can be fired.

Below is the implementation of the queryset count function.

```
from django.db import models, connections
from django.db.models import Count
from django.db.models.query import QuerySet

class ApproxCountQuerySet(QuerySet):
    """
    Use EXPLAIN query instead of complete COUNT(*) query.
    """

    def count(self):
        query = self.query
        if (
            not query.where and
            query.high_mark is None and
            query.low_mark == 0 and
            not query.select and
            not query.group_by and
            not query.having and
            not query.distinct
        ):
            cursor = connections[self.db].cursor()
            cursor.execute(
                "/*ApproxCountQuerySet*/ EXPLAIN SELECT COUNT(*) FROM `{}`".format(
                    self.model._meta.db_table
                )
            )
            n = cursor.fetchone()[8]
            # can be custom
            if n >= 1000:
                # for a proper string message
                return ApproximateInt(n)
        else:
            return super(ApproxCountQuerySet, self).count()

# custom manager
# only required when we want this feature to be available on the whole model instead
class NoCountManager(models.Manager):
    def get_query_set(self):
        return ApproxCountQuerySet(self.model, using=self._db)


# custom message class
class ApproximateInt(int):
    def __str__(self):
        # just to be clear
        return 'Approximately ' + super(ApproximateInt, self).__str__()
```


Now to use it in Admin class, just write

```
@admin.register(Book)
class BookAdmin(admin.ModelAdmin):

    def get_queryset(self, request):
        qs = super(BookAdmin, self).get_queryset(request)
        return qs._clone(klass=ApproxCountQuerySet)
```

Much of the help is taken from [Adam's great blog post](https://adamj.eu/tech/2014/07/16/extending-djangos-queryset-to-return-approximate-counts/)
