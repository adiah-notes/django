# Making queries

Referring to the following models for a model application:

```py
from datetime import date

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
	mod_date = models.DateField(default=date.today)
	authors = models.ManyToManyField(Author)
	number_of_comments = models.IntegerField(default=0)
	number_of_pingbacks = models.IntegerField(default=0)
	rating = models.IntegerField(default=5)

	def __str__(self):
		return self.headline
```

## Creating objects

```py
>>> from blog.models import Blog
>>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
>>> b.save()
```

This performs an `INSERT` SQL statement behind the scenes.
Django doesn't hit the database until you explicitly call `save()`.

> To create and save an object in a single step, use the `create()` method.

---

## Saving changes to objects

Use `save()`.

Given a `Blog` instance `b5` that has already been saved to the database:

```py
>>> b5.name = 'New name'
>>> b5.save()
```

This performs an `UPDATE` SQL statement behind the scenes. Must call `save()`.


### Saving ForeignKey and ManyToManyField fields

Updating a `ForeignKey` works exactly the same way.

```py
>>> from blog.models import Blog, Entry
>>> entry = Entry.objects.get(pk=1)
>>> cheese_blog = Blog.objects.get(name="Cheddar Talk")
>>> entry.blog = cheese_blog
>>> entry.save()
```

Updating a `ManyToManyField` works differently. use the `add()` method on the field
to add a record to the relation.

```py
>>> from blog.models import Author
>>> joe = Author.objects.create(name="Joe")
>>> entry.authors.add(joe)
```

To add multiple records to a `ManyToManyField` in one go, include multiple arguments 
in the call to `add()`:

```py
>>> john = Author.objects.create(name="John")
>>> paul = Author.objects.create(name="Paul")
>>> george = Author.objects.create(name="George")
>>> ringo = Author.objects.create(name="Ringo")
>>> entry.authors.add(john, paul, george, ringo)
```

---

## Retrieving objects

Construct a `QuerySet` via a `Manager` on the model class.

```py
>>> Blog.objects
<django.db.models.manager.Manager object at ...>
>>> b = Blog(name='Foo', tagline='Bar')
>>> b.objects
Traceback:
    ...
AttributeError: "Manager isn't accessible via Blog instances."
```

> Managers are accessible only via model classes rather than model instances.


### Retrieving all objects

Use the `all()` method.

```py
>>> all_entries = Entry.objects.all()
```

### Retrieving specific objects with filters

Two most common ways to refine a `QuerySet`:

* `filter(**kwargs)`
	Returns a new `QuerySet` containing objects that match the given lookup parameters.

* `exclude(**kwargs)`
	Returns a new `QuerySet` containing objects that do not match the given lookup 
	parameters.


To get a `QuerySet` of blog entries from the year 2006, use `filter()`:

```py 
Entry.objects.filter(pub_date__year=2006)
```

Which is the same as

```py
Entry.objects.all().filter(pub_date__year=2006)
```

#### Chaining filters

The result of refining a `QuerySet` is another `QuerySet` so you can chaing them:

```py
>>> Entry.objects.filter(
...     headline__startswith='What'
... ).exclude(
...     pub_date__gte=datetime.date.today()
... ).filter(
...     pub_date__gte=datetime.date(2005, 1, 30)
... )
```

The result is a `QuerySet` containing all entries with a headline that starts with
"What", that were published between January 30, 2005, and the current day.

---

#### Filtered QuerySets are unique

#### QuerySets are lazy

### Retrieving a single object with `get()`

If you know there is only one object that matches your query, use the `get()` method.
Returns the object directly instead of a `QuerySet`.

```py
>>> one_entry = Entry.objects.get(pk=1)
```

---

### Other QuerySet methods

use the [API Reference](api_reference.md)

---

### Limiting QuerySets

A subset of Python's array-slicing syntax. Similar to `LIMIT` and `OFFSET` clauses.

The first 5 objects (`LIMIT 5`):

```py
>>> Entry.objects.all()[:5]
```

The 6th through 10th objects (`OFFSET 5 LIMIT 5`):

```py
>>> Entry.objects.all()[5:10]
```

Negative indexing is not supported.

--- 

### Field lookups

Specify the meat of an SQL `WHERE` clause.

Basic lookups keyword arguments take the form `field__lookuptype=value`

```py
>>> Entry.objects.filter(pub_date__lte='2006-01-01')
```

Roughly translates to 

```sql
SELECT * FROM blog_entry WHERE pub_date <= '2006-01-01';
```

The field specified has to be the name of a model field.
In a `ForeignKey` you can specify the field name suffixed with `__id`.

```py
>>> Entry.objects.filter(blog__id=4)
```

Check out the [field lookup reference](field_lookup.md)

* **`exact`**

	An exact match

	```py
	>>> Entry.objects.get(headline__exact="Cat bites dog")
	```

	```sql
	SELECT ... WHERE headline = 'Cat bites dog';
	```

	If a lookup type is not provided `exact` is the default

* **`iexact`**

	A case-insensitive match

	```py
	>>> Blog.objects.get(name__iexact="beatles blog")
	```

	would match a `Blog` titled "Beatles Blog", "beatles blog", or even "BeAtlES blOG"

* **`contains`**

	Case-sensitive containment test

	```py
	Entry.objects.get(headline__contains='Lennon')
	```

	```sql
	SELECT ... WHERE headline LIKE '%Lennon%';
	```

---

### Lookups that span relationships

A way to "follow" relationships in lookups, taking care of JOINs.

Use the field name of related fields across models, separated by double underscores until you get to the
field you want.

All `Entry` objects with a `Blog` whose `name` in 'Beatles Blog':

```py
>>> Entry.objects.filter(blog__name='Beatles Blog')
```

The spanning can be as deep as you like.

Works backwards.

Get all `Blog` objects which have at least one `Entry` whose headline contains `Lennon':

```py
>>> Blog.objects.filter(entry__headline__contains='Lennon')
```

#### Spanning multi-valued relationships

Select all blogs containing at least one entry from 2008 having "Lennon" in its headline:

```py
Blog.objects.filter(entry__headline__contains='Lennon', entry__pub_date__year=2008)
```

Any blogs with some entry with 'Lennon' in its headline and some entry from 2008:

```py
Blog.objects.filter(entry__headline__contains='Lennon').filter(entry__pub_date__year=2008)
```

### Filters can reference fields on the model