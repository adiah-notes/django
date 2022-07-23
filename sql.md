# SQL, Models, and Migrations

## Data

We are going to store data in a table

origin | destination | duration
 --- | --- | --- 
New York | London | 415
Shanghai | Paris | 760
Istanbul | Tokyo | 700

## SQL
SQL lets us manage relational databases

Types (SQLite):

* Text
* Numeric
* Integer
* Real
* Blob

Create Table

    CREATE TABLE flights (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        origin TEXT NOT NULL,
        destination TEXT NOT NULL,
        duration INTEGER NOT NULL
    );

Restraints:
* check 
* default
* not null
* unique

Insert into table

    INSERT INTO flights
        (origin, destination, duration) VALUES ("New York", "London", 415);


Select

    SELECT * FROM flights;
    SELECT origin, destination FROM flights;
    SELECT * FROM flights WHERE id = 3;
    SELECT * FROM flights WHERE origin = "New York";


create a sqllite database

`ni flights.sql`

`sqlite3 flights.sql`

    CREATE TABLE flights (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        origin TEXT NOT NULL,
        destination TEXT NOT NULL,
        duration INTEGER NOT NULL
    );

`.mode columns`


    SELECT * FROM flights WHERE destination > 500 AND destination = "Paris";
    SELECT * FROM flights WHERE origin IN ("Lima", "New York")
    SELECT * FROM flights WHERE origin LIKE "%a%";

Can use functions:
* average
* sum
* max

Update

    UPDATE flights
        SET duration = 430
        WHERE origin = "New York"
        AND destination = "London";

Delete

    DELETE FROM flights WHERE origin = "Paris";

* LIMIT
* ORDER BY 
* GROUP BY 
* HAVING

### Foreign keys

Separate table for airports


## Join

    SELECT first, origin, destination
        FROM flights JOIN passengers
        ON passengers.flight_id  = flights.id;


## Create Index

    CREATE INDEX name_index ON passengers (last);

## SQL Injection


## Models

In each django app in a project, there is a models.py file, each model can represent a table
    
    class Flight(models.Model):
        origin = models.Charfield(max_lenght=64)
        destination = models.Charfield(max_length=64)
        duration = models.IntegerField()

Tell Django to update database using the models **Migration**
creating the instructions and then actually apply them

    python manage.py makemigrations

the migration file gives instructions to change the database

`python manage.py migrate` to apply the migrations

enter shell:

    python manage.py shell`
    from flights.models import Flight
    f = Flight(origin="New York", destination="London", duration=415)
    f.save()
    Flight.objects.all()


Any model can use:

    def __str__(self):
        return f"{self.id}: {self.origin} to {self.destination}

this is just a cleaner string representation of the name


    flight = flights.first()
    flight.id
    flight.origin


In the shell:

    from models import *
    jfk = Airport(code="JFK", city="New York")
    jfk.save()
    lhr = Airport(code="LHR", city="London")
    lhr.save()
    f = Flight(origin=jfk, destination=lhr, duration=415)
    f.save()

    f
    f.origin
    f.origin.city
    f.origin.code

    lhr.arrivals.all()


filtering

    Airport.objects.filter(city="New York").first()
    Airport.objects.get(city="New York")

Get only works if there is going to be only one result


    jfk = Airport.objects.get(city="New York")
    cdg = Airport.objects.get(city="Paris")
    f = Flight(origin=jfk, destination=cdg, duration=435)
    f.save()o


## Django Admin app
create an admin account

    python manage.py createsuperuser

add models to admin.py
    
    from .models import Flight, Airport

    admin.site.register(Airport)
    admin.site.register(Flight)

customize admin interface

in admin.py

    class FlightAdmin(admin.ModelAdmin):
        list_display = ("origin", "destination", "duration")

    class PassengerAdmin(admin.ModelAdmin):
        filter_horizontal = ("flights", )

    admin.site.register(Flight, FlightAdmin)


## Authentication


