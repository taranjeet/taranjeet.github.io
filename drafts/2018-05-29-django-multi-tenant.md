
## Steps to achieve multi-tenancy in Django

Lets us consider an application named djangolibrary, hosted at djangolibrary.com. Any company can signup and hosts its library. Consider that a company named `abc` wants to host its library. It will be hosted at `abc.djangolibrary.com`. Below are the steps to achieve multi-tenancy in Django for this application.

* First we need to add a middleware, which will attach subdomain value to the request. The middleware can be defined as

```
class AttachSubDomain(object):

    def process_request(self, request):
        domain = request.META.get('HTTP_HOST') or request.META.get('SERVER_NAME')
        pieces = domain.split('.')
        subdomain = ".".join(pieces[:-2])
        setattr(request, 'subdomain', subdomain)
        return None

```

This middleware can be enable in settings by including in `MIDDLEWARE_CLASSES` like

```
MIDDLEWARE_CLASSES = [
    ...,
    'path.to.AttachSubDomain',
]
```

This middleware attaches the value of subdomain on request. It can be accessed by `request.subdomain`


* Now we need to write a generic model, which will add support for subdomain in each table. Each table will contain a column named `company_id` which will be a reference to the original company instance.

```
# models.py

class Company(models.Model):
    short_code = models.CharField(max_length=255, help_text='Unique Id of the company. This is the value of subdomain')


class CompanyModel(models.Model);
    '''this is the generic model from which every model needs to inherit'''
    company = models.ForeignKey(Company, blank=True, null=True, db_index=True)

    class Meta:
        abstract = True
```

Lets say, we are creating a `Team` model, which is used to denote team for a company. This model will be inherited from `CompanyModel`, like

```
from path.to.where.CompanyModel.lives import CompanyModel

class Team(CompanyModel):
    name = models.CharField(max_length=100, unique=True)
```

This way, any other model can be inherited from `CompanyModel`.

* Once every model is inherited from `CompanyModel`, we need to add a generic support to fetch and save the value of subdomain in `company` column. For this we need to write a custom queryset and manager.

```
from django.db import models

class CompanyQueryset(models.query.QuerySet):

    def own_employees(self):
        # how get_request provides request, this is explained below
        request = get_request()
        if not request.subdomain:
            return self.none()
        return self.filter(company__short_code=request.subdomain)


class CompanyManager(models.Manager):

    def get_queryset(self):
        return CompanyQueryset(self.model, using=self._db)

    def own_employees(self):
        return self.get_queryset().own_employees()
```

Now this custom manager needs to included in each model, which is inherited from `CompanyModel`.

```

class Team(CompanyModel):

    name = models.CharField(max_length=100, unique=True)

    # keeping default model manager as well
    objects = models.Manager()
    company_manager = CompanyManager()

```

This can be used like `Team.company_manager.own_employees()`

* Till now, we have added support to automatically add where clause on subdomain, but if any new instance of model is saved, it will be saved without the subdomain reference. For saving reference to subdomain in a generic way, we will use `pre_save` signal

```
@receiver(pre_save)
def save_company_name(sender, instance, **kwargs):
    # as this runs for every model
    if not issubclass(sender, CompanyModel):
        return
    # how get_request provides request, this is explained below
    request = get_request()
    try:
        company = Company.objects.get(short_code=request.subdomain)
        instance.company = company
    except Exception as e:
        company = None
```

This signal runs before any model is saved. It checks if the sender is a subclass of `CompanyModel` and if it is, then it saves subdomain reference to the concerned model.

* This completes most of the part of multi-tenancy when models are concerned. One important thing that is required frequently will be to access current subdomain in the template. For this, a context processor can be added.

```
def get_company_name(request):

    try:
        company_name = request.subdomain
    except Exception as e:
        company_name = None

    return {
        'company_name': company_name
    }
```

Use this context processor in the settings like

```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [join(APP_DIR, 'templates'), ],
        'OPTIONS': {
            'context_processors': [
                ...
                'teamclone.base.context_processor.get_company_name',
            ],
        },
    },
```

Now to access subdomain value in the templates, we can use `{{company_name}}`.

* Since we are using middleware to attach subdomain value to the request, many a times we will need to access request where it is not available. It can be made available by using the [django-contrib-requestprovider](https://pypi.org/project/django-contrib-requestprovider/) package


