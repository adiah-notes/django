# Testing tools

## The test client

A Python class that acts as a dummy web browser, allowing you to test your views and interact with Django application
programmatically.

* Simulate GET and POST requests on a URL and observe the response.
* See the chain of redirects (if any) and check the URL and status at each step.
* Test that a given request is redered by a given Django template, with a template context that contains certain values.

It is not a replacement for `Selenium`

* Use Django's test client to establish that the correct template is being rendered and theat the template is passed the correct data.
* Use in-browser frameworks like Selenium to test _rendered_ HTML and the _behaviour_ of web pages, namely JavaScript functionality.

    Django also provides special support for thos frameworks


### Overview and a quick example

To use the test client, instantiate `django.test.Client` and retrieve web pages:


```py
>>> from django.test import Client
>>> c = Client()
>>> response = c.post('/login/', {'username': 'john', 'password': 'smith'})
>>> response.status_code
200
>>> response = c.get('/customer/details/')
>>> response.content
b'<!DOCTYPE html...'
```


You can instantiate `Client` from within a session of the Python interactive interpreter.

* The test client does _not_ require the web server to be running. It avoids the overhead of HTTP and deals directly with the Django framework.

* When retrieving pages, remember to specify the _path_ of the URL, not the whole domain

```py
>>> c.get('/login/')
```

**not**

```py
>>> c.get('https://www.example.com/login/')
```

The test client is not capable of retrieving web pages that are not powered by the Django project.

To retrieve other web pages, use a Python standard library module such as `urllib`.

* To resolve URLs, the test client uses whatever URLconf is pointed-to by the `ROOT_URLCONF` setting.

* Some functionality such as the template-related functionality is only available _while tests are running_.

* By default, the test client will disable any CSRF-checks performed by your site.

    Can force it to use CSRF checks:
    ```py
    >>> from django.test import Client
    >>> csrf_client = Client(enforce_csrf_checks=True)
    ```

---

### Making requests

use the `django.test.Client` class to make requests.

