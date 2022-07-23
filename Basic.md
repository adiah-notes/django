# Initial setup 

Install django in virtual environment as usual.

`django-admin startproject studybud`

Create app:

`manage.py startapp base`

and add it in the `settings.installed apps`

	'base.apps.BaseConfig',

Create some views in `base.views` and create `urls.py` in `base`

Then `include` the `base.urls` in the `studybud.urls`

# Templates

as usual

or if putting them all in one templates in the root folder,
go to `TEMPLATES` in `settings` and update the dirs with `BASE_DIR / 'templates'`


Using template tags and template inheritance

`{% include 'base/navbar.html' %}` and `{% extends 'base/layout.html'%}`





# Templates

put the templates that don't have to do with any particular app in the root templates folder

# Django Template Engine

`{{ variables }}`

`{% python-like logic %}`

# Dynamic url routing

use `<str:pk>`

and use `href="{% url 'room' room.id %}"`

# Database

Run the `manage.py migrate`

Create database tables in `models.py`.

register models in `admin.py`

Make queries of the database `queryset = ModelName.objects.all()`

Relationships:

`room = models.ForeignKey(Room, on_delete=models.CASCADE)`

django has a default User model already

`from django.contrib.auth.models import User`

Can customize this

Add a `meta` class to a model for ordering

``` python
class Meta:
		ordering = ['-updated', '-created']
```

# CRUD

## Forms

need a `{% csrf_token %}`

Use a model form, a class based representation of a form in `forms.py`.

	class RoomForm(ModelForm):
		class Meta:
			model = Room
			fields = '__all__'

If you specify `'__all__'`, it will give all editable fields, or you can create a list with the field names
`['name', 'user']`

Inside of `views.py`, import the form (eg `from .forms import RoomForm`) and then assign it in the view function
and add to the context:

	form = RoomForm()
	context = {'form': form}

Then add it to the template by simply calling the form variable: `{{form}}` or something like `{{form.as_p}}` or
with tags `{{forms|crispy}}`.

### Creating

Take in form data by checking `request.method == 'POST'`
can check the data manually, but since using the model form, just do:

```python
if request.method == 'POST':
	form = RoomForm(request.POST)
	if form.is_valid():
		form.save()
		return redirect('home')
```

And this will save this model to the database. And the `redirect` will take the user back to the home page.

### Updating

Pass an instance to the same form containing the specific item from the model/database that will populate the form.

Also pass the instance when processing the form data.

```py
def updateRoom(request, pk):
	room = Room.objects.get(id=pk)
	form = RoomForm(instance=room)

	if request.method == 'POST':
		form = RoomForm(request.POST, instance=room)
		if form.is_valid():
			form.save()
			return redirect('home')

	context = {}

	return render(request, 'base/room_form.html', context)
```

### Deleting

Create a template that asks for confirmation in a form or to go back.

Here's a way to go back to previous view using template tags to reference the `request` object.

```html
<a href="{{request.META.HTTP_REFERER}}">Go Back</a>
```

And add the view

```py
def deleteRoom(request, pk):
	room = Room.objects.get(id=pk)

	if request.method == 'POST':
		room.delete()
		return redirect('home')

	return render(request, 'base/delete.html', {'obj': room})
```


# Searching

This is a way to create a side bar with links to filter/search the objects on the page.  
So each topic in this case will be a link that just goes back to the same homepage, but with the `q` search parameter
added to the url.

```html
{% for topic in topics %}
	<div>
		<a href="{% url 'home' %}?q={{topic.name}}">{{topic.name}}</a>
	</div>
{% endfor %}
```

Add this to the view to filter the results based on the search parameter specified
```py
q = request.GET.get('q') if request.GET.get('q') != None else ''
	rooms = Room.objects.filter(topic__name__icontains=q)
```

Can also use 
`from django.db.models import Q`
to do a qlookup

```py
rooms = Room.objects.filter(
	Q(topic__name__icontains=q) |
	Q(name__icontains=q) |
	Q(description__icontains=q)
)
```
This allows chaining of query methods, using `and` or `or` as in here

Can do something like `room_count = rooms.count()` and pass that in the context, quicker than the `len` method.


# Authentication

Django has a database table called `sessions` for authentication. The backend sends back a session token.
It's stored in the cookies of the browser `sessionid`. Django takes care of creating and checking it.



