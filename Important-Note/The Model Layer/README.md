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

