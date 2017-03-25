I've been playing around with front-end JavaScript frameworks for a month or so, trying to fine one that I like. Looks like I have.

---

I'm always working on expanding my skills as a developer. While I've played on both sides of making a web app, back-ends with Python and front-ends with jQuery, front-end JavaScript frameworks seem to be all the rage these days. A lot of jobs require knowledge of one (or more) of these frameworks, so I started learning.

Please note that the opinions here are from a primarily back-end developer who has little eye for style or JavaScript itself.

## React

After reading a bunch of opinions on which is "best," I started with Facebook's [React](https://facebook.github.io/react/). I didn't spent long with React, because the idea of writing JSX turned me away very, very quickly. It just felt so weird, so backwards, to write HTML *inside* JavaScript. Because of that, I really didn't spend a lot of time with React. It's a shame, as I know that a lot of developers sing its praise, but I just could do it.

## Angular 2

Next up, Google's [Angular](https://angular.io/). I did a quick glance through the tutorial and noticed three things:

1. No JSX
2. Boilerplate
3. Wow, what a tutorial!

So, no JSX: great, continuing on.

Now, boilerplate. Angular handles it pretty well with the tutorial, but still - starting off a tutorial with 4 copy-paste files is a bit off-putting. It's probably worth noting that this was my first "real" introduction to npm. I've used npm to install this blog app, and to install `forever` to run it, but I'd never created a `package.json`, so this was a jump right into both a framework and a toolset.

Going through the tutorial was really quite enjoyable. It's very clear that the Angular team has spent a lot of time making it. The font is easy to read, colors are used well, and the entire "Hero" theme keeps reading interesting. Another thing that was new to me: Typescript. I still don't really know much about it (as I've moved to a different framework, keep reading), but it sure seems nice.

My nitpicking for Angular lied in the file structure.

I know, I know, that's pretty petty. I use SublimeText (which has plugins for Angular and Typescript), so switching to a file that I don't even have open is a simple Ctrl+P and fuzzy search for the filename - getting files open isn't a problem. Still, eh.

## Vue

Finally! Something that seems to really embody what I like about [Flask](http://flask.pocoo.org/) - it's simple. Start small and build up instead of starting off with a lot of boilerplate ([Django](https://www.djangoproject.com/) ...) and working out.

The thing I really love about Vue is that, going through the [guide](https://vuejs.org/guide/installation.html), I only needed a single JavaScript file that I could download or use a CDN for and a single HTML file to play with. "Hello world" using Vue is only 16 lines:

```language-html
<html>
<body>
    <div id="app">
        {{ message }}
    </div>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/vue/1.0.25/vue.js"></script>
    <script>
        new Vue({
            el: '#app',
            data: {
                message: 'Hello Vue.js!'
            }
        })
    </script>
</body>
</html>
```

That's all I'll really write for now. I've converted [one of my web apps](https://git.celeodor.com/Celeo/ASOIAF_roller/commits/master) (see from `f5b6ecc262` to `0a44b930a8`) from using just Flask to Flask for the back-end serving JSON to a Vue front-end and really enjoyed the process.
