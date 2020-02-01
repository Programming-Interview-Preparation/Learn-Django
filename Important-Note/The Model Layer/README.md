## Models
It contains the essential fields and behaviors of the data you’re storing. Generally, each model maps to a single database table.

### The basics:

1. Each model is a Python class that subclasses django.db.models.Model
2. Each attribute of the model represents a database field.

### Quick example:

This example model defines a Person, which has a first_name and last_name:

```python
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
```

Once you have defined your models, you need to tell Django you’re going to use those models.
Do this by editing your settings file and changing the INSTALLED_APPS setting to add the name of the module.
```python
INSTALLED_APPS = [
    #...
    'myapp',
    #...
]
```
When you add new apps to INSTALLED_APPS, be sure to run manage.py migrate, optionally making migrations for them first with manage.py makemigrations.