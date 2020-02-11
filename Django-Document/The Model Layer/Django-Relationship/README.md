## Relationships

Django offers ways to define the three most common types of database relationships: many-to-one, many-to-many and one-to-one.

#### Many-to-one relationships

* To define a many-to-one relationship, use django.db.models.ForeignKey. 
* ForeignKey requires a positional argument: the class to which the model is related.

For example, if a Car model has a Manufacturer – that is, a Manufacturer makes multiple cars but each Car only has one Manufacturer.

```python
from django.db import models

class Manufacturer(models.Model):
    # ...
    pass

class Car(models.Model):
    manufacturer = models.ForeignKey(Manufacturer, on_delete=models.CASCADE)
    # ...
```

#### Many-to-many relationships

* To define a many-to-many relationship, use ManyToManyField. 
* ManyToManyField requires a positional argument: the class to which the model is related.

For example, if a Pizza has multiple Topping objects – that is, a Topping can be on multiple pizzas and each Pizza has multiple toppings.

```python
from django.db import models

class Topping(models.Model):
    # ...
    pass

class Pizza(models.Model):
    # ...
    toppings = models.ManyToManyField(Topping)
```

#### One-to-one relationships

* To define a one-to-one relationship, use OneToOneField. 
* OneToOneField requires a positional argument: the class to which the model is related.

