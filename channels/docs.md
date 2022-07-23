Steps to use Channels

* Configure ASGI
* Consumers
* Routing
* WebSockets

```
pip install channels
```

Add `channels` to `INSTALLED_APPS`


Create routing configuration for the channels.

in `asgi.py` 

```py
from channels.routing import ProtocolTypeRouter

application = ProtocolTypeRouter({
	'http':get_asgi_application()
})
```

Back in `settings.py`

```py
ASGI_APPLICATION = 'mywebsite.asgi.application'
```


establish a webSocket connection

in `lobby.html` add a script tag with this
`<script type='text/javascript'>`

```js
let url = `ws://${window.location.host}/ws/socket-server/`

const chatSocket = new WebSocket(url)

chatSocket.onmessage = function(e) {
	let data = JSON.parse(e.data)
	console.log('Data:', data)
}
```


Consumers

in `chat` create `consumers.py`
- channels version of django views

```py
import json
from channels.generic.websocket import WebsocketConsumer

class ChatConsumer(WebsocketConsumer):
	def connect(self):
		self.accept()

		self.send(text_data=json.dumps({
			'type': 'connection_established',
			'message': 'You are now connected'
		}))
```

Create `routing.py`

```py
from django.urls import re_path
from . import consumers

websocket_urlpatterns = [
	re_path(r'ws/socket-server/', consumers.ChatConsumer.as_asgi()),
]
```

Update `asgi.py`:

```py
from channels.routing import URLRouter
from channels.auth import AuthMiddlewareStack
import chat.routing

application = ProtocolTypeRouter({
	...
	'websocket': AuthMiddlewareStack(
		URLRouter(
			chat.routing.websocket_urlpatterns
		)
	)
})
```

Run `migrate`.


Sending messages

In `lobby.html`:

Create a `<form>`

```html
<form action="" id="form">
	<input type="text" name="message" id="">
</form>
```

And in the `<script>` tag:

```js
let form = document.getElementById('form')
form.addEventListener('submit', (e) => {
	e.preventDefault()
	let message = e.target.message.value
	chatSocket.send(JSON.stringify({
		'message': message
	}))
	form.reset()
})
```

back in the `ChatConsumer` in `consumers.py`:

```py
def receive(self, text_data):
	text_data_json = json.loads(text_data)
	message = text_data_json['message']

	print('Message:', message)
```


Channel Layers

Create relations between different instances of an application for real-time communication.

Optional part of django channels.

Groups and channels

Think of groups as chatrooms that store info about users in a particular room. stored inside of an in-memory database.

Inside are a bunch of channels/users.
A channel is simply a mailbox representing a user in the room.

Anyone who knows the channel name, can send messages to that
user via that channel.

Anytime a user joins a room, add that users channel to that group.

Messages go to the group, and to all the channels in it.

Enable channel layers

In `settings.py`:

```py
CHANNEL_LAYERS = {
	'default': {
		'BACKEND': 'channels.layers.InMemoryChannelLayer'
	}
}
```

In a production environment, store it in something like redis
`InMemoryChannelLayer` is for testing purposes only.

In `consumers.py`:

```py
from asgiref.sync import async_to_sync

class ChatConsumer(websocketConsumer):
	def connect(self):
		self.room_group_name = 'test'

		async_to_sync(self.channel_layer.group_add)(
			self.room_group_name,
			# channel_name is created for us
			self.channel_name
		)

		self.accept()

	def receive(self, text_data):
		text_data_json = json.loads(text_data)
		message = text_data_json['message']

		async_to_sync(self.channel_layer.group_send)(
			self.room_group_name,
			{
				'type': 'chat_message',
				'message': message
			}
		)

	# Create a function 
	def chat_message(self.event):
		message = event['message']

		self.send(text_data=json.dumps({
			'type': 'chat',
			'message': message
		}))
```

Usually use a dynamic group name but in this case, just use `test`, 