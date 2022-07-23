# Note to self: 

**Don't extend abstract user if you want to change how django does the auth, even if it's just changing to email instead of username**

It affects the admin create user page, and username still has to be required for the django auths.

and use the one to one when you're not changing the default User either, else might as well just update the AbstractUser


# Proxy models

You can create, delete and update instances of the proxy model and all the data will be saved as if you were using the
original model. You can change the default model ordering or the default manager in the proxy, without having to alter
the original.

Declared like normal models, set the `proxy` attribute of the `Meta` class to `True`.

Example adding a method to the `Person` model:

```py
from django.db import models

class Person(models.Model):
	first_name = models.CharField(max_length=30)
	last_name = models.CharField(max_length=30)

class MyPerson(Person):
	class Meta:
		proxy = True
	
	def do_something(self):
		# ...
		pass
```

The MyPerson class operates on the same database table as its parent `Person` class. Any new instances will be
accessible through the other.

Use the proxy model to define a different default ordering on a model. Example if you regularly order by the
`last_name` attribute, but don't always want to order the `Person` model.

```py
class OrderedPerson(Person):
	class Meta:
		ordering = ["last_name"]
		proxy = True
```

So, normal `Person` queries will be unordered and `OrderedPerson` queries will be ordered by `last_name`.


If you don't specify any model managers on a proxy model, it inherits the managers from its model  parents.
If you define a manager on the proxy model, it will become the default, although any managers defined on the
parent classes will still be available.


```py
from django.db import models

class NewManager(models.Manager):
	# ...
	pass

class MyPerson(Person):
	objects = NewManager()

	class Meta:
		proxy = True
```


* If you are mirroring an existing model or database table and don't want all the original database table columns,
use `Meta.managed=False`. That option is normally useful for modelling database views and tables not under the
control of Django.

* If you are wanting to change the Python-only behavior of a model, but keep all the same fields as in the original,
use `Meta.proxy=True`. This sets things up so that the proxy model is an exact copy of the storage structure
of the original model when data is saved.

<br>

# Using a `OneToOneField` to extend the user


To store information related to `User`, use a `OneToOneField` to a model containing the fields for additional 
information. This one-to-one model is often called a profile model, as it might store non-auth related information
about a site user.

```py
from django.contrib.auth.models import User

class Employee(models.Model):
	user = models.OneToOneField(User, on_delete=models.CASCADE)
	department = models.CharField(max_length=100)
```

Assuming an existing `Employee` Fred Smith has both a `User` and `Employee` model, you can access the related
information using Django's standard related model conventions:

```
>>> u = User.objects.get(username='fsmith')
>>> freds_department = u.employee.department
```

To add a profile model's fields to the user page in the admin, define an `InlineModelAdmin` (in this example,
used a `StackedInline`) in the `admin.py` and add it to a `UserAdmin` class which is registered with the `User` class:

```py
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin
from django.contrib.auth.models import User

from my_user_profile_app.models import Employee

# Define an inline admin descriptor for Employee model
# which acts a bit like a singleton
class EmployeeInline(admin.StackedInline):
	model = Employee
	can_delete = False
	verbose_name_plural = 'employee'

# Define a new User admin
class UserAdmin(BaseUserAdmin):
	inlines = (EmployeeInline,)

# Re-register UserAdmin
admin.site.unregister(User)
admin.site.register(User, UserAdmin)

# or use a new model extending the AbstractUser
```

These models aren't created when a user is created, use a `django.db.models.signals.post_save` to create or update
related models as appropriate

Using related models results in additional queries or joins to retrieve the related data.


<br>

