# Authentication and authorization


Django's built-in permissions system provides a way to assign permissions to 
specific users and groups of users.

* Access to viw objects is limited to users with the "view" or "change" permission for that type of object.

* Access to view the "add" form and add an object is limited to users with the 
"add" permission for that type of object.

* Access to view the change list, view the "change" form and change an object is limited to users with the "change" permission for that type of object.

* Access to delete an object is limited to users with the "delete" permission for that type of object.

Can set permissions on a type of object and also a specific object instance.
By using the `has_view_permission()`, `has_add_permission()`, `has_change_permission()` and `has_delete_permission`
methods of the `ModelAdmin` class.


`User` objects have `groups` and `user_permissions` which are many-to-many fields, and can be accessed:

```py
myuser.groups.set([group_list])
myuser.groups.add(group, group, ...)
myuser.groups.remove(group, group, ...)
myuser.groups.clear()
myuser.user_permissions.set([permission_list])
myuser.user_permissions.add(permission, permission, ...)
myuser.user_permissions.remove(permission, permission, ...)
myuser.user_permissions.clear()
```


## Default Permissions

Once `django.contrib.auth` is in `INSTALLED_APPS`, the four default permissions: add, change, delete, and view are
created for each model. The function that creates permissions is connected to the `post_migrate`signal.

For an application with `app_label` foo and a model named Bar, to test for basic permissions:

* add: `user.has_perm('foo.add_bar')`
* change: `user.has_perm('foo.change_bar')`
* delete: `user.has_perm('foo.delete_bar')`
* view: `user.has_perm('foo.view_bar')`

The `Permission` model is rarely accessed directly.


## Groups

`django.contrib.auth.models.Group` models are a generic way of categorizing users to apply permissions, or some 
other labe, to those users. A user can belong to any number of groups.

A user in a group automatically has the permission granted to that group.

Groups are also a convenient way to categorize users to give them some label, or extend functionality.


## Programmatically creating permissions

Custom permissions can be defined with a model's `Meta` class, and also directly. Example create the 
`can_publish` permission for a `BlogPost` model in `myapp`:

```py
from myapp.models import BlogPost
from django.contrib.auth.models import Permission
from django.contrib.contenttypes.models import ContentType

content_type = ContentType.objects.get_for_model(BlogPost)
permission = Permission.objects.create(
    codename='can_publish',
    name='Can Publish Posts',
    content_type=content_type,
)
```

The permission can then be assigned to a `User` via its `user_permissions` attribute or to a `Group` via its
`permissions` attribute.


## Permission caching

The `ModelBackend` caches permissions on the user object. If you add permissions and need to check them immediately
afterward, in a test/view for example, the easiest solution is to re-fetch the user from the database:

```py
from django.contrib.auth.models import Permission, User
from django.contrib.contenttypes.models import ContentType
from django.shortcuts import get_object_or_404

from myapp.models import BlogPost

def user_gains_perms(request, user_id):
    user = get_object_or_404(User, pk=user_id)
    # Any permission check will cache the current set of permissions
    user.has_perm('myapp.change_blogpost')

    content_type = ContentType.objects.get_for_model(BlogPost)
    permission = Permission.objects.get(
        codename='change_blogpost',
        content_type=content_type,
    )
    user.user_permissions.add(permission)

    # Checking the cached permission set
    user.has_perm('myapp.change_blogpost') # False

    # Request new instance of User
    # Be aware that user.refresh_from_db() won't clear the cache.
    user = get_object_or_404(User, pk=user_id)

    # Permission cache is repopulated from the database
    user.has_perm('myapp.change_blogpost')  # True
```


## Authentication in web requests

A `request.user` attribute is provided on every request which represents the current user.
Is set to `AnonymousUser` if not logged in, otherwise it's an instance of `User`.

use:
```py
if request.user.is_authenticated:
    # Do something for authenticated users.
    ...
else:
    # Do something for anonymous users.
    ...
```

### Log a user in

```py
from django.contrib.auth import authenticate, login

def my_view(request):
    username = request.POST['username']
    password = request.POST['password']
    user = authenticate(request, username=username, password=password)
    if user is not None:
        login(request, user)
        # Redirect to a success page.
        ...
    else:
        # Return an 'invalid login' error message.
        ...
```

### Log a user out

```py
from django.contrib.auth import logout

def logout_view(request):
    logout(request)
    # Redirect to a success page.
```

`logout` doesn't throw any errors if the user wasn't logged in. It cleans out session data.
If anything needs to be in the session data that will be available after logging out, do that 
_after_ calling `django.contrib.auth.logout()`.


## Limiting access to logged-in users

One option is to use the `login_required` decorator:

