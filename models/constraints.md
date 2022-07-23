# Constraints reference

They are added in the model `Meta.constraints` option.

> Referencing built-in constraints
>
> Constraints are defined in `django.db.models.constraints` but for convenience they're imported into 
> `django.db.models`. The standard convention is to use `from django.db import models` and refer to the constraints
> as `models.<Foo>Constraint`.


> Constraints in abstract base classes
>
> Always specify a unique name for the constraint.


# CheckConstraint

`class CheckConstraint(*, check, name)`

Creates a check constraint in the database.

## check

`CheckConstraint.check`

A `Q` object or boolean [Expression]() that specifies the check you want the constraint to enforce.

E.g. `CheckConstraint(check=Q(age__gte=18), name='age_gte_18')` ensures the age field is never less than 18.

---

## name

`CheckConstraint.name`

The name of the constraint. Always specify a unique name for the constraint.

---

# UniqueConstraint

`class UniqueConstraint(*expressions, fields=(), name=None, condition=None, deferrable=None, include=None, opclasses=())`

Creates a unique constraint in the database.


## expressions

`UniqueConstraint.expressions`

Positional argument `*expressions` allows creating functional unique constraints on expressions and database
functions.

Eg.

```py
UniqueConstraint(Lower('name').desc(), 'category', name='unique_lower_name_category')
```

creates a unique constraint on the lowercased value of the `name` field in descending order and the category 
field in the default ascending order.

---

## fields

`UniqueConstraint.fields`

A list of field names that specifies the unique set of columns you want the constraint to enforce.

`UniqueConstraint(fields=['room', 'date'], name='unique_booking')` ensures each room can only be booked once
for each date.

---

## name

`UniqueConstraint.name`

The name of the constraint

---

## condition

`UniqueConstraint.condition`

A `Q` object that specifies the condition you want the constraint to enforce.

```py
UniqueConstraint(fields=['user'], condition=Q(status='DRAFT'), name='unique_draft_user')
```
ensures that each user only has one draft.

---

## deferrable

`UniqueConstraint.deferrable`

---

## include

`UniqueConstraint.include`

A list or tuple of the names of the fields to be included in the covering unique index as non-key columns.
This allows index-only scans to be used for queries that select only included fields (`include`) and filter
only by unique fields (`fields`).

```py
UniqueConstraint(name='unique_booking', fields=['room', 'date'], include=['full_name'])
```
will allow filtering on `room` and `date`, also selecting `full_name`, while fetching data only from the index.

---

