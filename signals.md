# Signals

Signals allow certain senders to notify a set of receivers that some action has taken place.

To receive a signal, register a receiver function using the `Signal.connect()` method.
The receiver function is called when the signal is sent.

`Signal.connect(receiver, sender=None, weak=True, dispatch_uid=None)`  

Parameters:
* receiver - the callback function which will be connected to this signal.
* sender - specifies a particular sender to receive signals from
* weak - pass `weak=False` when calling the `connect()` method
* dispatch_uid - A unique identifier for a signal receiver in cases where duplicate signals may be sent


Example - Registering a signal that gets called after each HTTP request is finished. Connecting to the `request_finished`
signal.

#### Receiver functions

Define a receiver function, can be any Python function or method:

```py
def my_callback(sender, **kwargs):
	print("Request finished!")
```
The function takes a `sender` argument, along with the `**kwargs`: all signal handlers must take these arguments

#### Connecting receiver functions

Two ways

* the manual connect route:

	```py
	from django.core.signals import request_finished

	request_finished.connect(my_callback)
	```

* using a `receiver()` decorator

	```py
	from django.core.signals import request_finished
	from django.dispatch import receiver

	@receiver(request_finished)
	def my_callbac(sender, **kwargs):
		print("Request finished!")
	```

Signal handlers are defined in a `signals.py` module.
Signal receivers are connected in the `ready()` method of the application configuration class (in `apps.py`)

if using the decorator, import the `signals` submodule inside `ready()`.

```py
from django.apps import AppConfig
from django.core.signals import request_finished

class MyAppConfig(AppConfig):
	...

	def ready(self):
		# Implicitly connect when using the @receiver decorator
		from . import signals

		# Explicitly connect a signal handler
		request_finished.connect(signals.my_callback)
```


#### Connecting to signals sent by specific senders

In the case of `django.db.models.signals.pre_save` for example, the sender will be the model class being saved,
so you can indicate that you only want signals sent by some model:

```py
from django.db.models.signals import pre_save
from django.dispatch import receiver
from myapp.models import MyModel

@receiver(pre_save, sender=MyModel)
def my_handler(sender, **kwargs):
	...
```

The `my_handler` function will only be called when an instance of `MyModel` is saved.







### `post_save`

`django.db.models.signals.post_save`, sent at the end of the `save()` method.

Arguments sent with this signal:

* `sender` - the model class
* `instance` - the actual instance being saved
* `created` - a boolean; `True` if a new record was created
* `raw` - a boolean; `True` if the model is saved exactly as presented. Don't query/modify other records in the database
as the database might not be in a consistent state yet.
* `using` - the database alias being used.
* `update_fields` - the set of fields to update as passed to `Model.save()` or `None` if `update_fields` wasn't passed
to `save()`.