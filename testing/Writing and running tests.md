# Writing Tests

Django's unit tests use the `unittest` module, which defines tests using a class-based approach.

Example which subclasses from `django.test.TestCaee`, which is a subclass of `unittest.TestCase`:

```py
from django.test import TestCase
from myapp.models import Animal

class AnimalTestCase(TestCase):
    def setUp(self):
        Animal.objects.create(name="lion", sound="roar")
        Animal.objects.create(name="cat", sound="meow")

    def test_animals_can_speak(self):
        """Animals that can speak are correctly identified"""
        lion = Animal.objects.get(name="lion")
        cat = Animal.objects.get(name="cat")
        self.assertEqual(lion.speak(), 'The lion says "roar"')
        self.assertEqual(cat.speak(), 'The cat says "meow"')
```

When running tests, the default behavior is to find all the
test cases (subclasses of `unittest.TestCase`) in any file 
whose name begins with `test`.

The default `startapp` template creates a `test.py` file. This is fine for a few tests, but when it grows, restructure it
into a tests package.

If tests rely on database access such as querying models, be sure to create classes as subclasses of 
`django.test.TestCase` rather than `unittest.TestCase`.

Using `unittest.TestCase` avoids the cost of running each test in a transaction and flushing the database.

---

# Running Tests

Run tests using the `test` command of the `manage.py` utility:

    $ ./manage.py test

Can specify particular tests to run by supplying any number of "test labels" to `./manage.py test`.

```
# Run all the tests in the animals.tests module
$ ./manage.py test animals.tests

# Run all the tests found within the 'animals' package
$ ./manage.py test animals

# Run just one test case
$ ./manage.py test animals.tests.AnimalTestCase

# Run just one test method
$ ./manage.py test animals.tests.AnimalTestCase.test_animals_can_speak
```

Can also provide a path to a directory to discover tests below that directory:

    $ ./manage.py test animals/

Can specify a custom filename pattern match using the `-p` (or `--pattern`) option, if your test files are named 
differently from the `test*.py` pattern:

    $ ./manage.py test --pattern="tests_*.py"


It's a good idea to run your tests with Python warnings enabled `python -Wa manage.py test`. Displays deprecation warnings.

## The test database

Tests that require a database (model tests) will not use the real (production) database. Separate, blank databaes are
created for the tests.

The databasses are destroyed when all tests have been executed.

Can prevent the databases from being destroyed by using the `test --keepdb` option.

If a test run is forcefull interrupted, the test database may not be destroyed. On the next run, you'll be asked whether 
you want to reuse or destroy the database. Use the `test --noinput` option to suppress that prompt and destroy.

---

## Order in which tests are executed

In order to guarantee that all `TesCase` code starts with a
clean database, the Django test runner reorders tests in the
following way:

* All `TestCase` subclasses are run first.
* Then, all other Django-based tests (test cases based on `SimpleTestCase`, including `TransactionTestCase`)
* Then any other `unittest.TestCase` tests (including doctests) that may alter the database without restoring it to its
original state are run.

---

## Rollback emulation

## Understanding the test output

Control the level of detail with the `verbosity` option 

    Creating test database...
    Creating table myapp_animal
    Creating table mypp_mineral

This tells you that the test runner is creating a test database.

Once the test database has been created, Django wil run the tests.
