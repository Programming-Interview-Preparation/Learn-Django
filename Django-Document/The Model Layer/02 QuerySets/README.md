#### Making queries

Once you’ve created your data models, Django automatically gives you a database-abstraction API that lets you create, retrieve, update and delete objects. 

```python
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def __str__(self):
        return self.name

class Author(models.Model):
    name = models.CharField(max_length=200)
    email = models.EmailField()

    def __str__(self):
        return self.name

class Entry(models.Model):
    blog = models.ForeignKey(Blog, on_delete=models.CASCADE)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField()
    authors = models.ManyToManyField(Author)
    number_of_comments = models.IntegerField()
    number_of_pingbacks = models.IntegerField()
    rating = models.IntegerField()

    def __str__(self):
        return self.headline
```

#### Creating objects

* A model class represents a database table, and an instance of that class represents a particular record in the database table.

- To create an object, instantiate it using keyword arguments to the model class, then call save() to save it to the database.

```python
# Assuming models live in a file mysite/blog/models.py
from blog.models import Blog
b = Blog(name='Hasan Mahmud', tagline='Software Engineer.')
b.save()
```
This performs an INSERT SQL statement behind the scenes. Django doesn’t hit the database until you explicitly call save().
The save() method has no return value.

#### Saving changes to objects

* To save changes to an object that’s already in the database, use save().

Given a Blog instance b5 that has already been saved to the database, this example changes its name and updates its record in the database:
```python
b5.name = 'New name'
b5.save()
```
This performs an UPDATE SQL statement behind the scenes. Django doesn’t hit the database until you explicitly call save().

#### Saving ForeignKey and ManyToManyField fields

* Updating a ForeignKey field works exactly the same way as saving a normal field – assign an object of the right type to the field in question. 
 
 This example updates the blog attribute of an Entry instance entry, assuming appropriate instances of Entry and Blog are already saved to the database (so we can retrieve them below):

 ```python
from blog.models import Blog, Entry
entry = Entry.objects.get(pk=1)
cheese_blog = Blog.objects.get(name="Cheddar Talk")
entry.blog = cheese_blog
entry.save()
 ```

* Updating a ManyToManyField works a little differently – use the add() method on the field to add a record to the relation. This example adds the Author instance joe to the entry object:

 ```python
from blog.models import Author
joe = Author.objects.create(name="Joe")
entry.authors.add(joe)
 ```
 To add multiple records to a ManyToManyField in one go, include multiple arguments in the call to add(), like this:
 ```python
john = Author.objects.create(name="John")
paul = Author.objects.create(name="Paul")
george = Author.objects.create(name="George")
ringo = Author.objects.create(name="Ringo")
entry.authors.add(john, paul, george, ringo)
 ```

#### Retrieving objects

- To retrieve objects from your database, construct a QuerySet via a Manager on your model class.
- A QuerySet represents a collection of objects from your database.
- It can have zero, one or many filters.

Note: Managers are accessible only via model classes, rather than from model instances, to enforce a separation between “table-level” operations and “record-level” operations.

#### Retrieving all object

The simplest way to retrieve objects from a table is to get all of them. To do this, use the all() method on a Manager:

```python
all_entries = Entry.objects.all()
```
The all() method returns a QuerySet of all the objects in the database.

#### Retrieving specific objects with filters

filter(**kwargs) -> Returns a new QuerySet containing objects that match the given lookup parameters.

exclude(**kwargs) -> Returns a new QuerySet containing objects that do not match the given lookup parameters.

For example, to get a QuerySet of blog entries from the year 2006, use filter() like so:
```python
Entry.objects.filter(pub_date__year=2006)
```
With the default manager class, it is the same as:
```python
Entry.objects.all().filter(pub_date__year=2006)
```

#### Chaining filters

```python
Entry.objects.filter(
...     headline__startswith='What'
... ).exclude(
...     pub_date__gte=datetime.date.today()
... ).filter(
...     pub_date__gte=datetime.date(2005, 1, 30)
... )
```
This takes the initial QuerySet of all entries in the database, adds a filter, then an exclusion, then another filter. The final result is a QuerySet containing all entries with a headline that starts with “What”, that were published between January 30, 2005, and the current day.

#### Filtered QuerySets are unique

Each time you refine a QuerySet, you get a brand-new QuerySet that is in no way bound to the previous QuerySet. Each refinement creates a separate and distinct QuerySet that can be stored, used and reused.

```python
q1 = Entry.objects.filter(headline__startswith="What")
q2 = q1.exclude(pub_date__gte=datetime.date.today())
q3 = q1.filter(pub_date__gte=datetime.date.today())
```

