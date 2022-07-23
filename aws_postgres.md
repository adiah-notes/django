Go to aws
* search for s3 bucket

* create bucket
	* give it name
	* take off block all public access
	* leave everything else
* open bucket

* go to permissions and add bucket policy to have read options

* Search iam user

* create new user
	* give programmatic access
	* attach policies
		* amazons3fullaccess

* get the access key and secret access key

in settings.py

first `pip install boto3` and `pip install django-storages`

add `storages` to installed apps

Get these from the docs

Store these in environment variables

```py
AWS_QUERYSTRING_AUTH = False

DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
AWS_ACCESS_KEY_ID = 
AWS_SECRET_ACCESS_KEY = 

AWS_STORAGE_BUCKET_NAME = 'photoshare-django'

```


# Postgres

To use `pip install psycopg2`

create the database using pg admin or maybe the cli

and use the name from there in `settings.py`

```py
DATABASES = {
'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'DEMO_TEST',
        'USER': 'postgres',
        'PASSWORD': os.environ.get('PG_PASS'),
        'HOST': 'localhost',
        'PORT': '5432'

    }
}
```

then run `./manage.py migrate`


## Create postgres db on aws

search for RDS

click create database

select postgres

and free tier

and add a username and password

Set public access to yes

note the database port 5432 in vpc

add a default database name

get the endpoint for the database

update the inbound security groups and add a rule for
postgresql with incoming from anywhere 


In pgadmin, create a new server
and for the host/name address enter the endpoint.

add the username and password for it.

update the credetials in settings

apply the migrations



# Using postgres on heroku

`pip install dj_database_url`
`import dj_database_url`

Add this uder DATABASES	:

```py
import dj_database_url
db_from_env = dj_database_url.config(conn_max_age=600)
DATABASES['default'].update(db_from_env)
```

```
git add .
git commit -m "Added database conn"
git push heroku main
```

Migrate the tables to the database

```
heroku run python manage.py migrate
heroku run python manage.py createsuperuser
```


## Using postgres from aws

set a `DATABASE_URL` config 
and the value:

`postgres://USERNAME:PASSWORD@database url endpoint:PORT/DB NAME`

In aws allow ip addresses to access database




---


<br>

