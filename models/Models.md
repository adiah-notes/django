# Models

A model contains the essential fields and behaviors of the data being stored. Each maps to a single db table.

* Each model is a Python class 
* Each attribute represents a database field

# Fields

Fields are the only required part of a model. Fields are specified by class attributes.

```py
from django.db import models

class Musician(models.Model):
	first_name = models.CharField(max_length=50)
	last_name = models.CharField(max_length=50)
	instrument = models.CharField(max_length=100)

class Album(models.Model):
	artist = models.ForeignKey(Musician, on_delete=models.CASCADE)
	name = mdels.CharField(max_length=100)
	release_date = models.DateField()
	num_stars = models.IntegerField()
```

## Field types

Each is an instance of the appropriate `Field` class. Determines:

* the column type, what kind of data to store (eg. INTEGER, VARCHAR, TEXT)
* the default HTML widget to use when rendering a form field (e.g. `<input type="text">`, `<select>`)
* the minimal validation requirements used in Django's admin and automatically generated forms.


## Field options

Each field has a certain set of field-specific arguments.
Check the [Model field reference](Model_field_reference.md)

Here are the most often used:

* **`null`**: 

	If `True`, Django will store empty values as `NULL` in the database. Default is `False`.

* **`blank`**: 

	If `True`, the field is allowed to be blank. Default is `False`

	**nb** `blank` is validation-related, while `null` is just database related. Relates to if form will
	have the field required or not.

* **`choices`**

	A sequence of 2-tuples to use as choices for the field. The default widget will be a select box.

	```py
	YEAR_IN_SCHOOL_CHOICES = [
		('FR', 'Freshman'),
		('SO', 'Sophomore'),
		('JR', 'Junior'),
		('SR', 'Senior'),
		('GR', 'Graduate'),
	]
	```

	The first element is the value stored in the database. The second element is displayed by the field's form.

	The display value for a field with `choices` can be accessed using the `get_FOO_display()` method:

	```py
	from django.db import models

	class Person(models.Model):
		SHIRT_SIZES = (
			('S', 'Small'),
			('M', 'Medium'),
			('L', 'Large'),
		)
		name = models.CharField(max_length=60)
		shirt_size = models.CharField(max_length=1, choices=SHIRT_SIZES)
	```

	```
	>>> p = Person(name="Fred Flintstone", shirt_size="L")
	>>> p.save()
	>>> p.shirt_size
	'L'
	>>> p.get_shirt_size_display()
	'Large'
	```

	Can also use enumeration:

	```py
	class Runner(models.Model):
		MedalType = models.TextChoices('MedalType', 'GOLD SILVER BRONZE')
		name = models.CharField(max_length=60)
		medal = models.CharField(blank=True, choices=MedalType.choices, max_length=10)
	```

* **`default`**

	the default value for the field. Value or a callable object. Will be called every time a new object is created.

* **`help_text`**

	Extra "help" text to be displayed with the form widget. Useful for documentation even if field isn't used on
	a form.

* **`primary_key`**

	If `True`, this is the primary key for the model.

	If not specified for any fields in the model, Django automatically adds a primary key in an `IntegerField`.

	The primary key field is read only. If you change the value of the pk and save it, a new object will be created
	alongside the ole one.:

	```py
	class Fruit(models.Model):
		name = models.CharField(max_length=100, primary_key=True)
	```

	```
	>>> fruit = Fruit.objects.create(name='Apple')
	>>> fruit.name = 'Pear'
	>>> fruit.save()
	>>> Fruit.objects.values_list('name', flat=True)
	<QuerySet ['Apple', 'Pear']>
	```

* **`unique`**

	If `True`, this field will be unique throughout the table.


## Verbose field names

Each field type except for `ForeignKey`, `ManyToManyField` and `OneToOneField`, takes an optional first positional
argument. A verbose name. If not given, Django will automatically create it.

The verbose name is "person's first name":
```py
first_name = models.CharField("person's first name", max_length=30)
```