These three QuerySets are separate. The first is a base QuerySet containing all entries that contain a headline starting with “What”. The second is a subset of the first, with an additional criteria that excludes records whose pub_date is today or in the future. The third is a subset of the first, with an additional criteria that selects only the records whose pub_date is today or in the future. The initial QuerySet (q1) is unaffected by the refinement process.

#### QuerySets are lazy

QuerySets are lazy – the act of creating a QuerySet doesn’t involve any database activity. You can stack filters together all day long, and Django won’t actually run the query until the QuerySet is evaluated. Take a look at this example:

```python
q = Entry.objects.filter(headline__startswith="What")
q = q.filter(pub_date__lte=datetime.date.today())
q = q.exclude(body_text__icontains="food")
print(q)
```

Though this looks like three database hits, in fact it hits the database only once, at the last line (print(q)). In general, the results of a QuerySet aren’t fetched from the database until you “ask” for them. When you do, the QuerySet is evaluated by accessing the database. 

#### Retrieving a single object with get()

filter() will always give you a QuerySet, even if only a single object matches the query - in this case, it will be a QuerySet containing a single element.

If you know there is only one object that matches your query, you can use the get() method on a Manager which returns the object directly:

```python
one_entry = Entry.objects.get(pk=1)
```

Note that there is a difference between using get(), and using filter() with a slice of [0]. If there are no results that match the query, get() will raise a DoesNotExist exception. 

#### Other QuerySet methods

Most of the time you’ll use all(), get(), filter() and exclude() when you need to look up objects from the database. 

#### Limiting QuerySets

Use a subset of Python’s array-slicing syntax to limit your QuerySet to a certain number of results. This is the equivalent of SQL’s LIMIT and OFFSET clauses.

For example, this returns the first 5 objects (LIMIT 5):

```python
Entry.objects.all()[:5]
```

This returns the sixth through tenth objects (OFFSET 5 LIMIT 5):

```python
Entry.objects.all()[5:10]
```

Negative indexing (i.e. Entry.objects.all()[-1]) is not supported.

#### Field lookups

Field lookups are how you specify the meat of an SQL WHERE clause. They’re specified as keyword arguments to the QuerySet methods filter(), exclude() and get().

Basic lookups keyword arguments take the form field__lookuptype=value. (That’s a double-underscore). For example:

```python
Entry.objects.filter(pub_date__lte='2006-01-01') # SELECT * FROM blog_entry WHERE pub_date <= '2006-01-01';
```

The field specified in a lookup has to be the name of a model field. There’s one exception though, in case of a ForeignKey you can specify the field name suffixed with _id. In this case, the value parameter is expected to contain the raw value of the foreign model’s primary key. For example:

```python
Entry.objects.filter(blog_id=4) # If you pass an invalid keyword argument, a lookup function will raise TypeError.
```
```python
Entry.objects.get(headline__exact="Cat bites dog") # SELECT ... WHERE headline = 'Cat bites dog';
```
```python
Blog.objects.get(id__exact=14)  # Explicit form
Blog.objects.get(id=14)         # __exact is implied
```
```python
Blog.objects.get(name__iexact="beatles blog")
# Would match a Blog titled "Beatles Blog", "beatles blog", or even "BeAtlES blOG".
```
```python
# Case-sensitive containment test. For example:
Entry.objects.get(headline__contains='Lennon') # SELECT ... WHERE headline LIKE '%Lennon%';
# Note this will match the headline 'Today Lennon honored' but not 'today lennon honored'.
# There’s also a case-insensitive version-> icontains, startswith, endswith, istartswith, iendswith.
```

#### Lookups that span relationships

Django offers a powerful and intuitive way to “follow” relationships in lookups, taking care of the SQL JOINs for you automatically, behind the scenes. To span a relationship, use the field name of related fields across models, separated by double underscores, until you get to the field you want.

This example retrieves all Entry objects with a Blog whose name is 'Beatles Blog':
```python
Entry.objects.filter(blog__name='Beatles Blog')
```
It works backwards, too. To refer to a “reverse” relationship, use the lowercase name of the model.
This example retrieves all Blog objects which have at least one Entry whose headline contains 'Lennon':
```python
Blog.objects.filter(entry__headline__contains='Lennon')
```

#### Spanning multi-valued relationships


#### Filters can reference fields on the model

#### The pk lookup shortcut

#### Escaping percent signs and underscores in LIKE statements

#### Caching and QuerySets

#### Complex lookups with Q objects

#### Comparing objects

#### Deleting objects

#### Copying model instances

#### Updating multiple objects at once

#### Related objects


#### Many-to-many relationships


#### One-to-one relationships


#### How are the backward relationships possible?


#### Queries over related objects




