Using websockets to communicate the progress of a task ran in a separate thread.

---

This article is to celebrate the 2.0 release of [flask-socketio](https://flask-socketio.readthedocs.io/en/latest/), specifically the fix of [Issue #47](https://github.com/miguelgrinberg/Flask-SocketIO/issues/47), which now allows the server to emit a message to connected websocket clients from a Celery task.

The full source of this app can be found [here](https://git.celeodor.com/Celeo/Flask-sockets-celery-redis). If you look, you'll see many similarities to flask-socketio's sample app.

## Setting up the app

We'll get the imports out of the way:

```language-python
from flask import Flask, render_template  
from flask_socketio import SocketIO, emit  
from datetime import timedelta  
from celery import Celery  
import time

import eventlet  
```

Note the last one: `import eventlet`. Eventlet is an asynchronous concurrent networking library. The next line is to enable the usage of a background thread:

```language-python
eventlet.monkey_patch()  
```

Next, generic app setup:

```language-python
app = Flask(__name__)  
app.config.from_pyfile('config.cfg')  
app.permanent_session_lifetime = timedelta(days=14)  
```

and the flask-socketio setup:

```language-python
socketio = SocketIO(app, async_mode='eventlet', message_queue=app.config['SOCKETIO_REDIS_URL'])  
```

The 'SOCKETIO_REDIS_URL' value is defined in the app's config file:

```language-pythno
SQLALCHEMY_DATABASE_URI = 'sqlite:///../data.db'  
SQLALCHEMY_TRACK_MODIFICATIONS = False  
CSRF_ENABLED = True  
SECRET_KEY = 'super-secret-key'  
SOCKETIO_REDIS_URL = 'redis://localhost:6379/0'  
CELERY_BROKER_URL = 'redis://localhost:6379/0'  
CELERY_RESULT_BACKEND = 'redis://localhost:6379/0'  
```

Back to the app, we'll setup Celery:

```language-python
celery = Celery(app.name, broker=app.config['CELERY_BROKER_URL'])  
celery.conf.update(app.config)  
```

which completes the setup. We have a Flask application with flask-socketio using eventlet to handle the websocket connections, and Celery setup to handle background tasks.

## Routes

This small app only has one page: the index page, defined simply as:

```language-python
@app.route('/')
def index():  
    return render_template('index.html')
```

That's not all that's in app.py, of course. We have an endpoint at `/bg` to run the background task:

```language-python
@app.route('/bg')
def start_background_thread():  
    background_task.delay(app.config['SOCKETIO_REDIS_URL'])
    return 'Started'
```

That background task is a Celery task:

```language-python
@celery.task()
def background_task(url):  
    local_socketio = SocketIO(message_queue=url)
    local_socketio.emit('my response', {'data': 'background task starting ...'}, namespace='/test')
    time.sleep(10)
    local_socketio.emit('my response', {'data': 'background task complete!'}, namespace='/test')
```

Here we're using flask-socketio's 2.0 release feature in the initialization of a SocketIO object, passing a message_queue. Note that it's the same as when we initialized the "master" SocketIO object as part of the app's setup. Then, we emit a message, wait 10 seconds to simulate a background task, and emit a message declaring completion.

Finally, we have a method to respond to messages sent by the frontend:

```language-python
@socketio.on('my event', namespace='/test')
def test_message(message):  
    emit('my response', {'data': 'Backend saw "' + message['data'] + '" from the frontend'})
```

This method echoes messages back to the frontend with a confirmation.

## Templates

The base.html template is pretty simple, the noteworthy section being the JavaScript to setup the websocket connection:

```language-javascript
$(document).ready(function(){
    namespace = '/test';
    socket = io.connect('http://' + document.domain + ':' + location.port + namespace);
    socket.on('connect', function() {
        socket.emit('my event', {data: 'I\'m connected!'});
        $('#log').append('Connected<br>');
    });
    socket.on('my response', function(msg) {
        console.log('Received: ' + msg.data);
        $('#log').append('Received: ' + msg.data + '<br>');
    });
});
```

This block is wrapped in a jQuery `$(document).ready` function so it's only executed once the page has finished loading. A variable called socket is created that is the frontend's connection to the websocket server. We setup two event listeners: one for when a connection is made, and the other for receiving 'my response' messages from the backend. Each logs the message, and the connection listener additionally messages the backend.

The index.html template has a textbox and button so the user can manually send messages to the backend through the websocket, a button to test the background thread functionality, and a log area to record messages from the backend.

The additional JavaScript in the head supports these:

```language-javascript
$(document).ready(function() {
    $('#send_message').on('click', function() {
        socket.emit('my event', {data: $('#message').val()});
        $('#log').append('Sent: ' + $('#message').val() + '<br>');
    });
    $('#background').on('click', function() {
        console.log('1');
        $.get("{{ url_for('start_background_thread') }}");
        console.log('2');
    });
});
```

When the first button, the one connected to the textbox, is clicked, the value of the textbox is sent through `socket.emit('my event', {data: $('#message').val()});`.

When the second button is clicked, a '1' is logged to the console, a GET request is sent to the URL for the background_task endpoint, and then a '2' is logged. The logging is to prove that the call isn't blocking.

## Development bumps

This is a simple proof of concept app that uses websockets and mock processing of a background task. Once completed, it doesn't look like it could have too many problems, but there were several hurdles that I had to get over while developing it.

 #1. from time import sleep

Originally, I was importing sleep straight from time instead of importing the entire time module and calling time.sleep, per usual. I found that, when I did this, the call to sleep was blocking the main processing thread, not just the background thread.

 #2. Not running the separate Celery worker task

This one was from a failure to read all the documentation available to me, but I originally wasn't running the Celery worker (present in the repository as start_worker.sh) and couldn't figure out why the background task wasn't being processed.

 #3. Not installing redis beforehand

This one is a bit embarassing, but I was operating under the impression that I had installed redis sometime in the past. I hadn't. Be sure that you don't waste the time that I did.