The verbose name is "first name":
```py
first_name = models.CharField(max_length=30)
```

The other fields require the first argument to be a model class, so use `verbose_name` keyword argument.
```py
poll = models.ForeignKey(Poll, on_delete=models.CASCADE, verbose_name="the related poll",)
sites = models.ManyToManyField(Site, verbose_name="list of sites")
place = models.OneToOneField(Place, on_delete=models.CASCADE, verbose_name="related place")
```

No need to capitalize the first letter.


## Relationships


### Many-to-one relationships

Use `django.db.models.ForeignKey`. Set it like any other Field type.

`ForeignKey` requires a positional argument: the class to which the model is related.

If a `Car` model has a `Manufacturer` -> a `Manufacturer` makes multiple cars but each `Car` only has one 
`Manufacturer`:

```py
from django.db import models

class Manufacturer(models.Model):
	# ...
	pass

class Car(models.Model):
	manufacturer = models.ForeignKey(Manufacturer, on_delete=models.CASCADE)
	# ...
```

Suggested to make the name of the field the name of the model, lowercase.

> See also [Many-to-one model](many_to_one.md)


### Many-to-many relationships

Use `ManyToManyField`.

Example if a `Pizza` has multiple `Topping` objects -> a `Topping` can be on multiple pizzas and each `Pizza` has 
multiple toppings:

```py
from django.db import models

class Topping(models.Model):
	# ...
	pass

class Pizza(models.Model):
	# ...
	toppings = models.ManyToManyField(Topping)
```

Can create [recursive relationships](recursive.md)

Suggested to name the field a plural describing the set of related model objects.

The field should be on only one of the models, doesn't matter which.

Generally, `ManyToManyField` instances should go in the object that's going to be edited on a form.
It's more natural to think about a pizza having toppings as a opposed to a topping being on multiple pizzas.
The `Pizza` form would let users select the toppings.

> See [many to many](many_to_many.md)


#### Extra fields on many-to-many relationships

Sometimes need to associate data with the relationship between two models.

Django allows you to specify the model that will be used to govern the many-to-many relationship. You can then put
extra fields on the intermediate model. The intermediate model is associated with the `ManyToManyField` using the
`through` argument to point to the model that will act as an intermediary.

```py
from django.db import models

class Person(models.Model):
	name = models.CharField(max_length=128)

	def __str__(self):
		return self.name

class Group(models.Model):
	name = models.CharField(max_length=128)
	members = models.ManyToManyField(Person, through='Membership')

	def __str__(self):
		return self.name

class Membership(models.Model):
	person = models.ForeignKey(Person, on_delete=models.CASCADE)
	group =  models.ForeignKey(Group, on_delete=models.CASCADE)
	date_joined = models.DateField()
	invite_reason = models.CharField(max_length=64)
```

Have to explicitly specify foreign keys to the models that are involved.

Restrictions:

* Must contain _only_ one foreign key to the source model, or specify the foreign keys Django should use for the
relationship using `ManyToManyField.through_fields`. 

Create many-to-many relationships by creating instances of the intermediate model.

```
>>> ringo = Person.objects.create(name="Ringo Starr")
>>> paul = Person.objects.create(name="Paul McCartney")
>>> beatles = Group.objects.create(name="The Beatles")
>>> m1 = Membership(person=ringo, group=beatles, date_joined=date(1962, 8, 16), invite_reason="Needed a new drummer.")
>>> m1.save()
>>> beatles.members.al()
<QuerySet [<Person: Ringo Starr>]>
>>> ringo.group_set.all()
<QuerySet [<Group: The Beatles>]>
>>>m2 = Membership.objects.create(person=paul, group=beatles, date_joined=date(1960, 8, 1,), invite_reason="Wanted to form a band.")
>>> beatles.members.all()
<QuerySet [<Person: Ringo Starr>, <Person: Paul McCartney>]>
```


You can also use `add()`, `create()`, or `set()` to create relationships, as long as you specify `through_defaults` for any required fields:

