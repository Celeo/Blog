[Prest](https://celeodor.com/prest/) now supports refresh tokens.

---

In the exceptionally well-*\*cough*\*-documented [GitHub Issue #4](https://github.com/Celeo/Prest/issues/4), I added support for refresh tokens to Prest.

When accessing the EVE CREST API, __access tokens__ are used to actually show CREST that you're acting on behalf of a specific character and have the right to do so. This access token is returned from the EVE SSO.

Here's the catch though - the access token is only valid for 20 minutes.

When the only thing you're using CREST for is authenticating a user as the owner of a character, this doesn't matter, so you don't need to bother with refresh tokens. In fact, just using the SSO to authenticate won't even return a refresh token.

If you're requesting any [scopes](https://eveonline-third-party-documentation.readthedocs.io/en/latest/crest/authentication.html#scopes), though, you'll probably want more than 20 minutes with that character's data. Thus, the __refresh token__.

When your app authenticates with the EVE SSO and you exchange the returned code for the access token, you'll get a refresh token back as well:

```json
{
  "CharacterID": 91316135, 
  "CharacterName": "Celeo Servasse", 
  "CharacterOwnerHash": "...", 
  "ExpiresOn": "2016-07-04T05:08:39", 
  "IntellectualProperty": "EVE", 
  "Scopes": "characterLocationRead", 
  "TokenType": "Character", 
  "access_token": "...", 
  "expires_in": 1200, 
  "refresh_token": "...", 
  "token_type": "Bearer"
}
```
<sup><sup>private information stripped</sup></sup>

From the time that this response is generated, you have 20 minutes with that access token. In this response, I've had the user grant my app access to reading their character's location while they're online - useful for fleet ops or wormhole mapping.

Just like when using CREST to authenticate character ownership, though, the access token expires in 20 minutes. When this happens, you'll need to use the refresh token to get a new access token. The refresh token does not expire and will last until the user revokes access to your app.

So yeah, the refresh token is very powerful.

Prest will now automatically get a new access token from CREST using the stored refresh token (if it exists) when the access token expires and then continue on with what you asked it to do.

What's next for Prest?

I want to continue work on the unit tests. I haven't had a lot of experience with unit testing, so I'm still getting used to it. In this last commit (which was a multi-commit feature branch squash-merged back into master), I broke some testable functionality out into separate methods, so they're prime targets for some unit tests. Also, I want to look into coverage - I know Pytest has a [coverage plugin](https://pypi.python.org/pypi/pytest-cov), so I want to use it.