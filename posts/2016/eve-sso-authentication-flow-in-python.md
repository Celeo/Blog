I've written a handful of web apps using [EVE Online's](https://www.eveonline.com/) [Single Sign-On service](https://community.eveonline.com/news/dev-blogs/eve-online-sso-and-what-you-need-to-know/), so I'm sharing an example app I've written to demonstrate that flow using my CREST library, [Preston](https://celeodor.com/prest/).

---

First off, I'd like to point to the [community-supported documentation on the SSO authentication flow](https://eveonline-third-party-documentation.readthedocs.io/en/latest/sso/authentication.html) over at readthedocs as it's what I used to learn the process (and a fair bit of trial and error, which is what I'd like to save you from).

Second, you can find the repository demonstrating this flow at [github.com/Celeo/EVE-SSO-Python](https://github.com/Celeo/EVE-SSO-Python). If you create your own EVE third party app at [developers.eveonline.com](http://developers.eveonline.com) and fill in the `config.cfg` file, you'll have a working example.

Preston handles most of this authentication flow for us, including

* generating the URL to redirect the user to
* generating the API token from the code returned by the service
* getting the user's character information from the EVE APIs

The job of the controller code wrapping this process is to:

1. get the URL from Preston
2. redirect the user to that URL
3. catch the returned response
4. give Preston the code
5. handle user login from the available character data

5 steps is not bad for authenticating a user against a service on another site.

First off, getting the URL from Preston is a call to `get_authorize_url`. I include this in the [`Flask.render_template` call to a template](https://github.com/Celeo/EVE-SSO-Python/blob/bc697dd2f1201254147b63a7bbf1a04669357d82/eve_sso/app.py#L64) that acts as the launch page for the user's trip to the EVE SSO:

```python
return render_template('login.html', url=preston.get_authorize_url())
```

When you set up an EVE third part application, you have to provide a **callback URL**. This is where that comes into play: when your user logs into their account and selects their character to authenticate with for your app, the SSO sends the user a redirect to your callback URL which needs to come back to your web app. Your controller's responsibilities 2 through 5 from the list above are handled at this point.

When the SSO sends your user back to your site, the code that Preston needs is in the query string, so the URL will look something like `[your_callback_url]?code=[auth_code]`. Grab the code from the URL with a [`request.args` lookup](https://github.com/Celeo/EVE-SSO-Python/blob/bc697dd2f1201254147b63a7bbf1a04669357d82/eve_sso/app.py#L86) and pass it to Preston:

```python
auth = preston.authenticate(request.args['code'])
```

It's worth noting that this code can be absent if something when wrong, so you'll want to either wrap this line in a `try-except` or use the `request.args.get` method to return `None` if the key isn't in the dictionary and then check for `if not code:`. I opt for the `try-except` as it can also catch anything that goes wrong as part of the process of getting the API token from the code inside Preston.

Now that you have an `auth` variable that's an instance of [`preston.crest.preston.AuthPreston`](https://github.com/Celeo/Preston/blob/master/preston/crest/preston.py#L270), you can access the user's authenticated character's basic information. Preston gives you access to this information through a call to [`whoami`](https://github.com/Celeo/Preston/blob/master/preston/crest/preston.py#L293):

```python
character_info = auth.whoami()
```

The returned dictionary contains the character's name, which this example is using for [flask-login](https://flask-login.readthedocs.io/en/latest/), and their API ID, which is useful for making additional lookups on the EVE XML API like their corporation:

```json
{
	"Scopes": "",
	"CharacterID": 91316135,
	"ExpiresOn": "2016-09-04T03:56:11",
	"TokenType": "Character",
	"CharacterName": "Celeo Servasse",
	"IntellectualProperty": "EVE",
	"CharacterOwnerHash": "hCgR7LGm..."
}
```

When solely using the SSO to authenticate a user, the `Scopes` field is always empty.

From here, it's as simple as taking the character's name and checking for an existing `User` model:

```python
user = User.query.filter_by(name=character_name).first()
```

If `user` isn't `None`, then this character has been used to authenticate a user before. Log the user in with that model and send them off to the rest of the app:

```python
if user:
    login_user(user)
    flash('Logged in', 'success')
    return redirect(url_for('index'))
```

If `user is None`, however, this is the first time that this character has been used for authentication on the app, so a new `User` model will need to be created before logging them in and sending them off:

```python
user = User(character_name)
db.session.add(user)
db.session.commit()
login_user(user)
flash('Logged in', 'success')
return redirect(url_for('index'))
```

That's really it. Add some `flask_login.login_required` decorators to endpoints that you don't want just anyone to see and you have an app that restricts access using EVE's SSO.

If you want to see a larger app that uses this flow, my [GETIN-HR](https://github.com/Celeo/GETIN-HR) app is a good example.