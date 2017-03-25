(The best) Web microframework

---

> "Flask is a microframework for Python based on Werkzeug, Jinja 2 and good intentions."

* [GitHub](https://github.com/pallets/flask)
* [PyPi](https://pypi.python.org/pypi/Flask)
* [Homepage](http://flask.pocoo.org/)
* [Docs](http://flask.pocoo.org/docs/0.10/)

When I got into web development originally, I, like so many other people, started with PHP on Apache on a free hosting platform. When I started to learn Python, I jumping right in with Python, the Django framework, and web development all at once. It was a rough journey, but well worth the effort eventually. After many months of using Django, I switched to Flask after a glowing recommendation from a fellow developer. After plugging around for a bit, I switched my active web apps from Django to Flask and have been using Flask ever since.

## Installation
Like many Python libraries, Flask's installation is a simple:

```language-bash
$ pip install flask
```

Flask comes with several dependencies, but none of them require anything special on your system.

## Hello world
Right from the framework's homepage, here's the standard "Hello world" application:

```language-python
from flask import Flask  
app = Flask(__name__)

@app.route("/")
def hello():  
    return "Hello World!"

if __name__ == "__main__":  
    app.run()
```

Open your browser and navigate to http://localhost:5000. You should see a black-on-white page with "Hello World!" displayed in the upper-left corner.

This is an extremely simple application to be sure, but take a minute to look back at the code you wrote (copy-pasted). 9 lines of code (11 if you have the trailing newline and an extra newline between the imports and the method for PEP8).

## Resources
I could write about how to do certain things with the library, starting all the way from the Hello world application, but other people have already done a wonderful job of doing that.

Miguel Grinberg's blog series [The Flask Mega-Tutorial](http://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world) is how I started learning the framework. I definitely recommend going through the entire series (and his other articles on Flask).

The official docs are an excellent resource. Unlike some libraries, this documentation is very well written and can take you from no knowledge to a simple application.

Finally, the [StackOverflow Flask tag](https://stackoverflow.com/questions/tagged/flask) is active and has several people who really know the framework.

My largest web application is written in Flask. You can view the source of EveTrackers on my [git server](https://git.celeodor.com/Celeo/EveTrackers).

## Plugins
When I switched from Django to Flask, I was disheartened to learn that many things that I had come to use in Django that were included out of the box, like an ORM, were not included (by default) in Flask. I quickly learned, however, that Flask has an active plugin community that provides all of these things. You can pick and choose what your application needs (like an ORM) and add those in. Some would argue that if the developer is going to add in everything that Django has out of the box, then using Django from the start makes more sense, but I like the ability to pick and choose what I need in my application to start from a small application and build up instead of starting with a larger application and just using what I need from what's already included.

## Review
Flask is my favorite web framework. I could say that several times with many more words, but it's that simple: I like Flask. I strongly recommend that you take a look at this simple but powerful library.