## Logging in

Create a `loginPage` view, and import django's `authenticate` and `login` methods.

get the username and password from the `POST` request and use it to authenticate and login user.

```py
def loginPage(request):
	if request.method == 'POST':
		username = request.POST.get('username')
		password = request.POST.get('password')

		try: 
			user = User.objects.get(username=username)
		except:
			messages.error(request, 'User does not exist')

		user = authenticate(request, username=username, password=password)

		if user is not None:
			login(request, user)
			return redirect('home')
		else:
			messages.error(request, 'Username or password does not exist')

	context={}
	return render(request, 'base/login_register.html', context)

```

Put this into your template to get the flashed messages:

```html
	{% if messages %}
		<ul class="messages">
			{% for message in messages %}
				<!-- <li {% if message.tags %} class="{{message.tags}}" {% endif %}>{{message}}</li> -->
				<li>{{message}}</li>
			{% endfor %}
		</ul>
	{% endif %}
```

Ideally in the `main.html` so that all views will get messages

## Logout

Only needs a view not a template.

```py
def logoutUser(request):
	logout(request)
	return redirect('home')
```

Can access the user in the template 

``` 
{% if request.user.is_authenticated %}
```

## Restricting views 

```py
from django.contrib.auth.decorators import login_required
```

Add decorator to a view where the user needs to be logged in:

```py
@login_required(login_url='login')
def view(request):
	...
```

the decorator also takes a `login_url` parameter that tells where to send the user if they are not logged in.

Here is a simple way to only allow only the user who created the object to update/delete

```py
if request.user != room.host:
		return HttpResponse('You are not allowed here')
```

Hide the links in the templates similarly

```py
{% if request.user == room.host %}
	<a href="{% url 'update-room' room.id %}">Edit</a>
	<a href="{% url 'delete-room' room.id %}">Delete</a>
{% endif %}
```


## Registration

Use the built-in django `UserCreationForm`

```py
def registerUser(request):
	page = 'register'
	form = UserCreationForm()

	if request.method == 'POST':
		form = UserCreationForm(request.POST)
		if form.is_valid():
			# Freezing the form to access the user right away
			user = form.save(commit=False)
			user.username = user.username.lower()
			user.save()
			login(request, user)
			return redirect('home')
		else:
			messages.error(request, 'An error occurred during registration')

	context={
		'page': page,
		'form': form
	}
	return render(request, 'base/login_register.html', context)
```

# Query

```py
messages = room.message_set.all()
```
Get the set of messages related to this specific room


Many to many relation with related name if the related field is already in use.
```py
participants = models.ManyToManyField(User, related_name='participants')
```


# Static files

If you create a static folder in the root, like the templates folder, do this to let django know about it

```py
STATICFILES_DIRS = [
    BASE_DIR / 'static'
]
```

Remember to load static files at the top: `{% load static %}` of the html doc.

Then call the file with the `static` tag:

```html
<link rel="stylesheet" href="{% static 'styles/main.css' %}">
```

# Installing a theme

Copy the html and then configure the static files and links to the django way.


# Building the api

Using `djanogrestframework`

along with django-cors-headers

# Custom User Model

Adding a one to one relation to a profile model.

Or actually modifying the user model

in `models.py`

```py
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
	pass
```

in `settings.py`

```py
AUTH_USER_MODEL = 'base.User'
```

Run some migrations

register in `admin.py`

```py
from .models import User

admin.site.register(User)

```

Now actually customize the model

```py
class User(AbstractUser):
	name = models.CharField(max_length=200, null=True)
	email = models.EmailField(unique=True)
	bio = models.TextField(null=True)

	USERNAME_FIELD = 'email'
	REQUIRED_FIELDS = []
```

Run migrations


# Image field

`pip install pillow`

in `settings.py`

```py
MEDIA_URL = '/images/'

MEDIA_ROOT = BASE_DIR / 'static/images/
```

and then add a path to the media urls in the main `urls.py` file

```py
from django.conf import settings
from django.conf.urls.static import static

urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

Call it in the template by `user.avatar.url`


and in forms add in `enctype="multipart/form-data"`

and in the views when the form takes in files:
`form = UserForm(request.POST, request.FILES, instance=user)`