class `Client(enforce_csrf_checks=False, json_encoder=DjangoJSONEncoder, **defaults)

It requires no arguments at time of construction. However, you can use keyword arguments to specify some default headers.

This will send a `User-Agent` HTTP header in each request:

```py
>>> c = Client(HTTP_USER_AGENT='Mozilla/5.0')
```

The values from the `extra` keyword arguments passed to `get()`, `post()`, etc have precedence over the defaults passed
to the class constructor.

The `enforce_csrf_checks` argument can be used to test CSRF protection.

The `json_encoder` argument allows setting a custom JSON encoder for the JSON serialization that's descibed in `post()`.

The `raise_request_exception` argument allows controlling whether or not exceptions raised during the request should
also be raised in the test. Defaults to `True`.


Once you have a `Client` instance, you can call any of the following methods:

* `get(path, data=None, follow=False, secure=False, **extra)`

    Makes a GET request on the provided `path` and returns a `Response` object.

    The key-value pairs in the `data` dictionary are used to create a GET data payload. Example:

    ```py
    >>> c = Client()
    >>> c.get('/customers/details/', {'name': 'fred', 'age': 7})
    ```

    will result in the evaluation of a GET request equivalent to:

    ```py
    /customers/details/?name=fred&age=7
    ```

    The `extra` keyword arguments parameter can be used to specify headers to be sent in the request:

    ```py
    >>> c = Client()
    >>> c.get('/customers/details/', {'name': 'fred', 'age': 7}, HTTP_ACCEPT='application/json')
    ```

    -> will send the HTTP header `HTTP_ACCEPT` to the details view, which is a good way to test code paths
    that use the `django.http.HttpRequest.accepts()` method.

    If you already have the GET arguments in URL-encoded form, you can use that encoding instead of using the data argument:

    ```py
    >>> c = Client()
    >>> c.get('/customers/details/?name=fred&age=7')
    ```

    If you provide both, the data argument will take precedence.


    If you set `follow` to `True` the client will follow any redirects and a `redirect_chain` attribute will be set
    in the response object containing tuples of the intermediate urls and status codes.

    If you had a URL `/redirect_me/` that redirected to `/next/`, that redirected to `/final/`, you'd see:

    ```py
    >>> response = c.get('/redirect_me/', follow=True)
    >>> response.redirect_chain
    [('http://testserver/next', 302), ('http://testserver/final/', 302)]
    ```

    If you set `secure` to `True` the client will emulate an HTTPS request.

* `post(path, data=None, content_type=MULTIPART_CONTENT, follow=False, secure=False, **extra)`

    Makes a POST request on the provided `path` and returns a `Response` object.

    The key-value pairs in the `data` dictionary are used to submit POST data:

    ```py
    >>> c = Client()
    >>> c.post('/login/', {'name': 'fred', 'passwd': 'secret'})
    ```

    ...will result in the evaluation of a POST request to this URL:

    ```py
    /login/
    ```

    ...with this POST data:

    ```py
    name=fred&passwd=secret
    ```

    If you provide `content_type` as application/json, the `data` is serialized using `json.dumps()` if it'a dict,
    list, or tuple. 

    Serialization is performed with `DjangoJSONEncoder` by default, and can be overridden by providing a 
    `json_encoder` argument to `Client`.

    This serialization also happens for `put()`, `patch()`, and `delete()` requests.


    If you provide an other `content_type`, the contents of `data` are sent as-is in the POST request.

    If you don't provide a value for `content_type`, the values in `data` will be transmitted with a content
    type of `multipart/form-data`.


    To submit multiple values for a given key, provide the values as a list or tuple for the required key.

    E.g. this value of `data` would submit three selected values for the field named `choices`:

    ```py
    {'choices': ('a', 'b', 'd')}
    ```


    Submitting files is a special case. To POST a file, you need only provide the file fields name as a key, 
    and a file handle to the file you wish to upload as a value:

    ```py
    >>> c = Client()
    >>> with open('wishlist.doc', 'rb') as fp:
    ...     c.post('/customers/wishes/', {'name': 'fred', 'attachment': fp})
    ```


    (The name `attachment` here is not relevant; use whatever name your code expects)

    You may also provide any file-like object as a file handle. If you're uploading to an `ImageField`, the
    object needs a `name` attribute that passes the `validate_image_file_extension` validator:

    ```py
    >>> from io import BytesIO
    >>> img = BytesIO(b'mybinarydata')
    >>> img.name = 'myimage.jpg'
    ```

* `head(path, data=None, follow=False, secure=False, **extra)`

    Makes a HEAD request on the provided `path` and returns a `Response` object. Works just like 
    `Client.get()`.

* `options(path, data=", content_type='application/octet-stream', follow=False, secure=False, **extra)`

    Makes an OPTIONS request on the provided `path` and returns a `Response` object. Useful for testing 
    RESTful interfaces.


    When `data` is provided, it is used as the request body, and a `Content-Type` header is set to `content_type`.

    `follow`, `secure` and `extra` arguments act the same as for `Client.get()`.

* `put(path, data='', content_type='application/octet-stream', follow=False, secure=False, **extra)`

    Makes a PUT request on the provided `path` and returns a `Response` object. Useful for RESTful interfaces.

    similar to the `options` above.

* `patch(path, data='', content_type='application/octet-stream', follow=False, secure=False, **extra)`

    Makes a PATCH request on the provided `path` and returns a `Response` object. 

* `delete(path, data='', content_type='application/octet-stream', follow=False, secure=False, **extra)`

    Makes a DELETE request on the provided `path` and returns a `Response` object.

* `trace(path, follow=False, secure=False, **extra)`

    Makes a TRACE request on the provided `path` and returns a `Response` object.

    `data` is not provided as a keyword.

    `follow`, `secure`, and `extra` same as `Client.get()`

* `login(**credentials)`

    Use the test client's `login()` method to simulate the effect of a user logging into the site.

    The format of the `credentials` argument depends on which authentication backend. 

    The standard authentication backend `credentials` should be the user's username and password.

    ```py
    >>> c = Client()
    >>> c.login(username='fred', password='secret')
    # Now you can access a view that's only available to logged in users.
    ```

    `login()` returns `True` if the credentials were accepted and login was successful.

    Remember to create user for the test data.

* `force_login(user, backend=None)`

    Can use instead of `login()` when a test requires a user be logged in and the details of how a user logged
    in arent' important.


* `logout()`

    used to simulate the effect of a user logging out


---

### Testing responses

The `get()` and `post()` methods both return a `Response` object. This is _not_ the same as the HttpResponse object
returned by Django views.

Has the following attributes

**`class Response`**

* `client`  

    The tet client that was used to make the request that resulted in the response.

* `content`  

    The body of the response, as a bytestring. The final page content as rendered by the view, or any error message.

* `context`

    The template `Context` instance that was used to render the template that produced the response.

    If the rendered page used multiple templates, then `context` will be a list of `Context` objects, in
    the order in which they were rendered.

    Can retrieve context values using the `[]` operator:

    ```py
    >>> response = client.get('/foo/')
    >>> response.context['name']
    'Arthur`
    ```

* `exc_info`

    A tuple of three values that provides info about the unhandled exception, if any, that occurred during the view.

    The values are (type, value, traceback):
    * _type_: The type of exception
    * _value_: The exception instance
    * _traceback_:  

    If no exception occurred, then `exc_info` will be `None`

* `json(**kwargs)`

    The body of the response, parsed as JSON. Extra keyword arguments are passed to `json.loads()`:

    ```py
    >>> response = client.get('/foo/')
    >>> response.json()['name']
    'Arthur'
    ```

    If the `Content-Type` header is not `"application/json"`, then a `ValueError` will be raised.

* `request`

    The request data that stimulated the response.

* `wsgi_request`

    The `WSGIRequest` instance generated by the test handler that generated the response.

* `status_code`

    The HTTP status of the response, as an integer. For a full list of defined codes.

* `templates`

    A list of `Template` instances used to render the final content, in the order they were rendered.

    For each template in the list, use `template.name` to get the template's file name.

* `resolver_match`

    An instance of `ResolverMatch` for the response.

    ```py
    # my_view here is a function based view
    self.assertEqual(response.resolver_match.func, my_view)

    # class-based views need to be compared by name, as the functions
    # generated by as_view() won't be equal
    self.assertEqual(response.resolver_match.func.__name__, MyView.as_view().__name__)
    ```

---

### Exceptions

If you point the test client at a view that raises an exception and `Client.raise_request_exceptions` is `True`,
that exception will be visible in the test case.

You can then use a standard `try ... except` block or `assertRaises()` to test for exceptions.

The only exceptions that are not visible to the test client are `Http404`, `PermissionDenied`, `SystemExit`, and
`SuspiciousOperation`. These are caught internally. Check
`response.status_code` in test.

---

### Persistent state

A test client has two attributes that store persistent state information.

* `Client.cookies`

    A Python SimpleCookie object, containing the current values of all the client cookies.

* `Client.session`

    A dictionary-like object containing session information.

    To modify the session and then save it, it must be stored in a variable first:

    ```py
    def test_something(self):
        session = self.client.session
        session['somekey'] = 'test'
        session.save()
    ```

---

### Setting the language

---

### Example

```py
import unittest
from django.test import Client

class SimpleTest(unittest.TestCase):
    def setUp(self):
        # Every test needs a client
        self.client = Client()

    def test_details(self):
        # issue a GET request.
        response = self.client.get('/customer/details/')

        # Check that the response is 200 ok.
        self.assertEqual(response.status-code, 200)

        # Check that the rendered context contains 5 customers.
        self.assertEqual(len(response.context['customers']), 5)
```

---

## Provided test case classes

Django provides a few extensions of this base class:

![test case classes](django_unittest_classes_hierarchy.svg)

Convert a normal `unittest.TestCase` to any of the subclasses by changing the base class of the test

### `SimpleTestCase`

**`class SimpleTestCase`**
A subclass of `unittest.TestCase` that adds functionality:

* Some useful assertions:

    * Checking that a callable raises a certain exception
    * Checking that a callable triggers a certain warning
    * Testing form field rendering and error treatment
    * Testing HTML responses for the presence/lack of a given fragment.
    * Verifying that a template has/hasn't been used to generate a given response content.
    * Verifying that two URLs are equal
    * Verifying an HTTP redirect is performed by the app.
    * Robustly testing two HTML fragments for equality/inequality or containment
    * Robustly testing two XML fragments for equality/inequality
    * Robustly testing two JSON fragments for equality

* The ability to run tests with modified settings

* Usin the `client Client`

If your tests make any database queries, use subclasses `TransactionTestCase` or `TestCase`.

**`simpleTestCase.databases`**

`SimpleTestCase` disallows database queries by default.

---

### `TransactionTestCase`

**`class TransactionTestCase`**

TransactionTestCase inherits from `SimpleTestCase` to add some database-specific features:

* Resetting the database to a known state at the beginning of each test to ease testing and using the ORM

* Database `fixtures`

* Test skipping based on database backend features

* The remaining specialized `assert*` methods.

Django's `TestCase` class is a more commonly used subclass of `TransactionTestCase` that makes use of db transaction
facilities to speed up the process of resetting the db to a known state at the beginning of each test.

Some database behaviors cannot be tested with `TestCase` class. Example cannot test that a block of code is 
executing within a transaction, as is required when using 
`select_for_update()`. use `TransactionTestCase`.

The only difference is the way the database is reset to a known state and the ability for test code to test the 
effects of commit and rollback:

* A `TransactionTestCase` resets the database after the test runs by truncating all tables. 

* A `TestCase` does not truncate tables. It encloses the
test code in a database transaction that is rolled back at the end of the test.

---

### `TestCase`

**`class TestCase`**

The most common class to use for writing tests. If your Django application doesn't use a database use `SimpleTestCase`.

The class:

* Wraps the tests within two nested `atomic()` blocks: one for the whole class and one for each test.

* Checks deferrable database constraints at the end of each test

**`classmethod TestCase.setUpTestData()`**

The class-level `atomic` block described above allows the creation of initial data at the class level, once for the
whole `TestCase`. This technique allows for faster tests as compared to using `setUp()`.

Example:

```py
from django.test import TestCase

class MyTests(TestCase):
    @classmethod
    def setUpTestData(cls):
        # Set up data for the whole TestCase
        cls.foo = Foo.objects.create(bar="Test")
        ...
    
    def test1(self):
        # Some test using self.foo
        ...

    def test2(self):
        # Some other test using self.foo
        ...
```

---

### `LiveServerTestCase`

**`class LiveServerTestCase`**

Does basically the same as `TransactionTestCase` but it also launches a live Django server in the background on setup.

Allows th use of automated test clients like the Selenium client, to execute functional tests inside a browser
and simulate a real user's actions.

The live server listens on `localhost` and binds to port 0.
The server's URL can be accessed with `self.live_server_url`

Try it with Selenium:

```
$ python -m pip install selenium
```

Then add a `LiveServerTestCase`-based test to tests module.

```py
from django.contrib.staticfiles.testing import StaticLiveServerTestCase
from selenium.webdriver.firefox.webdriver import WebDriver

class MySeleniumTests(StaticLiveServerTestCase):
    fixtures = ['user-data.json']

    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.selenium = WebDriver()
        cls.selenium.implicitly_wait(10)

    @classmethod
    def tearDownClass(cls):
        cls.selenium.quit()
        super().tearDownClass()

    def test_login(self):
        self.selenium.get('%s%s' % (self.live_server_url, '/login/'))
        username_input = self.selenium. find_element_by_name("username")
        username_input.send_keys('myuser')
        password_input = self.selenium.find_element_by_name("password")
        password_input.send_keys('secret')
        self.selenium.find_element_by_xpath('//input[@value="Log in"]').click()

```

Then run the test:

```
$ ./manage.py test myapp.tests.MySeleniumTests.test_login
```

---


## Test cases features

### Default test client

* `SimpleTestCase.client`




## Testing asynchronous code

## Email services

## Management Commands

## Skipping tests