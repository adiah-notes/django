Field lookups¶

Field lookups are how you specify the meat of an SQL WHERE clause. They’re specified as keyword arguments to the QuerySet methods filter(), exclude() and get().

For an introduction, see models and database queries documentation.

Django’s built-in lookups are listed below. It is also possible to write custom lookups for model fields.

As a convenience when no lookup type is provided (like in Entry.objects.get(id=14)) the lookup type is assumed to be exact.
exact¶

Exact match. If the value provided for comparison is None, it will be interpreted as an SQL NULL (see isnull for more details).

Examples:

Entry.objects.get(id__exact=14)
Entry.objects.get(id__exact=None)

SQL equivalents:

SELECT ... WHERE id = 14;
SELECT ... WHERE id IS NULL;

MySQL comparisons

In MySQL, a database table’s “collation” setting determines whether exact comparisons are case-sensitive. This is a database setting, not a Django setting. It’s possible to configure your MySQL tables to use case-sensitive comparisons, but some trade-offs are involved. For more information about this, see the collation section in the databases documentation.
iexact¶

Case-insensitive exact match. If the value provided for comparison is None, it will be interpreted as an SQL NULL (see isnull for more details).

Example:

Blog.objects.get(name__iexact='beatles blog')
Blog.objects.get(name__iexact=None)

SQL equivalents:

SELECT ... WHERE name ILIKE 'beatles blog';
SELECT ... WHERE name IS NULL;

Note the first query will match 'Beatles Blog', 'beatles blog', 'BeAtLes BLoG', etc.

SQLite users

When using the SQLite backend and non-ASCII strings, bear in mind the database note about string comparisons. SQLite does not do case-insensitive matching for non-ASCII strings.
contains¶

Case-sensitive containment test.

Example:

Entry.objects.get(headline__contains='Lennon')

SQL equivalent:

SELECT ... WHERE headline LIKE '%Lennon%';

Note this will match the headline 'Lennon honored today' but not 'lennon honored today'.

SQLite users

SQLite doesn’t support case-sensitive LIKE statements; contains acts like icontains for SQLite. See the database note for more information.
icontains¶

Case-insensitive containment test.

Example:

Entry.objects.get(headline__icontains='Lennon')

SQL equivalent:

SELECT ... WHERE headline ILIKE '%Lennon%';

SQLite users

When using the SQLite backend and non-ASCII strings, bear in mind the database note about string comparisons.
in¶

In a given iterable; often a list, tuple, or queryset. It’s not a common use case, but strings (being iterables) are accepted.

Examples:

Entry.objects.filter(id__in=[1, 3, 4])
Entry.objects.filter(headline__in='abc')

SQL equivalents:

SELECT ... WHERE id IN (1, 3, 4);
SELECT ... WHERE headline IN ('a', 'b', 'c');

You can also use a queryset to dynamically evaluate the list of values instead of providing a list of literal values:

inner_qs = Blog.objects.filter(name__contains='Cheddar')
entries = Entry.objects.filter(blog__in=inner_qs)

This queryset will be evaluated as subselect statement:

SELECT ... WHERE blog.id IN (SELECT id FROM ... WHERE NAME LIKE '%Cheddar%')

If you pass in a QuerySet resulting from values() or values_list() as the value to an __in lookup, you need to ensure you are only extracting one field in the result. For example, this will work (filtering on the blog names):

inner_qs = Blog.objects.filter(name__contains='Ch').values('name')
entries = Entry.objects.filter(blog__name__in=inner_qs)

This example will raise an exception, since the inner query is trying to extract two field values, where only one is expected:

# Bad code! Will raise a TypeError.
inner_qs = Blog.objects.filter(name__contains='Ch').values('name', 'id')
entries = Entry.objects.filter(blog__name__in=inner_qs)

Performance considerations

Be cautious about using nested queries and understand your database server’s performance characteristics (if in doubt, benchmark!). Some database backends, most notably MySQL, don’t optimize nested queries very well. It is more efficient, in those cases, to extract a list of values and then pass that into the second query. That is, execute two queries instead of one:

