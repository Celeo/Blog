Requests is a HTTP library.

---

* [GitHub link](https://github.com/kennethreitz/requests)
* [PyPi link](https://pypi.python.org/pypi/requests/)
* [Docs](http://docs.python-requests.org/en/master/)

Requests is an Apache2 Licensed HTTP library, written in Python, for human beings.

> Python’s standard urllib2 module provides most of the HTTP capabilities you need, but the API is thoroughly broken. It was built for a different time — and a different web. It requires an enormous amount of work (even method overrides) to perform the simplest of tasks.

> Things shouldn’t be this way. Not in Python.

I'm sure many (if not nearly all) of you have heard of and probably used requests, but in case you haven't, here's to you.

## Installation
Like most Python packages, you can install with pip:

```language-bash
$ pip install requests
```

## Usage
The simplest use of requests is to get the HTML source of a website, often for parsing the result for an app. The code for that is extremely simple:

```language-python
import requests

r = requests.get('http://google.com')  
```

Now, r is an instance of a `requests.models.Response` object:

```language-python
In [6]: r = requests.get('http://google.com')

In [7]: type(r)  
Out[7]: requests.models.Response  
```

With that object, we can get a lot of data about the page we targeted. The most common in my experience where the following:

* r.text - this is the HTML that the page returned to the "browser". Note that this is the HTML from the page, not the HTML after any JavaScript ran that may have modified it.
* r.status_code - this is the HTTP status code that the website returned - very useful for determining if you request failed: r.status_code in [400, 403, 404, 500, 502]
* r.url - this is the "final" URL that the server routed your request to (requests follows redirects)

The contents of r.text can then be fed to something like BeautifulSoup to parse. Remember, [don't parse HTML with regex](http://stackoverflow.com/a/1732454/2676531).

But that's all that requests can do. Do you have or need to test a RESTful API? Take a look back at the example code:

```language-python
r = requests.get('http://google.com')  
```

See the `.get` call? Well, there are also `.post`, `.put`, `.delete`, `.head`, and `.options`. These all return a Response object.