```
>>> beatles.members.add(john, through_defaults={'date_joined': date(1960, 8, 1)})
>>> beatles.members.create(name="George Harrison", through_defaults={'date_joined': date(1960, 8, 1)})
>>> beatles.members.set([john, paul, ringo, george], through_defaults={'date_joined': date(1960, 8, 1)})
```

The clear() method can be used to remove all many-to-many relationships for an instance:

```
>>> # Beatles have broken up
>>> beatles.members.clear()
>>> # Note that this deletes the intermediate model instances
>>> Membership.objects.all()
<QuerySet []>
```


Once you have established the many-to-many relationships, you can issue queries. Just as with normal many-to-many relationships, you can query using the attributes of the many-to-many-related model:

```
# Find all the groups with a member whose name starts with 'Paul'
>>> Group.objects.filter(members__name__startswith='Paul')
<QuerySet [<Group: The Beatles>]>
```

As you are using an intermediate model, you can also query on its attributes:

```
# Find all the members of the Beatles that joined after 1 Jan 1961
>>> Person.objects.filter(
...     group__name='The Beatles',
...     membership__date_joined__gt=date(1961,1,1))
<QuerySet [<Person: Ringo Starr]>
```

If you need to access a membership’s information you may do so by directly querying the Membership model:

```
>>> ringos_membership = Membership.objects.get(group=beatles, person=ringo)
>>> ringos_membership.date_joined
datetime.date(1962, 8, 16)
>>> ringos_membership.invite_reason
'Needed a new drummer.'
```

Another way to access the same information is by querying the many-to-many reverse relationship from a Person object:

```
>>> ringos_membership = ringo.membership_set.get(group=beatles)
>>> ringos_membership.date_joined
datetime.date(1962, 8, 16)
>>> ringos_membership.invite_reason
'Needed a new drummer.'
```


### One-to-one relationships

Most useful on the primary key of an object when that object "extends" another object in some way.

Example, a database of "places", build standard stuff such as address, phone number, etc. Then to build a 
database of restaurants on top of the places, instead of repeating yourself and replicating those fields in the
`Restaurant` model, you could make `Restaurant` have a `OneToOneField` to `Place` (you'd typically use `inheritance`
for this case since restaurant "is a" place though.)

> see [one to one](one_to_one.md)

`OneToOneField` also accepts an optional `parent_link` argument.


## Models across files

Can relate a model to one from another app. Just import the related model.

```py
from django.db import models
from geography.models import ZipCode

class Restaurant(models.Model):
	# ...
	zip_code = models.ForeignKey(ZipCode, on_delete=models.SET_NULL, blank=True, null=True)
```


## Meta options

Use an inner `class Meta`:

```py
from django.db import models

class Ox(models.Model):
	horn_length = models.IntegerField()

	class Meta:
		ordering = ["horn_length"]
		verbose_name_plural = "oxen"
```

Anything that isn't a field.

Check the [model option reference](option_reference.md)


## Model attributes

**`objects`**  
The most important attribute of a model is the `Manager`.

## Model methods

Define custom methods on a model to add custom "row-level" functionality to objects.

This keeps business logic in one place - the model.

```py
from django.db import models

class Person(models.Model):
	first_name = models.CharField(max_length=50)
	last_name = models.CharField(max_length=50)
	birth_date = models.DateField()

	def baby_boomer_status(self):
		"Returns the person's baby-boomer status."
		import datetime
		if self.birth_date < datetime.date(1945, 8, 1):
			return "Pre-boomer"
		elif self.birth_date < datetime.date(1965, 1, 1):
			return "Baby boomer"
		else:
			return "Post-boomer"

	@property
	def full_name(self):
		"Returns the person's full name."
		return '%s %s' (self.first_name, self.last_name)
```

* **`__str__()`**  
	Returns the string representation of an object.

* **`get_absolute_url()`**  
	Tells Django how to calculate the URL for an object.
	any object that has a URL that uniquely identifies it should define this method.

