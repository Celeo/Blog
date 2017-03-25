What do you get when you wrap a Flask app behind a Nginx reverse proxy managed by Circus and wrapped in a Docker container? The need to read a lot of documentation.

---

I've been recently working with Docker. I know I'm a bit late to the party, but I've been having some fun working with images and containers and daemons and Dockerfiles and all that. At home, I primarily work on Python applications, most of which are web apps with Flask.

Recently, I wanted to Dockerize (that's a verb, right?) a Flask application, but was running into issues with getting a container to run Nginx and the Flask app at the same time (the solution turned out to be running Nginx in a no-daemon mode, by the way) and most of the answers I read on Stack Overflow suggested using Supervisor to manage both the Nginx and Gunicorn (running the Flask app) processes inside the Docker, which is only supposed to run a single process per container.

I had a big problem though: I use Python 3 and Supervisor is only for Python 2.X.

Now, there were a few forks for Python 3, but one of the GitHub issue comments recommended using Circus, a 3.X-compatible app that could do the same, with more fancy stuff. I didn't need the fancy stuff, but I did need the 3.X compatibility, so off to Circus' website.

I want to specifically note that I'm only using Circus to manage the Nginx and Gunicorn processes. In the documentation I see instructions for using Circus to be a more integral part of the web stack, but since I'm used to Nginx+Gunicorn, I'm using it for this very limited scope. In the future I'll go back and learn more.

A Supervisor configuration file to do what I want would look like this:

```ini
[supervisord]
nodaemon=true

[program:nginx]
command=/usr/sbin/nginx

[program:gunicorn]
command=/usr/local/bin/gunicorn app:app -b localhost:5000
```

The Circus configuration file looks like this:

```ini
[watcher:nginx]
cmd = /usr/sbin/nginx

[watcher:gunicorn]
working_dir = /srv/server
cmd = /usr/local/bin/gunicorn app:app -b localhost:5000
```

Though the blocks and variables are named differently between the two apps, the overall structure of the configuration file is very similar.

In my Dockerfile, I have the following to run Circus:

```language-docker
COPY circus.conf /srv
CMD circusd /srv/circus.conf
```

When the container starts, the default CMD starts Circus and points it to the configuration file in /srv.

There's not much more to say here, I wanted to quickly write this up to say that, for the very simple use case of "I want to run Nginx and a web app in Docker", Circus is as simple to use a Supervisor.