```py
from django.contrib.auth.decorators import login_required

@login_required
def my_view(request):
    ...
```

* If the user isn't logged in, redirect to `settings.LOGIN_URL`, passing the current absolute path in the
query string. Example: `/accounts/login/?next=/polls/3/`.

* If the user is logged in, execute the view normally. The view code is free to assume the user is logged in.

`login_required()` takes an optional `redirect_field_name` parameter.

```py
from django.contrib.auth.decorators import login_required

@login_required(redirect_field_name='my_redirect_field')
def my_view(request):
    ...
```

Also takes an optional `login_url` parameter:
```py
from django.contrib.auth.decorators import login_required

@login_required(login_url='/accounts/login/')
def my_view(request):
    ...
```

If you don't specify the `login_url` parameter, ensure that the `settings.LOGIN_URL` and login view are properly
associated.


### Limiting access to logged-in users that pass a test

To limit access based on certain permissions or some other test, can use the `user_passes_test` decorator.

```py
from django.contrib.auth.decorators import user_passes_test

def email_check(user):
    return user.email.endswith('@example.com')

@user_passes_test(email_check)
def my_view(request):
    ...
```


Takes a required argument: a callable that takes a `User` object and returns `True` if the user is allowed to 
view the page. 

Two optional arguments:

* `login_url`: lets you specify the URL that users who don't pass the test will be redirected to. Defaults to
`settings.LOGIN_URL`

* `redirect_field_name`: Same as for the `login_required()`. setting it to `None` removes it from the URL, userful
if you redirect users that don't pass the test to a non-login page where there's no next page.

```py
@user_passes_test(email_check, login_url='/login/)
def my_view(request):
    ...
```

### The `permission_required` decorator

`permission_required(perm, login_url=None, raise_exception=False)`

```py
from django.contrib.auth.decorators import permission_required

@permission_required('polls.add_choice')
def my_view(request):
    ...
```

Permission names take the form `"<app label>.permissioncodename>"`

The decorator may also take an iterable of permissions, in which case the user must have all of the permissions
in order to access the view.

Also has a `login_url`.

If the `raise_exception` parameter is given, the decorator will raise `PermissionDenied` prompting the 403 view
instead of redirecting to the login page.

Can raise an exception and give users a chance to login first by adding `login_required` decorator.

```py
from django.contrib.auth.decorators import login_required, permission_required

@login_required
@permission_required('polls.add_choice', raise_exception=True)
def my_view(request):
    ...
```

## Authentication Views

Django provides several views that you can use for handling login, logout, and password management.
These use stock auth forms, but you can pass in your own forms as well.

There is no default template for the authentication views however.
Create your own templates for the views you want.

Different method to implement the views in your project. The easies way is to include the provided URLconf
in `django.contrib.auth.urls` in your own URLconf:

```py
urlpatterns = [
    path('accounts/', include('django.contrib.auth.urls')),
]
```

This includes the following URL patterns:

```py
accounts/login/ [name='login']
accounts/logout/ [name='logout']
accounts/password_change/ [name='password_change']
accounts/password_change/done/ [name='password_change_done']
accounts/password_reset/ [name='password_reset']
accounts/password_reset/done/ [name='password_reset_done']
accounts/reset/<uidb64>/<token>/ [name='password_reset_confirm']
accounts/reset/done/ [name='password_reset_complete']
```

For more control over urls, reference specific views in URLconf:

```py
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('change-password/', auth_views.PasswordChangeView.as_view()),
]
```

The views have optional arguments, example providing the `template_name` argument.

```py
urlpatterns = [
    path(
        'change-password/',
        auth_views.PasswordChangeView.as_view(template_name='change-password.html'),
    ),
]
```

All views are class-based


### class LoginView

**URL name**: login

**Methods and Attributes**

* **template_name**: the name of a template to display for the view used to log the user in. Defaults to 
`registration/login.html`

* **next_page**: The URL to redirect to after login. Defaults to `LOGIN_REDIRECT_URL`

* **redirect_field_name**: The name of a `Get` field containing the URL to redirect to after login. Defaults to 
`next`. Overrides the `get_default_redirect_url()`.

* **authentication_form**: A callable (typically a form class) to use for authentication. Defaults to 
`AuthenticaonForm`.

* **extra_context**: A dictionary of context data that will be added to the default context data passed to the
template.

* **redirect_authenticated_user**: A boolean that controls whether authenticated users accessing the login page
will be redirected as if they had just successfully logged in. Defaults to `False`.

    Can be used for social media fingerprinting: host all images and favicon on separate domain if this is 
    active.