values = Blog.objects.filter(
        name__contains='Cheddar').values_list('pk', flat=True)
entries = Entry.objects.filter(blog__in=list(values))

Note the list() call around the Blog QuerySet to force execution of the first query. Without it, a nested query would be executed, because QuerySets are lazy.
gt¶

Greater than.

Example:

Entry.objects.filter(id__gt=4)

SQL equivalent:

SELECT ... WHERE id > 4;

gte¶

Greater than or equal to.
lt¶

Less than.
lte¶

Less than or equal to.
startswith¶

Case-sensitive starts-with.

Example:

Entry.objects.filter(headline__startswith='Lennon')

SQL equivalent:

SELECT ... WHERE headline LIKE 'Lennon%';

SQLite doesn’t support case-sensitive LIKE statements; startswith acts like istartswith for SQLite.
istartswith¶

Case-insensitive starts-with.

Example:

Entry.objects.filter(headline__istartswith='Lennon')

SQL equivalent:

SELECT ... WHERE headline ILIKE 'Lennon%';

SQLite users

When using the SQLite backend and non-ASCII strings, bear in mind the database note about string comparisons.
endswith¶

Case-sensitive ends-with.

Example:

Entry.objects.filter(headline__endswith='Lennon')

SQL equivalent:

SELECT ... WHERE headline LIKE '%Lennon';

SQLite users

SQLite doesn’t support case-sensitive LIKE statements; endswith acts like iendswith for SQLite. Refer to the database note documentation for more.
iendswith¶

Case-insensitive ends-with.

Example:

Entry.objects.filter(headline__iendswith='Lennon')

SQL equivalent:

SELECT ... WHERE headline ILIKE '%Lennon'

SQLite users

When using the SQLite backend and non-ASCII strings, bear in mind the database note about string comparisons.
range¶

Range test (inclusive).

Example:

import datetime
start_date = datetime.date(2005, 1, 1)
end_date = datetime.date(2005, 3, 31)
Entry.objects.filter(pub_date__range=(start_date, end_date))

SQL equivalent:

SELECT ... WHERE pub_date BETWEEN '2005-01-01' and '2005-03-31';

You can use range anywhere you can use BETWEEN in SQL — for dates, numbers and even characters.

Warning

Filtering a DateTimeField with dates won’t include items on the last day, because the bounds are interpreted as “0am on the given date”. If pub_date was a DateTimeField, the above expression would be turned into this SQL:

SELECT ... WHERE pub_date BETWEEN '2005-01-01 00:00:00' and '2005-03-31 00:00:00';

Generally speaking, you can’t mix dates and datetimes.
date¶

For datetime fields, casts the value as date. Allows chaining additional field lookups. Takes a date value.

Example:

Entry.objects.filter(pub_date__date=datetime.date(2005, 1, 1))
Entry.objects.filter(pub_date__date__gt=datetime.date(2005, 1, 1))

(No equivalent SQL code fragment is included for this lookup because implementation of the relevant query varies among different database engines.)

When USE_TZ is True, fields are converted to the current time zone before filtering. This requires time zone definitions in the database.
year¶

For date and datetime fields, an exact year match. Allows chaining additional field lookups. Takes an integer year.

Example:

Entry.objects.filter(pub_date__year=2005)
Entry.objects.filter(pub_date__year__gte=2005)

SQL equivalent:

SELECT ... WHERE pub_date BETWEEN '2005-01-01' AND '2005-12-31';
SELECT ... WHERE pub_date >= '2005-01-01';

(The exact SQL syntax varies for each database engine.)

When USE_TZ is True, datetime fields are converted to the current time zone before filtering. This requires time zone definitions in the database.
iso_year¶

For date and datetime fields, an exact ISO 8601 week-numbering year match. Allows chaining additional field lookups. Takes an integer year.

Example:

Entry.objects.filter(pub_date__iso_year=2005)
Entry.objects.filter(pub_date__iso_year__gte=2005)

(The exact SQL syntax varies for each database engine.)

