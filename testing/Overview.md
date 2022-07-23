# Testing in Django

Use a collection tests - a **test suite** -to solve/avoid a number of problems:

* When writing new code, use tests to validate your code works as expected.
* When refactoring/modify old code, use tests to ensure changes haven't affected application's behaviour unexpectedly.

With Django's test-execution framework and assorted utilities, you can simulate requests, insert test data, inspect
output and generally verify code is doing what it should be.

The preferred way to write tests is using the `unittest` module built into the Python standard library.