* **success_url_allowed_hosts**: A set of hosts, in addition to `request.get_host()`, that are safe for
redirecting after login. Defaults to an empty set.

* **get_default_redirect_url()**: Returns the URL to redirect to after login. Default resolves and returns 
`next_page` if set, or `LOGIN_REDIRECT_URL` otherwise


What `LoginView` does:

* If called via `GET`, it displays a login form that POSTs to the same URL.

* If called via `POST` with user submitted credentials, it tries to log the user in. If login is successful, 
the view redirects to the URL specified in `next`. If `next` isn't provided, it redirects to 
`settings.LOGIN_REDIRECT_URL` (which defaults to `/accounts/profile/`). If login isn't successful, it 
displays the login form.

Have to provide the html for the login template, called `registration/login.html` by default. It gets passed
four template context variables:

* `form`: A `Form` object representing the `AuthenticationForm`.
* `next`: The URL to redirect to after successful login. This may contain a query string, too.
* `site`: The current `Site`, according to the `SITE_ID` setting. If the site framework isn't installed, thi
will be an instance of `RequestSite`, which derives the site name and domain from the current `HttpRequest`.
* `site_name`: An alias for `site.name`.

To call the template something else, can pass the `template_name` parameter via the extra arguments to the
`as_view` method. Example to use `myapp/login.html`:

```py
path('accounts/login/', authviews.LoginView.as_view(template_name='myapp/login.html')),
```


Sample template:

```html
{% extends 'base.html' %}

{% block content %}

{% if form.errors %}
<p>Your username and password didn't match. Please try again.</p>
{% endif %}

{% if next %}
    {% if user.is_authenticated %}
    <p>
        Your account doesn't have access to this page. To proceed, please login with an account that has
        access.
    </p>
    {% else %}
    <p>Please login to see this page.</p>
    {% endif %}
{% endif %}

<form method="post" action="{% url 'login' %}">
{% csrf_token %}
<table>
<tr>
    <td>{{ form.username.label_tag }}</td>
    <td>{{ form.username }}</td>
</tr>
<tr>
    <td>{{ form.password.label_tag }}</td>
    <td>{{ form.password }}</td>
</tr>
</table>

<input type="submit" value="login">
<input type="hidden" name="next" value="{{ next }}">
</form>

{# Assumes you set up the password_reset view in your URLconf #}
<p><a href="{% url 'password_reset' %}">Lost password?</a></p>


{% endblock %}
```

### class `PasswordChangeView`

**URL name**: `password_change`

**Attributes**:

* **template_name**: The full name of a template to use for displaying the password change form.

* **success_url**: The URL to redirect to after a successful password change. Defaults to '`password_change_done`'.

* **form_class**: A custom "change password" form which must accept a `user` keyword argument. The form is
responsible for actually changing the user's password. Defaults to `PasswordChangeForm`.

* **extra_context**: A dictionary of context data that will be added to the default context data passed to the 
template.

**Template context**:

* **form**: The password change form.


### class `PasswordChangeDoneView`

**URL name**: password_change_done

**Attributes**:

* **template_name**

* **extra_context**


### class `PasswordResetView`

**URL name**: password_reset

Allows a user to reset their password by generating a one-time use link that can be used to reset the password.

Sends an email if:

* The email address provided exists in the system.
* The requested user is active (`User.is_active` is `True`)
* The requested user has a usable password.

If any are not met, the email won't be sent but no error message either. To provide an error message, subclass
`PasswordResetForm` and use the `form_class` attribute

**nb** consider using a 3rd party package to send emails asynchronously, eg django-mailer.

**Attributes**:

* **template_name**

* **form_class**: Form that will be used to get the email of the user to reset the password for. Defaults to 
`PasswordResetForm`.

* **email_template_name**: The full name of a template to use for generating the email with the reset password
link.

* **subject_template_name**: The full name of a template to use for the subject of the email with the reset
password link.

* **token_generator**: Instance of the class to check the one time link. This will default to `default_token_generator`, 
it's an instance of `django.contrib.auth.tokens.PasswordResetTokenGenerator`.

* **success_url**

* **from_email**: A valid email address. By default Django uses the `DEFAULT_FROM_EMAIL`.jK:w

* **extra_context**

* **html_email_template_name**: The full name of a template to use for generating a text/html multipart email
with the password reset link. By default, HTML email is not sent.

* **extra_email_context**: A dictionary of context data that will be available in the email template. It can be used
to override default template context values listed below e.g. `domain`.

**Template context**:

* **form**: The form for resetting the user's password.

**Email template context**:

* **email**: An alias for `user.email`

* **user**: The current `User` according to the `email` form field. Only active users are able to reset their
passwords.

* **site_name**: An alias for `site.name`.