When USE_TZ is True, datetime fields are converted to the current time zone before filtering. This requires time zone definitions in the database.
month¶

For date and datetime fields, an exact month match. Allows chaining additional field lookups. Takes an integer 1 (January) through 12 (December).

Example:

Entry.objects.filter(pub_date__month=12)
Entry.objects.filter(pub_date__month__gte=6)

SQL equivalent:

SELECT ... WHERE EXTRACT('month' FROM pub_date) = '12';
SELECT ... WHERE EXTRACT('month' FROM pub_date) >= '6';

(The exact SQL syntax varies for each database engine.)

When USE_TZ is True, datetime fields are converted to the current time zone before filtering. This requires time zone definitions in the database.
day¶

For date and datetime fields, an exact day match. Allows chaining additional field lookups. Takes an integer day.

Example:

Entry.objects.filter(pub_date__day=3)
Entry.objects.filter(pub_date__day__gte=3)

SQL equivalent:

SELECT ... WHERE EXTRACT('day' FROM pub_date) = '3';
SELECT ... WHERE EXTRACT('day' FROM pub_date) >= '3';

(The exact SQL syntax varies for each database engine.)

Note this will match any record with a pub_date on the third day of the month, such as January 3, July 3, etc.

When USE_TZ is True, datetime fields are converted to the current time zone before filtering. This requires time zone definitions in the database.
week¶

For date and datetime fields, return the week number (1-52 or 53) according to ISO-8601, i.e., weeks start on a Monday and the first week contains the year’s first Thursday.

Example:

Entry.objects.filter(pub_date__week=52)
Entry.objects.filter(pub_date__week__gte=32, pub_date__week__lte=38)

(No equivalent SQL code fragment is included for this lookup because implementation of the relevant query varies among different database engines.)

When USE_TZ is True, datetime fields are converted to the current time zone before filtering. This requires time zone definitions in the database.
week_day¶

For date and datetime fields, a ‘day of the week’ match. Allows chaining additional field lookups.

Takes an integer value representing the day of week from 1 (Sunday) to 7 (Saturday).

Example:

Entry.objects.filter(pub_date__week_day=2)
Entry.objects.filter(pub_date__week_day__gte=2)

(No equivalent SQL code fragment is included for this lookup because implementation of the relevant query varies among different database engines.)

Note this will match any record with a pub_date that falls on a Monday (day 2 of the week), regardless of the month or year in which it occurs. Week days are indexed with day 1 being Sunday and day 7 being Saturday.

When USE_TZ is True, datetime fields are converted to the current time zone before filtering. This requires time zone definitions in the database.
iso_week_day¶

For date and datetime fields, an exact ISO 8601 day of the week match. Allows chaining additional field lookups.

Takes an integer value representing the day of the week from 1 (Monday) to 7 (Sunday).

Example:

Entry.objects.filter(pub_date__iso_week_day=1)
Entry.objects.filter(pub_date__iso_week_day__gte=1)

(No equivalent SQL code fragment is included for this lookup because implementation of the relevant query varies among different database engines.)

Note this will match any record with a pub_date that falls on a Monday (day 1 of the week), regardless of the month or year in which it occurs. Week days are indexed with day 1 being Monday and day 7 being Sunday.

When USE_TZ is True, datetime fields are converted to the current time zone before filtering. This requires time zone definitions in the database.
quarter¶

For date and datetime fields, a ‘quarter of the year’ match. Allows chaining additional field lookups. Takes an integer value between 1 and 4 representing the quarter of the year.

Example to retrieve entries in the second quarter (April 1 to June 30):

Entry.objects.filter(pub_date__quarter=2)

(No equivalent SQL code fragment is included for this lookup because implementation of the relevant query varies among different database engines.)

When USE_TZ is True, datetime fields are converted to the current time zone before filtering. This requires time zone definitions in the database.
time¶

For datetime fields, casts the value as time. Allows chaining additional field lookups. Takes a datetime.time value.

Example:

