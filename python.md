# Python

## Hello, World

    print("Hello, world!")

## Variables

    a = 28
    b = 1.5
    c = "Hello!"
    d = True
    e = None

Variables do not need to be "typed"

NoneType is a special type of variable

### fStrings

`print(f"Hello, {name}")`

## Conditions

- Remember that input only takes strings as input, so you need to convert it

## tupples

Pairs of two values for eg coordinates

    coordinate = (10.0, 20.0)

## Data Structures

- list - sequence of mutable values
- tuple - sequence of immutable values
- set - collection of unique values
- dict - collection of key-value pairs

Can use `len` with all

### Lists

Useful for storing items when the order is important.

`names.append('Draco')` - adds item to end of list
`names.sort()` - sorts list

### Sets

items cannot repeat

### Dictionaries

Stores info as key-value pairs

create dictionary:  
-> use `{}`

dictionaries cannot contain duplicate keys

```py
monthConversions = {
    "Jan": "January",
    "Feb": "February",
    "Mar": "March"
}
```

```py
print(monthConversions["Nov"]) # searches for key
```

```py
print(montConversions.get("Dec")) #-> can specify a default value if this isn't found
.get("Luv", "Not a valid key")
```

keys don't have to be strings, they could be numbers

## Loops

## Object Oriented Programming

Can create a class of objects to represent something

    class Point():
        def __init__(self, x, y):
            self.x = x
            self.y = y

    p = Point(2, 8)
    print(p.x)
    print(p.y)

init is used to create object
self represents the object itself
can store things about the object within it's self

## Lambda

    people = [
        {'name': 'harry', 'house': 'Gryffindor'},
        {'name': 'Cho', 'house': 'Ravenclaw'}
    ]

    def f(person):
        return person['name']

    people.sort(key=f)

    print(people)

can use `people.sort(key=lamda person: person['name'])`
where the function is simple and only used once

## Exceptions

    try:
        result = x/y
    except ZeroDivisionError:
        print(Error:Cannot divide by zero)
        sys.exit(1)