* **domain**: An alias for `site.domain`. If you don't have the site framework installed, this will be set to the
value of `request.get_host()`.

* **protocol**: http or https

* **uid**: The user's primary key encoded in base 64.

* **token**: Token to check that the reset link is valid.

Sample template (email body template):

```jinja
Someone asked for password reset for email {{ email }}. Follow the link below:
{{ protocol }}://{{ domain }}{% url 'password_reset_confirm' uidb64=uid token=token %}
```

The same template context is used for subject template.

<br>

### class `PasswordResetDoneView`

**URL name**: password_reset_done

**Attributes**:

* **template_name**

* **extra_context**

<br>

### class `PasswordResetConfirmView`

**URL name**: password_reset_confirm

**Keyword arguments from the URL:**

* **uidb64**: The user's id encoded in base 64.
* **token**: Token to check that the password is valid.


**Attributes:**

* **template_name**

* **token_generator:** Instance of the class to check the password.

* **post_reset_login:** A boolean indicating if the user should be automatically authenticated after a successful
password reset. Defaults to `False`.

* **post_reset_login_backend:** A dotted path to the authentication backend to use when authenticating a user.

* **form_class:** Form that will be used to set the password. Defaults to `SetPasswordForm`.

* **success_url**

* **extra_context**

* **reset_url_token:** Token parameter displayed as a component of password reset URLS. Defaults to `set-password`.


**Template context:**

* **form:** The form for setting the new user's password.
* **validlink:** Boolean, True if the link is valid or unused yet.

<br>

### class `PasswordResetCompleteView`

**URL name:** password_reset_complete

**Attributes:**

* **template_name**
* **extra_context**

---

<br>

## Helper functions

`redirect_to_login(next, login_url=None, redirect_field_name='next')`

Redirects to the login page, and then back to another URL after a successful login.

**Required arguments:**
* **next:** The URL to redirect to after a successful login.

**Optional arguments:**
* **login_url:** The URL of the login page to redirect to. Defaults to `settings.LOGIN_URL` if not supplied.

* **redirect_field_name:**


## Built-in forms


## Authentication data in templates

### Users

When rendering a RequestContext, the currently logged-in user, either a `User` instance or an `AnonymousUser`
instance is stored in the template variable {{user}}

```html
{% if user.is_authenticated %}
    <p>Welcome, {{ user.username }}. Thanks for logging in.</p>
{% else %}
    <p>Welcome, new user. Please log in.</p>
{% endif %}
```


### Permissions

The currently logged-in user's permissions are stored in the template variable `{{ perms }}`.
Evaluating a single-attribute lookup of `{{ perms }}` as a boolean is a proxy to `User.has_module_perms()`.
For example, to check if the logged-in user has any permissions in the `foo` app:

    {% if perms.foo %}

Evaluating a two-level-attribute lookup as a boolean is a proxy to `User.has_perm()`.
For example, to check if the logged-in user has the permission `foo.add_vote`:

    {% if perms.foo.add_vote %}

A more complete example of checking permissions in a template.

```html
{% if perms.foo %}
    <p>You have permission to do something in the foo app.</p>
    {% if perms.foo.add_vote %}
        <p>You can vote!</p>
    {% endif %}
    {% if perms.foo.add_driving %}
        <p>You can drive!</p>
    {% endif %}
{% else %}
    <p>You don't have permission to do anything in the foo app.</p>
{% endif %}
```

It is possible to also look permissions up by `{% if in %}` statements:

```html
{% if 'foo' in perms %}
    {% if 'foo.add_vote' in perms %}
        <p>In lookup works, too.</p>
    {% endif %}
{% endif %}
```

---

## Authentication settings

* **AUTH_USER_MODEL**
    Default: `auth.User`  
    The model to use to represent a User. Change this before any migrations are made.

* **LOGIN_REDIRECT_URL**
    Default: `'/accounts/profile/'`

    The URL or named URL pattern where requests are redirected after login when the `LoginView` doesn't get a 
    `next` GET parameter.

* **LOGIN_URL**
    Default: `'/accounts/login/`

    The URL or named URL pattern where requests are redirected for login when using the `login_required()`
    decorator.

* **LOGOUT_REDIRECT_URL**
    Default: `None`

    The URL or named URL pattern where requests are redirected after logout if `LogoutView` doesn't have a 
    `next_page` attribute.

    If `None`, no redirect will be performed.

* **PASSWORD_RESET_TIMEOUT**
    Default: 259200 (3 days, in seconds)

    the number of seconds a password reset link is valid for.

* **PASSWORD_HASHERS**

* **AUTH_PASSWORD_VALIDATORS**
    Default: `[]` 