Entry.objects.filter(pub_date__time=datetime.time(14, 30))
Entry.objects.filter(pub_date__time__range=(datetime.time(8), datetime.time(17)))

(No equivalent SQL code fragment is included for this lookup because implementation of the relevant query varies among different database engines.)

When USE_TZ is True, fields are converted to the current time zone before filtering. This requires time zone definitions in the database.
hour¶

For datetime and time fields, an exact hour match. Allows chaining additional field lookups. Takes an integer between 0 and 23.

Example:

Event.objects.filter(timestamp__hour=23)
Event.objects.filter(time__hour=5)
Event.objects.filter(timestamp__hour__gte=12)

SQL equivalent:

SELECT ... WHERE EXTRACT('hour' FROM timestamp) = '23';
SELECT ... WHERE EXTRACT('hour' FROM time) = '5';
SELECT ... WHERE EXTRACT('hour' FROM timestamp) >= '12';

(The exact SQL syntax varies for each database engine.)

When USE_TZ is True, datetime fields are converted to the current time zone before filtering. This requires time zone definitions in the database.
minute¶

For datetime and time fields, an exact minute match. Allows chaining additional field lookups. Takes an integer between 0 and 59.

Example:

Event.objects.filter(timestamp__minute=29)
Event.objects.filter(time__minute=46)
Event.objects.filter(timestamp__minute__gte=29)

SQL equivalent:

SELECT ... WHERE EXTRACT('minute' FROM timestamp) = '29';
SELECT ... WHERE EXTRACT('minute' FROM time) = '46';
SELECT ... WHERE EXTRACT('minute' FROM timestamp) >= '29';

(The exact SQL syntax varies for each database engine.)

When USE_TZ is True, datetime fields are converted to the current time zone before filtering. This requires time zone definitions in the database.
second¶

For datetime and time fields, an exact second match. Allows chaining additional field lookups. Takes an integer between 0 and 59.

Example:

Event.objects.filter(timestamp__second=31)
Event.objects.filter(time__second=2)
Event.objects.filter(timestamp__second__gte=31)

SQL equivalent:

SELECT ... WHERE EXTRACT('second' FROM timestamp) = '31';
SELECT ... WHERE EXTRACT('second' FROM time) = '2';
SELECT ... WHERE EXTRACT('second' FROM timestamp) >= '31';

(The exact SQL syntax varies for each database engine.)

When USE_TZ is True, datetime fields are converted to the current time zone before filtering. This requires time zone definitions in the database.
isnull¶

Takes either True or False, which correspond to SQL queries of IS NULL and IS NOT NULL, respectively.

Example:

Entry.objects.filter(pub_date__isnull=True)

SQL equivalent:

SELECT ... WHERE pub_date IS NULL;

regex¶

Case-sensitive regular expression match.

The regular expression syntax is that of the database backend in use. In the case of SQLite, which has no built in regular expression support, this feature is provided by a (Python) user-defined REGEXP function, and the regular expression syntax is therefore that of Python’s re module.

Example:

Entry.objects.get(title__regex=r'^(An?|The) +')

SQL equivalents:

SELECT ... WHERE title REGEXP BINARY '^(An?|The) +'; -- MySQL

SELECT ... WHERE REGEXP_LIKE(title, '^(An?|The) +', 'c'); -- Oracle

SELECT ... WHERE title ~ '^(An?|The) +'; -- PostgreSQL

SELECT ... WHERE title REGEXP '^(An?|The) +'; -- SQLite

Using raw strings (e.g., r'foo' instead of 'foo') for passing in the regular expression syntax is recommended.
iregex¶

Case-insensitive regular expression match.

Example:

Entry.objects.get(title__iregex=r'^(an?|the) +')

SQL equivalents:

SELECT ... WHERE title REGEXP '^(an?|the) +'; -- MySQL

SELECT ... WHERE REGEXP_LIKE(title, '^(an?|the) +', 'i'); -- Oracle

SELECT ... WHERE title ~* '^(an?|the) +'; -- PostgreSQL

SELECT ... WHERE title REGEXP '(?i)^(an?|the) +'; -- SQLite

