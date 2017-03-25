I recently came across a small community building custom "start pages" for their browsers.

---

When you open a new page in your browser of choice, it can be configured to show several different pages. I use Chrome, so I installed the [New Tab Redirect](https://chrome.google.com/webstore/detail/new-tab-redirect/icpgjfneehieebagbmdbhnlpiopdcmna/related) extension to allow me to show any page, including a file on my local PC.

I've been working with [Vue.js](https://vuejs.org/) lately, so I wanted to take the opportunity to build a web page that I could use for my start page, like the people over at [/r/startpages](https://www.reddit.com/r/startpages/) are doing.

Repo link: [github.com/Celeo/Start_page](https://github.com/Celeo/Start_page)

I have all the sites that I need to visit either on my bookmarks bar (which is always showing) or in muscle memory, so I didn't want to have a bunch of links on my start page, although this is a common usecase of the page and it works well for many people. There are only 2 things I'm really interested in: the time and the weather.

Now sure, I could just look at my taskbar for the time and go to any number of weather sites (or look outside) for the weather, but I wanted to add *something* to this page.

### Time

Adding the time to the page is the simplest: [Moment.js](http://momentjs.com/) is an easy way to access the time and display it in a variety of formats. I added a data field in my Vue object for the time:

```javascript
export default {
  data() {
    return {
      time: moment()
    }
  },
```

Vue will take care of updating the DOM when the object is changed, but it won't update on its own:

```javascript
mounted() {
  setInterval(() => this.time = moment(), 1000)
}
```

Just add the variable to the template and you'll have the up-to-date time (and/or date!) on the start page.

### Weather

Weather is a bit more tricky. I tried querying a variety of sites straight from the page but was repeatedly met with [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS) problems. I needed to query the weather from a server, not a webpage. Luckily, [Express](http://expressjs.com/) is a simple web framework for Node. I created a small, single-endpoint web app that I could run in the background on my workstation without any performance problems, just to be the go-between for my start page and the remote weather API, for which I'm using [Dark Sky](https://darksky.net/dev/).

The server *could* have it's own configuration file, but I found it easier to abstract that behavior on the server's side and leave it to the client to send the server all the configuration information it needs as part of the request (mainly because I was tired of fighting with getting files to be in the right places during development and deployment, but this is the better approach regardless).

The server, when it receives a request from the client (the start page), sends a request over to Dark Sky for the weather, gets that response, parses it for the information the client wants, and sends a response back to the start page. The start page then takes this information, stores it in local storage for caching, and Vue updates the DOM. The information is cached for two reasons:

1. Weather doesn't change very fast
2. Dark Sky has daily API access limits

### Moving forward

I decided to use the [webpack-simple](https://github.com/vuejs-templates/webpack-simple/) [vue-cli](https://github.com/vuejs/vue-cli) template for the Vue app and bundle it all with webpack. To use the start page, I pointed New Tab Redirect at Webpack's build target **index.html**. Running the sever is a simple as using whichever mechanism your OS has for running console apps in the background without a window.

To update the client app, just pull from Github and rebuild. To update the server app, you'll have to kill the background process, build, and start it again.

I started to use Google's Maps API to track the time it'd take me to get home from work, which would only show on the page after 4 PM, but it seems that Google sends the same time, regardless of the traffic, so it's not accurate.