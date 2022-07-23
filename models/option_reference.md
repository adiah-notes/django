# Model `Meta` options

## abstract

`Options.abstract`. If `abstract = True`, this model will be an abstract base class.

---

## app_label

`Options.app_label`

If a model is defined outside of an application in `INSTALLED_APPS`, it must declare which app it belongs to:

```py
app_label = 'myapp'
```

---

## base_manager_name

`Options.base_manager_name`

The attribute name of the manager, for example, `'objects'`, to use for the model's `_base_manager`.

---

## db_table

`Options.db_table`

The name of the database table to use for the model:

```py
db_table = 'music_album'
```

---

## db_tablespace

`Options.db_tablespace`

---

## default_manager_name

`Options.default_manager_name`

The name of the manager to use for the model's `_default_manager`.

---

## default_related_name

`Options.default_related_name`

The name used by default for the relation from a related object back to this one. The default is `<model_name>_set`.

Also sets `related_query_name`.

---

## get_latest_by

`Options.get_latest_by`

The name of a field or a list of field names in the model, typically `DateField`, `DateTimeField`, or `IntegerField`.
Specifies the default field(s) to use in model `Manager`'s `latest()` and `earliest()` methods.

```py
# Lates by ascending order_date.
get_latest_by = "order_date"

# Latest by priority descending, order_date ascending.
get_latest_by = ['-priority', 'order_date']
```

---

## order_with_respect_to

`Options.order_with_respect_to`

Makes this object orderable with respect to the given field, usually a `ForeignKey`.
Used to make related objects orderable with respect to a parent object. For example, if an `Answer` relates to a 
`Question` object, and a question has more than one answer, and the order of answersmatters:

```py
from django.db import models

class Question(models.Model):
	text = models.TextField()
	# ...

class Answer(models.Model):
	question = models.ForeignKey(Question, on_delete=models.CASCADE)
	# ...

	class Meta:
		order_with_respect_to = 'question'
```

Two additional methods are provided to retrieve and to set the order of the related objects: `get_RELATED_order()` 
and `set_RELATED_order()`, where `RELATED` is the lowercased model name.

```
>>> question = Question.objects.get(id=1)
>>> question.get_answer_order()
[1, 2, 3]
```

The order of a `Question` object's related `Answer` objects can be set by passing in a list of `Answer` primary keys:

```
>>> question.set_answer_order([3, 1, 2])
```

The related objects also get two methods `get_next_in_order()` and `get_previous_in_order()`.

```
>>> answer = Answer.objects.get(id=2)
>>> answer.get_next_in_order()
<Answer: 3>
>>> answer.get_previous_in_order()
<Answer: 1>
```

---

## ordering

`Options.ordering`

The default ordering for the object, for use when obtaining lists of objects:

```py
ordering = ['-order_date']
```

A tuple or list of strings and/or query expressions.
Use "-" to order descending, and string "?" to order randomly


To order by `author` ascending and make null values sort last:

```py
from django.db.models import F

ordering = [F('author').asc(nulls_last=True)]
```

---

## permissions

`Options.permissions`

Extra permissions to enter into the permissions table when creating this object.
Add, change, delete, and view permissions are automatically created for each model.

```py
permissions = [('can_deliver_pizzas', 'Can deliver pizzas')]
```

A list of tuple of 2-tuples in the format `(permission_code, human_readable_permission_name)`.

---

## default_permissions


---

## proxy

`Options.proxy`

If `proxy = True`, a model which subclasses another model will be treated as a proxy model.

---

## indexes

`Options.indexes`

A list of indexes to define on the model.

```py
from django.db import models

class Customer(models.Model):
	first_name = models.CharField(max_length=100)
	last_name = models.CharField(max_length=100)

	class Meta:
		indexes = [
			models.Index(fields=['last_name', 'first_name']),
			models.Index(fields=['first_name'], name='first_name_idx'),
		]
```

---

## unique_together

`Options.unique_together`

Use `UniqueConstraint` with the `constraints` option instead.

---

## constraints

`Options.contraints`

A list of [constraints](constraints.md) that you want to define on the model.

```py
from django.db import models

class Customer(models.Model):
	age = models.IntegerField()

	class Meta:
		constraints = [
			models.CheckConstraint(check=models.Q(age__gte=18), name='age_gte_18'),
		]
```

## verbose_name

`Options.verbose_name`

A human readable name for the object, singular

```py
verbose_name = "pizza"
```

---

## verbose_name_plural

`Options.verbose_name_plural`

The plural name for the object

```py
verbose_name_plural = "stories"
```

---

# Read-only `Meta` attributes

## label

`Options.label`

Representation of the object, returns `app_label.object_name`, eg `'polls.Question'`.

---

## label_lower

`Options.label_lower`

Representation of the model, returns `app_label.model_name` eg `'polls.question'`.