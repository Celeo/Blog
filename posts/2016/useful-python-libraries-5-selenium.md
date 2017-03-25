Web browser automation

---

> "Selenium automates browsers. That's it! What you do with that power is entirely up to you. Primarily, it is for automating web applications for testing purposes, but is certainly not limited to just that. Boring web-based administration tasks can (and should!) also be automated as well."

* [GitHub](https://github.com/SeleniumHQ/selenium)
* [PyPi](https://pypi.python.org/pypi/selenium)
* [Homepage](http://www.seleniumhq.org/)
* [General Docs](http://www.seleniumhq.org/docs/)
* [Official Python Docs](https://seleniumhq.github.io/selenium/docs/api/py/api.html)
* [Additional Python Docs](http://selenium-python.readthedocs.io/)

Selenium has bindings for Java, C#, Python, Ruby, PHP, Perl, and JavaScript. We'll be looking at the Python package. But it also has a full IDE in the form of a Firefox plugin that lets the user create test suites without ever touching code.

## Installation
The Python bindings are installed with a single command:

```language-bash
$ pip install selenium
```

## Making the connection
To start off, we only have to import the webdriver module:

```language-python
from selenium import webdriver  
```

This lets us create our driver that we'll use to control the browser. By default, Firefox is supported out of the box (if you have Firefox installed on your system, of course). We'll talk about Chrome, IE, and others later.

```language-python
driver = webdriver.Firefox()  
```

We'll need to keep this driver object around - creating additional webdriver.Firefox objects will open additional browsers so unless that's your intention, stick with the single object.

Navigating to a URL is done with the get method:

```language-python
driver.get('google.com')  
```

## Interacting with the page
To interact with the page, we need to locate DOM elements on the page. We do this through `find_element(s)_by_` methods. My favorite is `find_element_by_xpath` due to the ease of getting the paths from Chrome's DevTools console.

Open Chrome and navigate to the page manually that you want to interact with. Right-click the element you'll be interacting with (clicking, getting the text, etc.) and select Inspect from the context menu. The DevTools panel will pop up. Right-click the highlighted row and select Copy > Copy XPath. This will copy the XPath to your clipboard.

Back in the code (I find that using the Python interpreter is extremely useful for setting up Selenium test cases due to the instant feedback):

```language-python
element = driver.find_element_by_xpath('//*[@id="tsf"]/div[2]/div[3]/center/input[1]')  
```

This gets us a `WebElement` object.

The methods here I use most often are:

* click - click the element as if the user had clicked with the mouse; used for links to navigate about the site without directly navigating to URLs
* send_keys - enter values into a textbox or similar; used in submitting forms, often for logging in
* submit - submit a form; used to submit those forms that you entered data into

If we wanted to enter "selenium python" into Google's search bar and click the search button, we'd write

```language-python
driver.find_element_by_xpath('//*[@id="lst-ib"]').send_keys('selenium python')  
driver.find_element_by_xpath('//*[@id="tsf"]/div[2]/div[3]/center/input[1]').click()  
```

## Taking screenshots
We can have the driver take a screenshot of what's currently displayed in the browser with get_screenshot_as_file:

```language-python
driver.get_screenshot_as_file('/path/to/screenshot.png')  
```

Depending on how you structure your code, you may want to wrap large portions of the calls in try-except blocks and have the except block take a screenshot of what's displayed on the page for debugging later.

## Other browsers
I noted earlier that Firefox works out of the box, but Chrome, IE, Opera, and Safari can also be used. Use of any of these browsers requires 2 things:

1. The browser is installed on the system
1. You download the driver for those browser and pass it to the respective constructor

First, download the driver(s) that you want:

* [Chrome](https://github.com/seleniumhq/selenium-google-code-issue-archive)
* [IE](https://selenium-release.storage.googleapis.com/index.html)
* [Opera](https://github.com/operasoftware/operachromiumdriver)
* [Safari](https://github.com/SeleniumHQ/selenium/wiki/SafariDriver)

And then pass the path to the binary when constructing the object:

```language-python
driver = webdriver.Chrome('/path/to/chromedriver.exe')  
driver = webdriver.Ie('/path/to/iedriver.exe')  
driver = webdriver.Opera('/path/to/operadriver.exe')  
driver = webdriver.Safari('/path/to/safaridriver.exe')  
```

I've found that using the IE driver is more trouble than it's worth - the driver is extremely slow.

## Review
Selenium lets us automate web browsers. This can be used to perform automated tests of a website (write a script, toss it into a cron job), check your favorite website for changes, or just perform a routine task that you'd rather not manually click through yourself.