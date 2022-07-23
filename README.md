# Django

Useful for creating for pages that are not static. ie dynamic web applications.

install django `pip install Django`

`django-admin startproject Project Name`

this makes a directory called the ProjectName with some files in it

* settings.py 
* urls.py  
    like a table of contents of all urls in web app


To run web app, in terminal: 
`python manage.py runserver`

A Django Project has many django applications within it
can have only 1 app but has the capacity for more

`ctrl + C` to exit server

Create an application 
`python manage.py startapp <appname>`

creates a directory <appname> with files for app

* views.py  
    contains the routes for the particular application.


Go to seetings.py and into Installed_Apps and add it there

then create the views/routes for your application

### views

need to define a function with an argument that represents the http request


each app should have its own urls.py file inside needs a variable called urlpatterns which is a list of all the available urls that can be accessed

import path from django.urls

    from django.urls import path
    from . import views

    urlpatterns = [
        path("", views.index, name="index")
    ]
    
where views.index represent a particular view

then go to the main urls.py file
and add a path to the hello app

`path('hello/', include("hello.urls"))`


can create url patterns with placeholders

    def greet(request, name):
        return HttpResponse(f"Hello, {name.capitalize()}!")

in urls.py:

    path("<str:name>", views.greet, name="greet")



Separate html from the actual python code

instead of `HttpResponse`

`return render(request, "hello/index.html")`

* create a new directory called templates inside of hello (the application directory)
    then create directory called hello inside templates to namespace the urls

can treat html as template using django, so variables can be used

    def greet(request, name):
        return render(request, "hello/greet.html",{
            "name": name.capitalize()
        })

the dictionary is called the context


Unchanging files are stored as static files: eg css etc
don't hardcode the href url for the static files


Django middleware lets you authenticate csrf tokens automatically

There is also built in forms that allow you to do client-side validation,
Server side validation should always be done in addition.

`form = NewTaskForm(request.POST)` gives all the information that the user entered when the form was submitted

store tasks in users session route

need to create django_session table, or allow it to be created
`python manage.py migrate`


## Wiki project