### Overriding predefined model methods

A classic use-case for overriding the built-in methods is if you want something to happen whenever you save an object. For example (see save() for documentation of the parameters it accepts):

```py
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def save(self, *args, **kwargs):
        do_something()
        super().save(*args, **kwargs)  # Call the "real" save() method.
        do_something_else()
```

You can also prevent saving:


```py
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def save(self, *args, **kwargs):
        if self.name == "Yoko Ono's blog":
            return # Yoko shall never have her own blog!
        else:
            super().save(*args, **kwargs)  # Call the "real" save() method.
```

It’s important to remember to call the superclass method – that’s that super().save(*args, **kwargs) business – to ensure that the object still gets saved into the database. If you forget to call the superclass method, the default behavior won’t happen and the database won’t get touched.

It’s also important that you pass through the arguments that can be passed to the model method – that’s what the *args, **kwargs bit does. Django will, from time to time, extend the capabilities of built-in model methods, adding new arguments. If you use *args, **kwargs in your method definitions, you are guaranteed that your code will automatically support those arguments when they are added.


## Model inheritance

Three styles of inheritance:

1. Just use the parent class to hold information that you don't want to have to type out for each child model.
This class isn't used in isolation so [Abstract base classes]() are what you need.

2. Subclassing an existing model and want each model to have its own database table, [Multi-table inheritance]()

3. If you only want to modify hte Python-level behavior without changing the models fields in any way use [Proxy models]()

### Abstract base classes

Put some common information into a number of other models. Write the base class and put `abstract=True` in the 
`Meta` class. The model won't be used to create any database table. When it's used as a base class for other models
its fields will be added to those of the child clas.

```py
from django.db import models

class CommonInfo(models.Model):
	name = models.CharField(max_length=100)
	age = models.PositiveIntegerField()

	class Meta:
		abstract = True

class Student(CommonInfo):
	home_group = models.CharField(max_length=5)
```

Fields inherited from abstract base classes can be overridden with another field or value, or be removed with `None`.

Provides a way to factor out common information at the Python level, while still only creating one database table
per child model at the database level.


#### Meta inheritance

Django makes any `Meta` inner class you declared in the base class available as an attribute.

```py
from django.db import models

class CommonInfo(models.Model):
	# ...
	class Meta:
		abstract = True
		ordering = ['name']

class Student(CommonInfo):
	# ...
	class Meta(CommonInfo.Meta):
		db_table = 'student_info'
```


### Multi-table inheritance

Each model in the hierarchy is a model all by itself. Has its own database table and can be queried and created
individually. The inheritance relationship introduces links between the child model and each of its parents via
an automatic OneToOneField.

```py
from django.db import models

class Place(models.Model):
	name = models.CharField(max_length=50)
	address = models.CharField(max_length=80)

class Restaurant(Place):
	serves_hot_dogs = models.BooleanField(default=False)
	serves_pizza = models.BooleanField(default=False)
```

All of the fields of `Place` will also be available in `Restaurant`, although the data will reside in a different
database table:

```
>>> Place.objects.filter(name="Bob's Cafe")
>>> Restaurant.objects.filter(name="Bob's cafe")
```

If you have a `Place` that's also a `Restaurant`, you can get from the `Place` object to the `Restaurant` object by
using the lowercase version of the model name:

```
>>> p = Place.objects.get(id=12)
# If p is a Restaurant object, this will give the child class:
>>> p.restaurant
<Restaurant: ...> 
```

#### Meta and multi-table inheritance

It doesn't make sense for a child class to inherit from its parent's `Meta` class.

A child model does not have access to its parent's `Meta` class.

#### Inheritance and reverse relations

It's possible to move from the parent down to the child. However this uses up the name that is the default
`related_name` value for `ForeignKey` and `ManyToManyField` relations.

If you are putting those types of relations on a subclass of the parent model, specify the `related_name` attribute
on each field.

### Proxy Models