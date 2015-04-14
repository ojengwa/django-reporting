**NOTE: django reporting is in beta version, what means that while working, it lacks a lot of features, and has not been well tested.**

# Introduction #

Django Reporting System allows you to create dynamic reports for your models, consolidating and aggregating data, filtering and sorting it.

It requires Django 1.1, since it uses Django's ORM aggregation.


## Add to exiting django project ##

settings.py
```
INSTALLED_APPS = (
    [...]
    'reporting',
    'django.contrib.admin', # admin has to go before reporting in order to have links to the reports on the admin site

```

urs.py
```
from django.conf.urls.defaults import *
from django.contrib import admin
import reporting                                           # import the module

admin.autodiscover()
reporting.autodiscover()                                   # autodiscover reports in applications

urlpatterns = patterns('',
    [...]
    (r'^reporting/', include('reporting.urls')),
)
```

for more details see a 'samples' directory inside repository

# Configure report #
Let's say you have the follwing schema:

models.py:
```
class Department(models.Model):
    [...]
    
class Occupation(models.Model):
    [...]

class Person(models.Model):
    name = models.CharField(max_length=255)                         # we won't use it in a summary report
    occupation = models.ForeignKey(Occupation)                      # we'll be able to group and to filter by both occupation and country
    department = models.ForeignKey(Department)                      # we'll be able to group and to filter by department and it leader
    country = models.ForeignKey(Country)
    birth_date = models.DateField()                                 # we'll be able to filter by year
    salary = models.DecimalField(max_digits=16, decimal_places=2)   # we'll sum and calculate average for salary and expenses 
    expenses = models.DecimalField(max_digits=16, decimal_places=2)
```

in your application create a **reports.py**

reports.py:
```
import reporting
from django.db.models import Sum, Avg, Count
from models import Person

class PersonReport(reporting.Report):
    model = Person
    verbose_name = 'Person Report'
    annotate = (                    # Annotation fields (tupples of field, func, title)
        ('id', Count, 'Total'),     # example of custom title for column 
        ('salary', Sum),            # no title - column will be "Salary Sum"
        ('expenses', Sum),
    )
    aggregate = (                   # columns that will be aggregated (syntax the same as for annotate)
        ('id', Count, 'Total'),
        ('salary', Sum, 'Salary'),
        ('expenses', Sum, 'Expenses'),
    )
    group_by = [                   # list of fields and lookups for group-by options
        'department',
        'department__leader', 
        'occupation', 
    ]
    list_filter = [                # This are report filter options (similar to django-admin)
       'occupation',
       'country',
    ]
    
    detail_list_display = [        # if detail_list_display is defined user will be able to see how rows was grouped  
        'name', 
        'salary',
        'expenses', 
    ]

    date_hierarchy = 'birth_date' # the same as django-admin


reporting.register('people', PersonReport) # Do not forget to 'register' your class in reports
```
for more details see a 'samples' projects inside repository