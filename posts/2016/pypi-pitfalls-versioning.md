PyPI is a wonderful tool, but uploading is sometimes a nuisance.

---

## Markdown vs RST

[Prest](https://celeodor.com/prest/) is the first Python code I've uploaded to PyPI and I've written about uploading it to PyPI. As I update the code, I push versions up to PyPI so they're available to download and install with [pip](https://pip.pypa.io/en/stable/).

I haven't quite gotten the hang of it, though.

My first problem was that PyPI [doesn't](https://bitbucket.org/pypa/pypi/issues/148/support-markdown-for-readmes) [support](http://stackoverflow.com/q/26737222/2676531) [Markdown](https://kromey.us/2015/01/pypi-doesnt-like-your-markdown-685.html). It won't render it, so it'll appear as plaintext, which looks really bad. PyPI does, however, support [reStructuredText](https://en.wikipedia.org/wiki/ReStructuredText). Though GitHub also supports RST, [Gogs](https://gogs.io/) [doesn't](https://github.com/gogits/gogs/issues/211). Having to update two versions of the same documents would take way more effort than I want to put in, so I had to find a way to convert the Markdown doc into RST for the purpose of getting it onto PyPI and not look like I don't care about the README.

Enter [Pandoc](http://pandoc.org/).

I came across Pandoc in the same way people find most things: Google. I was led to [this](http://stackoverflow.com/a/10719349/2676531) Stack Overflow answer which recommended it, but the comment from [Jonathan Eunice](https://stackoverflow.com/users/240490/jonathan-eunice):

> The magical invocation is: `pandoc --from=markdown --to=rst --output=README.rst README.md`

This is one of the cases where the comment on the answer is actually more useful than the answer itself.

I wrapped my PyPI upload command into a [script](https://git.celeodor.com/Celeo/Prest/src/master/pypi_upload.sh):

```bash
#!/bin/sh
pandoc --from=markdown --to=rst --output=README.rst README.md || exit 1
python setup.py sdist upload -r pypi || exit 1
rm README.rst || exit 1
echo 'Done'
```

See the `|| exit 1`s?

## Immutable versioning

I'm not really sure what else to call it, but once you upload a version of a package to PyPI, you cannot upload again under that version.

Now, first off, I get it. It makes sense - it promotes ... at least, decent, versioning - versioning that a user could understand (hopefully). If someone uploads a version of their library to PyPI and attempts to upload new code some time later without updating their library's version, PyPI would catch it and they wouldn't confuse a bunch of users who may be wondering why their `pip install -r requirements.txt` is pulling the same version of the library from PyPI but their program isn't working.

It is, however, a pain when the library developer makes a mistake, like, say, uploading their README in Markdown intead of RST. Instead of quietly re-uploading (like the dev could do with `git commit --amend` and `git push origin master -f`) and hoping it didn't impact anyone, the mistake is clear.

Raise your hand if you've been there.

[I have](https://git.celeodor.com/Celeo/Prest/commit/c19594a1884c25a5b3504fe7b29d08a74f8fc4fa).

So what's the fix? Well, carefully reviewing files before uploading, and PyPI Test.

## The Test Server

As far as I know, [PyPI Test](https://testpypi.python.org/pypi) is the same thing as the main site, but with the following differences:

1. Different accounts - you'll need to register for an account separately on the test site
2. The package index

This comes in handy when you're trying out, for a totally random, didn't-happen-to-me example, how the site renders READMEs. Change your PyPI upload command to `python setup.py sdist upload -r pypitest` with the right [`.pypirc`](https://docs.python.org/3.5/distutils/packageindex.html#pypirc) and your changes are off to the test site, where packages are cleared every so often. This is the place to test these sort of things and get everything nice and pretty before putting it up on the main site for everyone to see.

Now, sure - everyone uploading to PyPI is a developer, and the pitfalls that ensnared me are hardly unique, so it's not a *big* deal. I just want to make sure that the stuff I'm putting out there as "ready" is actually ready.